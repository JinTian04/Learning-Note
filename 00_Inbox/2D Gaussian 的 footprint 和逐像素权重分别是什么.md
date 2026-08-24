---
type: question
status: active
feynman_pass: false
mastery_evidence: none
video_value: possible
related_concepts:
parent_question: "[[一个 Gaussian 的 SH 与 opacity 能否在同一 footprint 内表达不同颜色]]"
created: 2026-08-24
---

# 2D Gaussian 的 footprint 和逐像素权重分别是什么？

## 为什么会问这个问题？

用户在解释单个 Gaussian 的 SH 与 opacity 时，尚未确定屏幕空间 footprint、二维 Gaussian 值、alpha 和 blend weight 分别表示什么。这个缺口同时阻塞投影、alpha compositing、densification 和父问题 [[一个 Gaussian 的 SH 与 opacity 能否在同一 footprint 内表达不同颜色]]，因此先单独处理。

## 第一次闭卷回答

> 什么叫2D Gaussian：像素位于 footprint 的哪里 → 权重多大？权重指什么，这里的意思是说，我把高斯投影到了平面上，形成了一个二维的椭球，这个椭球就是footprint吗？权重是指它占据像素平面的比例吗？我理解你这里说的这个color_pixel += blend_weight * gaussian_color公式，它基本的意思就是说，我计算光线经过每个高斯时损失了多少能量，然后用剩下的能量来混合颜色，那么高阶高阶 SH 表达“换一个观察方向时颜色如何变化”，例如镜面反光；它不表达同一 footprint 左右两侧的不同颜色的意思就是说，我这个投影出来的2D椭球的颜色只受这个角度sh系数的影响，换个角度，我就受另外一些sh系数影响比较多？

## 我卡住的地方

- footprint 是硬边界椭圆，还是高斯影响显著的有限屏幕区域。
- `gaussian` 权重是像素面积占比，还是在像素位置对二维高斯函数求值。
- `alpha`、`blend_weight` 和 `transmittance` 的关系。
- 高阶 SH 是切换不同系数，还是固定系数在不同方向基函数值下重新加权求和。

## 最小真实代码证据

`kernels_forward.cuh`：

```cpp
const float2 delta = collected_mean2d[j] - pixel;
const float exponent = -0.5f * quadratic_form(conic, delta);
const float gaussian = expf(fminf(exponent, 0.0f));
const float alpha = opacity * gaussian;
const float blend_weight = transmittance * alpha;

color_pixel += blend_weight * gaussian_color;
transmittance *= 1.0f - alpha;
```

`sh_utils.cuh`：

```cpp
float3 result = 0.5f + C0 * sh_coefficients_0[primitive_idx];
const float3 direction = normalize(position - cam_position);
result += SH_basis(direction) * sh_coefficients_rest;
```

## Codex 教学内容

- 3D Gaussian 经相机投影后，在屏幕上形成二维椭圆形的高斯分布。它没有天然的硬边界；中心影响最大，向外连续衰减。工程实现会按阈值截断出一个有限椭圆包围区域，并找出它覆盖的 tile。这个实际参与计算的屏幕区域通常称为 footprint。
- 对一个像素中心 `p`，kernel 计算它相对二维高斯中心的偏移 `delta`，再求 `G(p)=exp(-1/2·deltaᵀΣ₂D⁻¹delta)`。`gaussian` 就是这个值：中心接近 1，离中心越远越接近 0。它不是该高斯占据像素面积的比例，而是该高斯在这个像素位置的空间影响强度。
- `alpha(p)=opacity×G(p)`：把高斯自身 opacity 与该像素处的空间影响结合。
- `blend_weight=T×alpha`：`T` 表示到达当前高斯前还剩多少未被前方高斯遮挡的透过率。当前高斯对像素颜色的实际系数是 `T×alpha`。随后 `T←T(1-alpha)`，为更后面的高斯保留剩余透过率。
- “能量损失”可以作为初步直觉，但这里更准确的是前向 alpha compositing 的透过率/可见性权重，不应当直接当成严格的物理光能模拟。
- 一个高斯的 SH 系数不会随视角更换。变化的是由观察方向 `(x,y,z)` 计算出的各个 SH 基函数值；所有已激活的固定系数与这些基函数值重新加权求和，得到该视角下的一个 RGB。

## Codex 费曼追问

> 现在关闭资料，不看上面的解释，用你自己的话重新解释一次：一个投影后的 2D Gaussian 为什么不是有硬边界的实心椭圆？某像素处的 `gaussian`、`alpha` 和 `blend_weight` 分别由什么决定，它们为什么都不是“占据像素面积的比例”？

## 我的补充回答

---

## 暴露出的子问题

暂无。

## 最终闭卷解释

> 等待用户闭卷复述。

## Feynman Test

### What

### Why

### How

### Example

### Common Misunderstanding

## 涉及知识

暂不提取 Concept。

## 项目关联

- [[基于 Faster-GS 的快速训练优化与海光 DCU 适配]]

## 视频价值

possible

## 学习结果

- Feynman Pass: false
- Remaining gaps: 等待教学后的闭卷复述。
