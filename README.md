# migrate-conversations-to-qoder

一个 **Qoder Skill**：把任意来源导出的对话 Markdown（小米 MiMo、ChatGPT、Claude、或任何能导出成 `.md` 的 AI 工具）**批量迁进 Qoder**，每个对话成为一个**能接着聊的独立会话**，并尽量保留原标题。

An LLM-driven workflow skill — no runtime dependencies. All logic lives in [`SKILL.md`](./SKILL.md); Qoder CLI reads it and drives the builtin MCP tools (`create_chat_session` / `list_chat_sessions` / `read_chat_session` / `wait_chat_sessions`).

## 它解决什么

Qoder 官方「数据导入」只认自家 Quest / QoderWork；第三方工具（如 SessionHarbor）能读 MiMo、写 Claude Code JSONL，但**没有 Qoder 适配器、也不迁记忆**。本技能改走「导出 Markdown → 逐个 `create_chat_session` 种子会话」的路子，让每个历史对话在 Qoder 里都变成一个可续聊的独立会话。

## 安装

Clone 到用户级技能目录（任何项目可用）：

```bash
git clone https://github.com/YTZ-create/migrate-conversations-to-qoder.git \
  ~/.qoder-cn/skills/migrate-conversations-to-qoder
```

或项目级：clone 到 `<项目>/.qoder/skills/migrate-conversations-to-qoder`。

装好后运行 `/skills reload`（或重启会话），用 `/skills list` 确认能看到它。

## 使用

**必须在目标项目文件夹里打开 Qoder 并新建对话执行**（`create_chat_session` 不能指定目录，会话会落在当前工作区）。

```
/migrate-conversations-to-qoder <导出目录> [索引文件]
```

- `<导出目录>`：一个对话一个 `.md` 的文件夹。
- `[索引文件]`（可选）：含 `标题 | 文件名 | 消息数` 的清单；没有则技能会扫目录自动生成并让你确认。

## 关键约束（技能已内建应对）

- `create_chat_session` **只有 `prompt` 一个入参，无 title、无重命名工具** → 标题只能「偏置」不能精确设定。技能把目标标题放种子 prompt **首行**并声明勿概括，以提高命中率。
- 创建的会话**无法用工具删除** → 技能强制**先建最小样板、验证通过再批量**，避免堆出删不掉的错名会话。
- 收尾产出 `sessionId → 目标标题` **对照表**，供在界面手动重命名（同名靠列表位置区分，**从下往上改**）。
- 附注了「App 显示标题 ≠ 数据库 `title`」这个坑：有些 App（如 MiMo）允许用户在 UI 改名、之后又用自动生成的长句覆盖库里的 `title`，此时以用户 UI/截图标题为准。

## License

[MIT](./LICENSE) © 2026 YTZ-create
