---
type: question
status: learning
feynman_pass: false
mastery_evidence: none
video_value: none
related_concepts:
parent_question: "[[AI IP Studio 的 Harness 如何把一次内容任务变成可持续且可诊断的工作流]]"
created: 2026-08-17
---

# 一次 Harness 调用从用户任务到最终交付经历什么顺序？

## 为什么会问这个问题？

用户已经能用大脑、职位、手法、工具、协议和多人协作来类比 Harness 的组成部分，但还没有形成一次调用的先后顺序，因此暂时无法根据运行记录定位故障。

这个前置问题通过后，返回父问题中的“出生地错误”诊断案例。

---

## 第一次闭卷回答

> 在回答这个问题之前，我希望你带我过一遍整个Harness系统的调用流程，我还没有建立过一个调用顺序的印象

---

## 我卡住的地方

- 不知道任务、Session、Agent、Model、Skill、Tool/MCP、Workflow 与产物之间的调用先后。
- 因此还不能把 Session 中的证据还原成一条因果链。

---

## Codex 教学内容

> 以下内容由 Codex 在 Teaching Mode 中提供，不算作用户的 Feynman Pass。

### 一条最小 Harness 主循环

1. **启动与装配**：Harness 根据配置装载 Model 适配器、Agent、Skill、Tool/MCP、Session 存储、权限与 Workflow。
2. **接收任务**：创建或恢复 Session，把用户任务写入记录；Agent Loop 认领本轮输入。
3. **组装上下文**：根据 Agent 身份选择系统指令与 Skill，从 Session 派生历史，并通过检索或工具取得本轮需要的世界观资料；同时向 Model 声明可用 Tool。
4. **调用 Model**：Model 只看到本次请求中实际提供的内容，返回文本、判断或 Tool Call。
5. **执行 Tool Call**：Harness 校验参数、权限与审批，再调用本地 Tool；如果能力来自外部程序或服务，可以通过 MCP 连接。调用请求、原始结果和错误都写入 Session。
6. **继续 Step**：Tool 结果重新进入上下文，再次调用 Model。Model 与 Tool 可以往返多次，直到 Agent 给出本阶段结果。
7. **推进 Workflow**：保存并版本化写手产物；Workflow 再把它交给审核 Agent、正史检查 Agent 或其他角色。每个 Agent 内部仍运行同样的 Model ↔ Tool 循环。
8. **验证与结束**：检查验收条件；通过则交付并写入 checkpoint，失败则根据策略重试、回退、转交人工或开始下一轮。Session 保留恢复和诊断所需的事件。

### DSH 中的微观顺序

一次 `Turn` 可以包含多个 `Step`；一个 `Step` 的核心是一轮 Model 请求及其 Tool 调用：

`turn/start → claim input → assemble prompt/tool schemas → step/start → model request → tool call/result → step/end → 必要时下一 Step → turn/end`

### 三个容易混淆的边界

- **Session 主要保存状态与证据**，不是负责审核的角色。
- **Workflow 决定阶段和 Agent 的先后关系**，每个 Agent 内部仍由 Agent Loop 驱动 Model 与 Tool。
- **Harness 不只是零件总称**；它负责装配、调度、上下文组装、权限、记录、恢复和验证，使零件成为一个可运行系统。

### 最小诊断链

`任务输入 → 实际提供给 Model 的上下文 → Model 输出/Tool Call → Tool/MCP 原始结果 → Agent 产物 → 审核输入与结论 → 最终交付`

定位错误时，沿这条链逐段比较“应该出现什么”和“实际记录了什么”。

---

## Codex 费曼追问

教学完成后，关闭资料，用自己的话重新解释：

> 一个写手 Agent 接到“根据角色设定写短篇故事”的任务后，从 Harness 收到任务开始，到故事交给审核 Agent 为止，按顺序发生了什么？

---

## 我的补充回答

---

## 暴露出的子问题

- 等待用户重新解释后判断。

---

## 最终闭卷解释

> 等待学习和追问完成后，由用户闭卷解释。

---

## Feynman Test

### What

### Why

### How

### Example

### Common Misunderstanding

---

## 涉及知识

暂不创建 Concept；等待本 Question 通过并在真实工作流中验证。

---

## 项目关联

- [[DSH × Codex 的 AI 工作流与 IP 世界验证]]

---

## 视频价值

none

---

## 学习结果

- Feynman Pass: false
- Remaining gaps: 等待用户闭卷复述调用顺序。
