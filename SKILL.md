---
name: "w-token-efficient"
description: "Use for multi-step tasks that read/write files or a repo, spawn subagents, or run retries/unattended loops, or when a session is getting long: map-first reads, scoped output, bounded loops, compact discipline, brief reports."
---

# Token-efficient work rules (Token 高效工作法)

Finish the same task with fewer tokens. These rules change HOW you read, split, stop, compact and report, not WHAT you do. Tokens leak through unneeded reads, stale history re-carried every turn, unbounded retries, noisy tool output and recaps.

## Rules

1. **Map first, files on demand.** Start from a short map (file tree + entry points/key interfaces). Read a file only when the task needs it; full re-read only on the first pass or after a big refactor. Not: reading all of `src/` "to get familiar".
2. **Scope every read and output.** Explicit paths/modules; skip node_modules, build output, logs, large data. Cap noisy command output (`head`, `tail`, `-n`, `--stat`, grep with a path); use Read with offset+limit. Not: an unfiltered recursive grep from the repo root.
3. **Compact at breakpoints, not at the wall** (below).
4. **Subagents only when useful.** Independent tasks that don't read the same files; start with 2-3 max; narrow scope each; ask for conclusions + file paths, not pasted code. Each starts cold, so overlapping reads are paid twice.
5. **Always bound loops.** Set up front: max attempts, time/budget cap, stop rule (same step fails 3 times in a row → stop, report what was tried and why it failed). Never edit tests just to make them pass (no commented-out assertions, no "rerun until green").
6. **Right-size model and plan.** Smaller model for simple edits/lookups; for changes touching more than ~3 files write a change list first.
7. **Report briefly.** Result + one next step; no recap of actions.

## Compact discipline

Every turn re-carries the full history, so long context costs more per step and quality drops before the window is full.

- One task per session; unrelated new work → `/clear` or a new session, not `/compact`.
- Breakpoints: subtask done, tests passing, failed approach abandoned, or before moving to another part of the repo.
- Claude can't run `/compact` or see the context percentage. At a breakpoint in a long session (many tool calls, big outputs, several subtasks), say once, in one line, that now is a good time to compact, with preservation text, e.g. `/compact keep: goal, files changed, commands run, failing tests + exact errors, decisions, next steps; drop: old exploration, repeated logs`. Don't repeat until the next breakpoint; never interrupt mid-subtask.
- Before compacting, write must-survive state to a notes file, TODO list or commit, not only chat.
- User side: compact around ~70% (`/context` or a status line shows usage); past ~90% quality is already degraded. A `# Compact instructions` section in CLAUDE.md can list what to keep/drop. `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=70` makes auto-compact fire earlier (reported to only lower the threshold, never raise it).

## Cheap habits

Keep CLAUDE.md short (occasional guidance → skills, one-off workflows → commands); disable MCP servers not needed for the task, each adds passive context.

Worked example: `examples.md`.
