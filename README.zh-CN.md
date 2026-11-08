# w-token-efficient

[English](README.md) | 简体中文

一个让 Claude Code 在多步骤任务中少花 token 的 skill：先建地图再读文件、限定读取和输出范围、给重试设上限、在合适的断点提醒 compact、最后只做简短汇报。

## 解决什么问题

长任务里 token 主要浪费在五个地方：

1. 读了用不到的文件
2. 旧对话历史每一轮都被重复带上
3. 失败后无上限地重试
4. 命令输出太长、太杂
5. 结束时复述一遍做过的事

这个 skill 不改变“做什么”，只约束“怎么读、怎么拆、什么时候停、什么时候 compact、怎么汇报”。

## 什么时候会触发

适用于：读写文件或代码仓库的多步骤任务、需要开子代理、需要重试或无人值守循环，以及会话已经变得很长的时候。一次性的小问题不需要它。

## 规则概览

| # | 规则 | 要点 |
|---|---|---|
| 1 | 先建地图，按需读文件 | 先看文件树和入口，不要为了“熟悉项目”读完整个 `src/` |
| 2 | 限定读取和输出 | 用明确路径；排除 `node_modules`、构建产物、日志；命令输出加 `head`、`tail`、`-n`、`--stat`；Read 用 offset 和 limit |
| 3 | 在断点 compact | 见下方“Compact 纪律” |
| 4 | 子代理只在有用时用 | 任务相互独立、不读同一批文件，先用 2-3 个，只要结论和文件路径 |
| 5 | 循环必须有边界 | 事先定最大尝试次数、时间或预算上限；同一步连续失败 3 次就停下来汇报；不为了通过而改测试 |
| 6 | 模型和计划要匹配 | 简单修改用小模型；改动较大先写变更清单 |
| 7 | 简短汇报 | 只说结果和一个下一步 |

## Compact 纪律

- 一个会话只做一个任务；无关的新任务用 `/clear` 或新会话，而不是 `/compact`。
- 断点包括：子任务完成、测试通过、放弃一条失败的思路、要切换到仓库另一部分之前。
- **Claude 不能自己执行 `/compact`，也看不到上下文百分比。** 所以在长会话的断点处，它只会用一行话提醒你“现在适合 compact”，并附上建议保留和丢弃的内容；同一断点不重复提醒，也不会在子任务中途打断。
- compact 前，把必须保留的状态写进笔记文件、TODO 或 commit，不要只留在聊天里。
- 用户侧可以自己配合：
  - 用 `/context` 或状态栏查看占用，大约 70% 时 compact；超过约 90% 质量已经下降。
  - 在 `CLAUDE.md` 里加一段 `# Compact instructions`，写明要保留和丢弃什么。
  - 设置 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=70` 让自动 compact 更早触发（有 issue 反馈它只能调低阈值，不能调高）。

## 安装

在终端运行，把占位符换成你的仓库地址：

```bash
git clone <你的仓库地址> ~/.claude/skills/w-token-efficient
```

文件夹名要和 `SKILL.md` 里的 `name`（`w-token-efficient`）一致。如果只想在某个项目里用，可以放到该项目的 `.claude/skills/w-token-efficient/`。

## 使用

安装后无需手动调用，Claude 会在符合描述的场景下自动加载。也可以直接说“按 w-token-efficient 的规则来做”。

## 文件说明

| 文件 | 作用 |
|---|---|
| `SKILL.md` | 规则本体，触发时加载 |
| `examples.md` | 一个完整的应用示例（修复 CSV 导出日期 bug），需要时才读 |
| `README.md` / `README.zh-CN.md` | 给人看的说明，Claude 不会自动加载 |

## 已知限制

- 没有在真实任务上和基线对比过，节省多少 token 没有数据。
- 触发准确率没有用真实 prompt 测过。
- `# Compact instructions` 和 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 的做法来自公开文章和 issue，没有在本 skill 中实测。
- “改动超过约 3 个文件先写变更清单”是经验阈值，可按需调整。
- skill 只是文字说明，不会监控上下文用量，也不能替你执行命令；想要真正的自动提醒，需要配合状态栏或 hook。

## 参考

整理时参考了这些公开项目和文章：

- [context_guard](https://github.com/kyuna0312/context_guard)：auto-compact 配置和状态栏思路
- [pro-workflow / context-optimizer](https://openskillindex.com/skills/rohitg00-pro-workflow-context-optimizer)：70% / 90% 的 compact 阈值
- [skill-compressor](https://github.com/David0Ming/skill-compressor)：压缩 skill 本身体积的方法
- [23 Tips for Smart Claude Code Token Saving](https://www.analyticsvidhya.com/blog/2026/05/tips-for-claude-code-token-saving/)：Compact instructions 与自动 compact 阈值

## License

由仓库所有者自行添加。
