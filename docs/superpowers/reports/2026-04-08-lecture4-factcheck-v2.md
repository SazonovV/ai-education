# Lecture 4 Fact-Check v2 — § 3.4 & § 5

**Date:** 2026-04-08
**Scope:** § 3.4 (tools comparison table) + § 5 (hooks)
**Baseline:** 2026-03-31-lecture4-factcheck.md
**Status:** IN PROGRESS

## § 3.4 — Tools comparison table

### Criterion: Модель субагентности

- **Claim:** "Явный spawn через Agent tool" (part4_subagents_hooks.md:299)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The underlying tool is confirmed as "Agent tool" (renamed from "Task tool" in v2.1.63 — old `Task(...)` references still work as aliases). However, the primary invocation model is *automatic delegation* by Claude based on subagent descriptions, not explicit spawn; explicit invocation is an opt-in via @-mention or `--agent` flag, not the default mode.
  - **Recommendation:** "Автоматическая делегация через Agent tool (явный вызов через @-mention)"

<entries to be filled in Task 2>

### Criterion: Конфигурация

<entries to be filled in Task 3>

### Criterion: Параллельность

- **Claim:** "Да (несколько Agent-вызовов)" (part4_subagents_hooks.md:301)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The docs explicitly document running multiple subagents in parallel: "For independent investigations, spawn multiple subagents to work simultaneously." Background subagents run concurrently while the user continues working. The tool was renamed from `Task` to `Agent` in v2.1.63 ("In version 2.1.63, the Task tool was renamed to Agent"), so the lecture's phrasing "Agent-вызовов" is now the correct name. Both foreground (blocking) and background (concurrent) invocations are supported.
  - **Recommendation:** —

- **Claim:** "Последовательная (Boomerang)" (part4_subagents_hooks.md:301)
  - **Status:** VERIFIED
  - **Source:** https://docs.roocode.com/features/boomerang-tasks
  - **Evidence:** The Boomerang Tasks model is explicitly sequential: a parent task pauses, the subtask executes in a specialized mode, and upon completion the parent resumes with only the subtask's summary. The documentation describes a clear pause-and-resume mechanic with no mention of parallel subtask execution. Kilo Code inherits the same Boomerang Task architecture from Roo Code; the Task Manager UI adds visual stack navigation but does not add parallel execution.
  - **Recommendation:** —

- **Claim:** "Зависит от реализации" for OpenCode (part4_subagents_hooks.md:301)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://opencode.ai/docs/agents/
  - **Evidence:** OpenCode agents are configuration profiles (different system prompt, model, tools, permissions) that activate one at a time within a session. The docs describe switching between agents via `@` mention or cycling through child sessions serially (`session_child_cycle`). There is no native built-in mechanism to spawn multiple agents concurrently. Third-party tooling (e.g., running multiple `opencode` processes in parallel git worktrees) can simulate parallelism externally, but that depends on external orchestration, not OpenCode's own agent system. The vague "Зависит от реализации" is technically defensible (parallelism is possible via external scripting), but misleadingly implies the feature is present in some implementations when it is actually absent natively.
  - **Recommendation:** "Нет (агенты — профили конфигурации, параллельный запуск требует внешней оркестровки)"

### Criterion: Изоляция

<entries to be filled in Task 5>

### Criterion: Кастомные типы

<entries to be filled in Task 6>

### Criterion: Фоновый запуск

- **Claim:** "`run_in_background: true`" (part4_subagents_hooks.md:304)
  - **Status:** INACCURATE
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The current frontmatter field name is `background: true`, not `run_in_background: true`. The docs list `background` as an optional field: "Set to `true` to always run this subagent as a background task. Default: `false`." The `--agents` JSON flag documentation also uses `background`. No mention of `run_in_background` anywhere in current docs.
  - **Recommendation:** "`background: true`"

- **Claim:** "Нет" for Roo Code / Kilo Code (part4_subagents_hooks.md:304)
  - **Status:** VERIFIED
  - **Source:** https://docs.roocode.com/features/boomerang-tasks
  - **Evidence:** Boomerang Tasks are explicitly sequential: "The parent task pauses, and the new subtask begins in a different, specialized mode. When the subtask's goal is achieved, Roo signals completion. The parent task resumes." No background, async, or concurrent task execution is documented. GitHub releases also contain no mention of a background execution feature.
  - **Recommendation:** —

- **Claim:** "Нет" for OpenCode (part4_subagents_hooks.md:304)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/agents
  - **Evidence:** OpenCode agent documentation describes only synchronous, interactive, session-based workflows. No background execution, async mode, or fire-and-forget agent feature is mentioned. The General subagent can run multiple units of work in parallel within a session, but there is no background/detached execution model comparable to Claude Code's `background: true`.
  - **Recommendation:** —

### Post-table paragraph

<entry to be filled in Task 8>

## § 5 — Hooks

### § 5.1 Brief reminder

<entries to be filled in Task 9>

### § 5.2 Advanced scenarios (7 scenarios)

<entries to be filled in Task 9>

### § 5.3 Claude Code — advanced configs

<entries to be filled in Task 10>

### § 5.4 OpenCode — plugins

<entries to be filled in Task 11>

### § 5.5 Roo Code / Kilo Code — emulation

<entries to be filled in Task 12>

### § 5.6 Hooks comparison table

<entries to be filled in Task 13>

### § 5.7 Selection strategy

<entries to be filled in Task 14>

## Summary

<to be filled in Task 15>

## Delta vs 2026-03-31 baseline

<to be filled in Task 16>
