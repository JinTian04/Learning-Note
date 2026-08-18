# Learning Dashboard

> 这里是日常入口。问题驱动知识生长，知识图谱只是学习自然产生的副产品。

## 现在怎么开始

1. 直接向 Codex 提出一个具体、可以回答的问题；也可以从 [[Feynman Question Template]] 创建笔记，保存到 `00_Inbox/`。
2. 关闭资料，先亲自写下第一次回答。
3. Codex 进入 Feynman Examiner Mode，每轮只追问 1～3 个最关键问题。
4. 如果暴露基础缺口，先解决 Sub Question，再返回主问题。
5. 通过后由 Codex 维护 Concept、Wiki Links、Mastery Evidence 和 Video Candidate。

如果只是记录一天接触过的学习材料，仍可使用 [[Learning Note Template]]。

---

## ❓ Active Questions

- [[COLMAP 如何从图像匹配恢复三维结构并优化相机位姿]] — 已记录第一次闭卷解释；主问题暂时停在“两视图初始化”断点。
- [[AI IP Studio 的 Harness 如何把一次内容任务变成可持续且可诊断的工作流]] — 主问题暂停；先建立一次 Harness 调用的完整顺序。

---

## 🌱 Sub Questions

- [[一次 Harness 调用从用户任务到最终交付经历什么顺序]] — 当前前置问题；已进入 Teaching Mode，学习后需要闭卷复述。
- [[两张图像如何从 2D-2D 匹配恢复相对位姿并三角化初始 3D 点]] — 已完成本质矩阵估计与位姿分解；继续实际三角化和初始3D点过滤。
- [[为什么可以先估计 E 再分解 R t]] — 当前阻塞问题；区分直接估计 `E`、SVD 的三个用途与后续恢复 `R,t`。
- [[Insta360 X5 视频抽帧后如何为 COLMAP 选择相机模型并诊断内参问题]] — 已记录真实实验；正在确认输入投影、模型差异与质量证据。

---

## ✅ Passed Questions

- [[矩阵乘法在 x2T E x1 中具体做了什么运算]] — 已能从数据形状、逐项乘加、叉乘和平面法向量解释 `x2T E x1 = 0`。
- [[为什么本质矩阵只有 5 个自由度]] — 已能解释旋转3自由度、平移方向2自由度及尺度不可观察性。
- [[SVD 如何把矩阵变换拆成旋转缩放旋转]] — 已能解释输入方向对齐、轴向缩放、输出方向定向三段作用，并区分候选旋转。
- [[如何从本质矩阵分解相对位姿并选择正确解]] — 已能解释零奇异值、`±t`、`R1/R2`、四组候选与正深度选解。

---

## 🔥 我正在学习

当前还没有 Concept。完成第一次真实 Learning Note 后再建立。

---

## 🧠 还没有通过费曼测试

当前为空。这里只列 `status: learning` 且 `feynman_pass: false` 的重要 Concept。

---

## 🔧 我会用，但讲不明白

当前为空。这里只列有实践证据、`mastery >= 3` 且 `feynman_pass: false` 的 Concept。

---

## 🎥 可以考虑做视频

当前为空。优先列 `feynman_pass: true` 且 `video_value: possible` 或 `strong` 的问题。

---

## 📥 最近的 Learning Notes

当前为空。新笔记保存在 `00_Inbox/`。

---

## 🧪 当前项目

- [[Insta360 X5 视频抽帧 COLMAP 重建实验]] — 已比较 `PINHOLE` 与 `OPENCV` 并调整焦距初值；因果诊断仍在进行。
- [[DSH × Codex 的 AI 工作流与 IP 世界验证]] — 已明确 Studio 与 Harness 两条研究线；正在建立第一次闭卷架构基线。

---

## Mastery 速查

- `0 — Awareness`：知道它存在。
- `1 — Understand`：在资料或提示帮助下理解它解决什么问题。
- `2 — Explain`：能闭卷解释，并通过一轮费曼追问。
- `3 — Apply`：有项目、代码、实验或 Debug 的实践证据。
- `4 — Teach`：能完整教学并回答连续追问，适合公开输出。

## 工作区

- `00_Inbox/`：Active Questions、Sub Questions、Learning Notes
- `01_Concepts/`：通过真实学习沉淀的核心概念
- `02_Projects/`：项目实践证据
- `03_Videos/`：视频想法、提纲与脚本
- `98_Archive/`：旧系统与历史内容，只归档不删除
- `99_Templates/`：日常模板
