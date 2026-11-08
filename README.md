# w-token-efficient

English | [简体中文](README.zh-CN.md)

A Claude Code skill that keeps token use low on multi-step work: map-first reads, scoped output, bounded retry loops, compact discipline, and short final reports.

## The problem

On long tasks, tokens mostly leak in five places:

1. Reading files that are not needed
2. Old conversation history re-carried on every turn
3. Retrying failures with no limit
4. Long, noisy command output
5. Recapping everything that was done at the end

This skill does not change *what* gets done. It only constrains *how* to read, split, stop, compact and report.

## When it applies

Use it for multi-step tasks that read or write files or a repository, spawn subagents, run retries or unattended loops, or when a session has already become long. One-off small questions do not need it.

## Rules at a glance

| # | Rule | Gist |
|---|---|---|
| 1 | Map first, files on demand | Start from the file tree and entry points; don't read all of `src/` just to "get familiar" |
| 2 | Scope every read and output | Use explicit paths; skip `node_modules`, build output and logs; cap command output with `head`, `tail`, `-n`, `--stat`; use Read with offset and limit |
| 3 | Compact at breakpoints | See "Compact discipline" below |
| 4 | Subagents only when useful | Independent tasks that don't read the same files; start with 2-3; ask for conclusions and file paths only |
| 5 | Always bound loops | Set max attempts and a time or budget cap up front; stop after the same step fails 3 times in a row and report; never edit tests just to make them pass |
| 6 | Right-size model and plan | Smaller model for simple edits; write a change list first for larger changes |
| 7 | Report briefly | The result plus one next step |

## Compact discipline

- One task per session. For unrelated new work, use `/clear` or a new session rather than `/compact`.
- Breakpoints: a subtask is done, tests pass, a failed approach is abandoned, or you are about to move to a different part of the repo.
- **Claude cannot run `/compact` and cannot see the context percentage.** So at a breakpoint in a long session it only says, in one line, that now is a good time to compact, together with suggested keep/drop text. It does not repeat the reminder at the same breakpoint and does not interrupt mid-subtask.
- Before compacting, write any state that must survive to a notes file, TODO list or commit, not only in the chat.
- What you can do on your side:
  - Check usage with `/context` or a status line; compact around 70%. Past about 90%, quality is already degraded.
  - Add a `# Compact instructions` section to `CLAUDE.md` listing what to keep and what to drop.
  - Set `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=70` so auto-compact fires earlier (an issue report says it can only lower the threshold, never raise it).

## Install

Run this in a terminal, replacing the placeholder with your repository URL:

```bash
git clone <your-repo-url> ~/.claude/skills/w-token-efficient
```

The folder name must match the `name` field in `SKILL.md` (`w-token-efficient`). To use it in a single project only, put it under that project's `.claude/skills/w-token-efficient/` instead.

## Usage

No manual invocation is needed; Claude loads the skill when a task matches its description. You can also say "follow the w-token-efficient rules".

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The rules; loaded when the skill triggers |
| `examples.md` | One worked example (fixing a CSV export date bug); read only when needed |
| `README.md` / `README.zh-CN.md` | For humans; not loaded by Claude automatically |

## Known limitations

- It has not been compared against a baseline on real tasks, so there is no data on how many tokens it saves.
- Trigger accuracy has not been tested with real prompts.
- The `# Compact instructions` section and `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` come from public articles and issue reports and were not tested for this skill.
- "Write a change list first when touching more than about 3 files" is a rule-of-thumb threshold; adjust it to taste.
- A skill is only text. It cannot monitor context usage or run commands for you. For a real automatic reminder you need a status line or a hook.

## References

Ideas were gathered from these public projects and articles:

- [context_guard](https://github.com/kyuna0312/context_guard): auto-compact configuration and status line idea
- [pro-workflow / context-optimizer](https://openskillindex.com/skills/rohitg00-pro-workflow-context-optimizer): the 70% / 90% compact thresholds
- [skill-compressor](https://github.com/David0Ming/skill-compressor): method for shrinking a skill's own size
- [23 Tips for Smart Claude Code Token Saving](https://www.analyticsvidhya.com/blog/2026/05/tips-for-claude-code-token-saving/): compact instructions and the auto-compact threshold

## License

To be added by the repository owner.
