---
type: question
status: active
feynman_pass: false
mastery_evidence: none
video_value: strong
related_concepts:
parent_question:
created: 2026-08-28
---

# 一轮 3DGS 训练如何从输入图像走到 Gaussian 参数更新？

## 为什么会问这个问题？

用户明确指出此前学习连续下钻 SH、footprint 和 alpha 等细节，却缺少 3DGS 的整体地图。当前教学策略调整为“先建立完整生命周期，再在阶段边界做费曼检查”。本题作为 3DGS 源码学习的新总入口，目标是闭卷讲清输入、初始化、单轮训练、动态增密和最终模型之间的关系。

## 第一次闭卷回答

> 基本上来说，输入3DGS算法的是一组相机位姿，对应视角的图片，和空间中的一组稀疏点云，点云有xyz坐标，它们一开始会被初始化为一个高斯圆球，包括了mean，sh，alpha，协方差矩阵（rotation，scale），然后这些圆球会被投影到二维平面上，平面被划分了一个个的tile，每个instance会投影到好几个tile中，每个tile有着对应的instance序号，然后根据这个序号和深度，计算每个tile里面椭球透明度的堆积程度，最后得到tile的颜色，最终得到了当前的渲染图片，和真实的图片做loss，得到了当前RGB Loss，得到loss以后就开始反向传播，优化器确认一个梯度下降的方向，由pytorch进行链式求导，把mean，sh，alpha，协方差矩阵（rotation，scale）的变化传递到这些参数中，椭球的形态也发生了改变，在此过程中，如果这个区域的椭球数量不够拟合图片，或者椭球数量过多，会进行椭球分裂，或者椭球剔除的操作，然后继续这样渲染一遍，得到Loss，反向传播，直到到达指定轮次

## 当前判断

宏观主干已经成立：输入与初始化 → Gaussian 参数 → 投影和 Gaussian–Tile instances → 渲染图 → loss → backward → optimizer → densification/pruning → 循环。当前不下钻公式，只修正会改变整体执行模型的结构点：

- 输入还包括相机内参；训练迭代通常从训练相机集合中选择一个视角，而不是每轮同时处理所有图片。
- 初始化时 scale 常为三个方向相同、rotation 为单位旋转，因此可近似称为圆球；训练后一般成为有方向的椭球。代码优化的是 scale/rotation 表示，协方差可在 rasterizer 内由它们计算。
- tile 是并行组织和复用 Gaussian 列表的单位，不是最终颜色单位；同一 tile 内每个像素线程分别计算自己的 Gaussian 权重、alpha 和颜色。
- backward 先产生参数梯度，optimizer 再根据梯度及 Adam 状态更新参数；不能把两者合并成“optimizer 负责链式求导”。
- densification/pruning 不是直接理解场景语义，而是根据屏幕空间梯度、Gaussian 尺寸、opacity 和训练阶段等启发式信号执行，也不是每轮都发生。

## 最小真实代码证据

FasterGS `blend_cu` 中，一个 CUDA block 对应一个 tile，但 block 内 thread 对应具体像素：

```cpp
const uint2 pixel_coords = make_uint2(
    tile_x * tile_width + thread_x,
    tile_y * tile_height + thread_y
);

const uint tile_idx = tile_y * grid_width + tile_x;
const uint2 tile_range = tile_instance_ranges[tile_idx];

float3 color_pixel = make_float3(0.0f);
float transmittance = 1.0f;

// 每个像素使用同一 tile 的 Gaussian 列表，但按自己的 pixel 坐标求权重并混合。
image[pixel_idx] = color_pixel.x;
```

## Codex 费曼追问

当前只修正一个宏观结构点：

> 请用自己的话区分 tile 和 pixel 在渲染中的职责：tile 为 Gaussian 做了什么组织工作？同一 tile 内的每个像素线程又分别计算什么？为什么同一 tile 内的像素即使共享同一份 Gaussian instance 列表，最终颜色仍然可以不同？

## 我的补充回答

第一轮追问回答：

> tile和pixel的意思就好比cuda线程中的warp和thread，我们只是一次性用一个tile去求pixel的颜色，因为一个Gaussian的颜色从各个角度不一样，因为sh系数的原因，所以每个pixel线程求出来的颜色也不相同

判断：已经理解 tile 是一组像素的并行组织单位、pixel thread 独立产生输出。需要纠正两点：在当前 `blend_cu<<<grid, block>>>` 中更准确的映射是 tile 对应 CUDA block、pixel 对应 block 内 thread；block 内可能包含多个 warp。固定相机下，一个 Gaussian 的 SH 颜色由 Gaussian 位置到相机的观察方向求得，并在其 footprint 内共享；同一 tile 的像素颜色不同，主要因为每个像素与 Gaussian 投影中心的 `delta` 不同，得到不同 `gaussian`、alpha 和 transmittance，并对多个 Gaussian 形成不同混合结果。

第二轮仍停留在渲染模块的宏观输入输出，不扩展 CUDA 细节：

> 假设一个 tile 中只有同一个红色 Gaussian，像素 A 靠近它的投影中心，像素 B 位于 footprint 边缘；两者读取的 Gaussian instance 和 SH 颜色完全相同。为什么 A 和 B 最终仍会得到不同深浅的红色？

第二轮回答：

> 因为高斯椭球是一个越到边缘，分布越稀疏的结构，所以中心的颜色要更深，边缘颜色逐渐变浅，因此A和B最终会得到不同深浅的红色

判断：直觉正确。更准确的表述是投影后的二维 Gaussian 函数值从中心向外连续衰减，而不是高斯内部存在离散粒子疏密；因此相同 SH 颜色在 A、B 像素处乘到不同的空间权重和 alpha。tile/block、pixel/thread、共享 Gaussian 列表与逐像素颜色的宏观关系已解决。

第三轮返回单轮 iteration 的数据输入：

> 假设训练集有 100 个相机视角，这份 `train.py` 的一次 iteration 是同时渲染 100 张图，还是只随机选择其中一个视角？如果每轮只用一个视角，为什么经过很多轮以后，同一组 Gaussian 仍能逐渐同时符合多个视角？

---

## 暴露出的子问题

- 细节问题暂时停放：[[2D Gaussian 的 footprint 和逐像素权重分别是什么]]。
- 后续返回：[[一个 Gaussian 的 SH 与 opacity 能否在同一 footprint 内表达不同颜色]]。
- 改进机制阶段再返回：[[为什么渐进分辨率训练必须与高斯增长预算协同]]。
- 后端阶段再返回：[[FasterGS 中 loss.backward 如何穿过自定义 CUDA rasterizer 把梯度传回 Gaussian 参数]]。

## 最终闭卷解释

> 只有在完整生命周期和必要阶段检查完成后填写，优先保留用户自己的语言。

## Feynman Test

### What

### Why

### How

### Example

### Common Misunderstanding

## 涉及知识

暂不提取 Concept；先稳定整张地图。

## 项目关联

- [[基于 Faster-GS 的快速训练优化与海光 DCU 适配]]

## 视频价值

strong

## 学习结果

- Feynman Pass: false
- Remaining gaps: tile/block、pixel/thread、共享 SH 颜色与逐像素空间权重已解决；继续确认单轮随机相机输入、iteration 阶段顺序、语言边界和改进模块定位。
