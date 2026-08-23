---
type: project
status: active
---

# 基于 Faster-GS 的快速训练优化与海光 DCU 适配

## 项目目标

把 3DGS 的训练加速理解成一条完整链路，而不是若干互不相关的技巧：

```text
渐进分辨率降低像素开销
→ 高斯增长预算控制表示规模
→ Top-K clone/split 把有限容量给更需要的高斯
→ Faster-GS 降低 Gaussian–Tile 光栅化成本
→ CUDA/HIP 异构适配保证不同 GPU 执行模型下的正确性
→ 用训练时间、质量、显存和稳定性实验验证结果
```

对外技术定位：以 Faster-GS 高性能后端为基础，融合并适配 DashGaussian 类分辨率调度和高斯增长机制，并完成海光 K100_AI DCU/HIP 后端移植与稳定性验证。

## 归因边界

- Faster-GS 的高性能光栅化后端来自公开项目；本项目对其进行了集成、工程改动和异构平台适配。
- 频率感知渐进分辨率、动态 primitive budget 和 Top-K densification 属于 DashGaussian 的公开机制；本地版本包含适配和二次修改，不能表述为从零原创。
- FastGS 的 VCD、VCP 和 Compact Box 是另一项目的机制；当前本地代码尚未发现对应实现，暂时只作为比较边界，不写入自己的实现贡献。
- 个人贡献重点是组合方案理解、本地二次开发、性能/正确性诊断，以及海光 DCU 落地证据。

## 我实际做了什么？

根据 `C:\Users\zjt50\Desktop\海光机器编译Fastergs后端记录.md` 中保存的工程记录：

- 将 FasterGS CUDA Backend 迁移到海光 K100_AI（gfx928）、PyTorch 2.5.1、HIP 6.1.25012 环境。
- 处理 CUDA/HIP 向量运算符、host/device 数学函数、结构化绑定、CUB/hipCUB 命名空间等编译兼容问题。
- 针对 NVIDIA warp32 与 DCU wave64 的差异实现 `ballot_32`、无符号 lane mask、`nth_set_bit_32` 和逻辑 32-thread tile 内的广播适配。
- 使用哨兵、offset validator、逐 Gaussian 写入区间和串行参考重算，定位 densification 后 VMFault 的真正根因。
- 在 AMD/DCU 路径改用串行精确 tile count，NVIDIA/CUDA 保留 cooperative 路径。
- 完成带 densification 的 4,300 次诊断/发布版训练，以及 30,000 次长期训练。

## 可验证的项目证据

海光训练记录中的最终结果：

```text
初始 Gaussian：20,036
densification：iteration 600 到 14,900
最终 Gaussian：1,737,449
总迭代：30,000
训练时间：1:02:27
吞吐：8.00 it/s
峰值 allocated VRAM：4.71 GiB
TRAIN_EXIT_CODE=0
```

这能证明后端已走通编译、动态加载、前向、反向、densification、checkpoint 和长期训练。它不能替代 RTX 4090 的速度/画质对照实验，也不能证明本地组合方案达到“一分钟训练”。

## 核心故障与因果链

```text
wave64 环境下 cooperative tile count 偶发 ±1
→ prefix sum 使用了错误的实例数量
→ instance 区间过长或过短
→ 产生 0xFFFFFFFF 空洞或跨区间写入
→ 排序结果与 tile range 被污染
→ 后续 blend kernel 读取非法索引
→ 最终表现为 VMFault
```

最终报错发生在 blend 附近，但根因位于更早的精确 tile 数量计算。这是项目中最有价值的 GPU 调试证据。

## 本地算法集成证据

### 频率感知分辨率调度

- `utils/schedule_utils.py` 对训练图像执行 FFT，按累计频率显著性生成分辨率阶段。
- `train.py` 使用 `render_scale` 同时改变渲染尺寸和监督图像尺寸。
- `gaussian_renderer/__init__.py` 根据目标尺寸同步重算宽高、焦距和主点。

### 高斯增长预算与 Top-K 增密

- `TrainingScheduler.get_densify_rate()` 计算目标高斯数量和单轮 densify rate 上限。
- `prune_and_densify()` 先剪枝，再计算本轮 `target_total`。
- clone 先消耗配额，split 只能使用剩余配额。
- 只有同时满足原始梯度/尺寸条件和 Top-K 条件的高斯才会被 clone 或 split。

当前需要谨慎表述：低分辨率通过 `next_n_gaussian` 压低目标总量，但本地第 104 行又让单轮 rate cap 随 `cur_scale` 增大；实际增密率是二者的最小值。不能把它简单说成“分辨率越低，单轮增密越克制”。

## 当前学习主线

1. [[为什么渐进分辨率训练必须与高斯增长预算协同]]
2. FFT 频率显著性如何变成分辨率时间表？
3. `target_total`、Top-K、clone 和 split 如何共同落实增长预算？
4. Gaussian–Tile 流水线的时间和显存到底消耗在哪里？
5. 为什么 tile count 的 ±1 会沿 prefix sum、排序和 blend 放大成 VMFault？
6. 为什么 warp32 代码迁移到 wave64 后不能机械替换 ballot？
7. [[FasterGS 中 loss.backward 如何穿过自定义 CUDA rasterizer 把梯度传回 Gaussian 参数]]
8. 如何设计可写进简历的 RTX 4090/DCU 性能、画质和稳定性对照实验？

学习顺序有意先建立算法—计算量因果链，再进入 kernel 细节。FastGS 的 VCD/VCP 暂不进入主线。

## 仍然不会解释的知识

- FFT 频谱累计量为什么可以决定某个阶段所需的渲染分辨率。
- 本地修改后的 primitive budget 公式与 DashGaussian 原公式有何精确差异。
- Top-K 的 score、原始 clone/split 条件与最终新增数量之间的完整关系。
- Faster-GS 每个优化分别减少了哪种中间数据、同步或原子操作。
- `ballot_32` 已适配后，原 cooperative tile count 为什么仍会偶发 ±1。
- 串行精确 tile count 的正确性收益与性能代价如何测量。
- RTX 4090 上组合方案的训练时间、PSNR、显存和消融结果。

## 简历表述约束

可以使用“融合、适配、二次开发、协同优化、定位并修复”。在没有提交记录或设计证据前，不把 Faster-GS/DashGaussian 的公开机制写成个人原创。

以下数据仍需补齐后才能写成定量性能 bullet：

- 场景和数据集。
- 输入/训练分辨率。
- 总迭代数与 densification 区间。
- 训练时间是否包含初始化、评测和保存。
- RTX 4090 上 baseline 与组合方案的同口径时间、PSNR/SSIM/LPIPS、Gaussian 数量和显存。

## 参考

- [Faster-GS 官方仓库](https://github.com/nerficg-project/faster-gaussian-splatting)
- [Faster-GS 论文](https://arxiv.org/abs/2602.09999)
- [DashGaussian 官方仓库](https://github.com/YouyuChen0207/DashGaussian)
- [DashGaussian 论文](https://openaccess.thecvf.com/content/CVPR2025/html/Chen_DashGaussian_Optimizing_3D_Gaussian_Splatting_in_200_Seconds_CVPR_2025_paper.html)
- [FastGS 官方仓库](https://github.com/fastgs/FastGS)

