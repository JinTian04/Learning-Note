# Learning Note

这是一个由真实问题驱动、以费曼学习为核心的长期学习系统。

## 核心原则

- 用户负责提出问题、闭卷回答、承认知识缺口和重新解释。
- Codex 负责 Examiner、Learning Coach、Knowledge Architect 和 Obsidian Librarian。
- Question 驱动 Concept、Project、Video 和 Wiki Links 自然生长。
- 对话负责当前推理，Vault 文件负责跨任务、跨设备的长期记忆。
- 不把流畅的 AI 笔记误认为用户已经理解。

完整工作协议见 [AGENTS.md](AGENTS.md)，当前状态见 [Dashboard.md](Dashboard.md)。

## 新电脑恢复

```powershell
git clone https://github.com/JinTian04/Learning-Note.git
```

1. 在 Obsidian 中将克隆后的 `Learning-Note` 文件夹作为 Vault 打开。
2. 在 Codex 中将同一文件夹作为工作区打开。
3. 对 Codex 说：

   > 阅读根目录 AGENTS.md、Dashboard.md 和当前 Active Question，恢复学习状态并继续。不要依赖旧对话记忆。

4. 开始学习前先运行 `git pull --ff-only`；完成一次学习或重要整理后提交并 `git push`。

`.obsidian/` 不进入版本控制，因此每台电脑可以保留自己的 Obsidian 界面配置。

## 长期记忆策略

- 正在进行的问题保存在 `00_Inbox/`。
- 稳定知识保存在 `01_Concepts/`。
- 项目实践证据保存在 `02_Projects/`。
- 视频输出保存在 `03_Videos/`。
- 规则保存在 `AGENTS.md`，当前状态保存在 `Dashboard.md`。
- 一个问题结束后可以结束对应对话；只需确保最终解释、遗留缺口、关联和证据已经写回 Vault。

