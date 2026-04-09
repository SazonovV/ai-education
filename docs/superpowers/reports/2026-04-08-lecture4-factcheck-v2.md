# Lecture 4 Fact-Check v2 — § 3.4 & § 5

**Date:** 2026-04-08 (Kilo Code verification completed 2026-04-09)
**Scope:** § 3.4 (tools comparison table) + § 5 (hooks)
**Baseline:** 2026-03-31-lecture4-factcheck.md
**Status:** COMPLETE — 15 issues found (7 HIGH, 6 MEDIUM, 2 LOW), ready for content edit phase

## Scope change — 2026-04-08 / 2026-04-09

User decisions during execution:

1. **Roo Code is being removed from the lecture entirely.** All Roo Code claims are marked `OUT OF SCOPE — to be removed from lecture` instead of being verified. The content edit phase will drop § 3.2's "Roo Code" framing, the "Roo Code / Kilo Code" column header, and any other Roo Code references.

2. **Kilo Code stays in the lecture as a standalone tool.** The current "Roo Code / Kilo Code" combined column is relabeled to "Kilo Code". Kilo Code is no longer a direct Roo Code fork — it has diverged significantly (see below). The existing lecture text, which treats Kilo as a Roo clone, is now substantially outdated and needs a full rewrite of § 3.2, § 3.4 Kilo column, § 5.5, § 5.6 Kilo column, and § 5.7 Kilo bullet.

3. **Kilo Code verification (2026-04-09).** On the first pass fetches against `docs.kilocode.ai` / `kilo.ai/docs` hung and the Kilo Code claims were deferred to `2026-04-08-lecture4-factcheck-v2-kilo-deferred.md`. On 2026-04-09 the user re-authorized Kilo Code verification, docs came back online, and all Kilo Code claims were verified against `kilo.ai/docs`. Findings are now integrated into the main report below. The deferred file is kept as a historical record and should not be treated as authoritative.

### Kilo Code divergence summary (2026-04-09)

The lecture describes Kilo Code as a cosmetic fork of Roo Code with an added Task Manager UI. This is no longer accurate:

- **Terminology:** "Modes" → "Agents". The Orchestrator mode still exists but is **deprecated** — "Agents with full tool access (Code, Plan, Debug) now support subagents natively."
- **Built-in set:** old Roo modes `Code / Architect / Ask / Debug / Orchestrator` → Kilo agents `code / plan / ask / debug / orchestrator (deprecated)`. Note **"Architect" is renamed to "plan"**.
- **Config file:** `.roomodes` (JSON) → **`.kilocodemodes`** (YAML preferred, JSON kept for back-compat) for project; `custom_modes.yaml` for global; `.kilo/agents/<name>.md` for the newer Markdown agent format.
- **Groups field:** Roo's `["read", "edit", "command", "browser", "mcp"]` → Kilo uses Claude Code-style tool names `["read", "edit", "bash", "grep", "glob", "list", "task", "webfetch", "websearch", ...]`. The old `command`/`mcp` values are no longer documented.
- **Task Manager UI:** not documented anywhere on kilo.ai/docs. Only residual reference is the `/newtask` slash command in the legacy VSCode extension. The lecture's claim of "визуальная панель с деревом подзадач" is not supported by current docs.
- **Boomerang Tasks:** `https://kilo.ai/docs/features/boomerang-tasks` returns 404. No Boomerang Tasks feature page exists in current docs. The delegation flow survives only as the deprecated Orchestrator agent.
- **Hooks:** no hooks documentation exists at `kilo.ai/docs`. The "Automate" section covers code reviews, MCP, shell integration, agent manager — no native hook / event-trigger system.
- **Parallelism:** not documented. The docs describe sequential agent switching and delegation; no concurrent execution mechanism is mentioned.

This divergence means almost every Kilo-specific claim in the lecture is **OUTDATED**, not merely Roo-labelled. See per-criterion entries below for details.

## § 3.4 — Tools comparison table

### Criterion: Модель субагентности

- **Claim:** "Явный spawn через Agent tool" (part4_subagents_hooks.md:299)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The underlying tool is confirmed as "Agent tool" (renamed from "Task tool" in v2.1.63 — old `Task(...)` references still work as aliases). However, the primary invocation model is *automatic delegation* by Claude based on subagent descriptions, not explicit spawn; explicit invocation is an opt-in via @-mention or `--agent` flag, not the default mode.
  - **Recommendation:** "Автоматическая делегация через Agent tool (явный вызов через @-mention)"

- **Claim:** "Режимы + Orchestrator / Boomerang" for Kilo Code (part4_subagents_hooks.md:299)
  - **Status:** OUTDATED
  - **Source:** https://kilo.ai/docs/code-with-ai/using-modes
  - **Evidence:** Current Kilo Code docs describe built-in agents `code`, `plan`, `ask`, `debug`, and `orchestrator` with orchestrator explicitly marked deprecated: "A strategic workflow orchestrator who coordinates complex tasks by delegating them to appropriate specialized agents … Agents with full tool access (Code, Plan, Debug) now support subagents natively." There is no separate "Boomerang Tasks" feature page (`kilo.ai/docs/features/boomerang-tasks` → 404). The Boomerang Tasks delegation flow survives only as the deprecated Orchestrator agent. Terminology has shifted from "Modes" to "Agents".
  - **Recommendation:** "Агенты + нативные субагенты (Orchestrator deprecated)". Or more descriptive: "Агенты `code` / `plan` / `ask` / `debug` с нативной поддержкой субагентов (Orchestrator — deprecated)".

- **Claim:** "Агенты в конфигурации" for OpenCode (part4_subagents_hooks.md:299)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/agents
  - **Evidence:** OpenCode agents are pre-defined configuration profiles, not dynamically spawned. They live either in `opencode.json` (under the `agents` key) or as markdown files in `.opencode/agents/` (project) / `~/.config/opencode/agents/` (global). Each agent has a `mode` field (`primary` / `subagent` / `all`) and is selected by the user or invoked via `@`-mention — there is no dynamic spawn primitive.
  - **Recommendation:** —

### Criterion: Конфигурация

- **Claim:** "`AGENTS.md` + `subagent_type`" for Claude Code (part4_subagents_hooks.md:300)
  - **Status:** INACCURATE
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** Subagents are not defined in `AGENTS.md`. They are individual Markdown files with YAML frontmatter, stored in `.claude/agents/` (project), `~/.claude/agents/` (user), or a plugin's `agents/` directory. The frontmatter fields include `name`, `description`, `tools`, `model`, `isolation`, `background`, `permissionMode`, `hooks`, etc. `AGENTS.md` is a separate concept — the project-level memory file (the successor to `CLAUDE.md`) — not a subagent registry. The Agent tool's `subagent_type` parameter does exist (it references the subagent definition's `name`), so half the claim is correct.
  - **Recommendation:** "`.claude/agents/<name>.md` (Markdown с YAML frontmatter) + параметр `subagent_type` Agent-тула"

- **Claim:** "`.roomodes` (JSON)" for Kilo Code (part4_subagents_hooks.md:300)
  - **Status:** OUTDATED
  - **Source:** https://kilo.ai/docs/customize/custom-modes ; https://kilo.ai/docs/features/custom-modes
  - **Evidence:** Kilo Code no longer uses `.roomodes`. Current docs state: "Custom modes are stored in `.kilocodemodes` (project-level) or `custom_modes.yaml` (global). YAML is the preferred format, though JSON remains supported for backward compatibility." A newer Markdown-based format also exists: `.kilo/agents/<name>.md`. The `.roomodes` file name is a Roo Code artifact; in Kilo Code it is `.kilocodemodes`.
  - **Recommendation:** "`.kilocodemodes` (YAML) или `.kilo/agents/*.md`"

- **Claim:** "`opencode.json` / `.opencode/agents/`" for OpenCode (part4_subagents_hooks.md:300)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/agents
  - **Evidence:** Both methods are documented: agents can be configured in JSON (`opencode.json`) or as Markdown files in `.opencode/agents/` (project) / `~/.config/opencode/agents/` (global). The two formats are interchangeable.
  - **Recommendation:** —

### Criterion: Параллельность

- **Claim:** "Да (несколько Agent-вызовов)" (part4_subagents_hooks.md:301)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The docs explicitly document running multiple subagents in parallel: "For independent investigations, spawn multiple subagents to work simultaneously." Background subagents run concurrently while the user continues working. The tool was renamed from `Task` to `Agent` in v2.1.63 ("In version 2.1.63, the Task tool was renamed to Agent"), so the lecture's phrasing "Agent-вызовов" is now the correct name. Both foreground (blocking) and background (concurrent) invocations are supported.
  - **Recommendation:** —

- **Claim:** "Последовательная (Boomerang)" for Kilo Code (part4_subagents_hooks.md:301)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://kilo.ai/docs/code-with-ai/using-modes
  - **Evidence:** Current Kilo Code docs do not document parallel subagent execution — agent delegation is described only through the (deprecated) Orchestrator agent, which sequentially delegates to specialized agents. No concurrent execution primitive is mentioned. However, the Boomerang Tasks *name* is gone — `kilo.ai/docs/features/boomerang-tasks` returns 404. The mechanism is still sequential, but the "Boomerang" framing is obsolete.
  - **Recommendation:** "Последовательная (через Orchestrator, deprecated)". Drop the "Boomerang" name.

- **Claim:** "Зависит от реализации" for OpenCode (part4_subagents_hooks.md:301)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://opencode.ai/docs/agents/
  - **Evidence:** OpenCode agents are configuration profiles (different system prompt, model, tools, permissions) that activate one at a time within a session. The docs describe switching between agents via `@` mention or cycling through child sessions serially (`session_child_cycle`). There is no native built-in mechanism to spawn multiple agents concurrently. Third-party tooling (e.g., running multiple `opencode` processes in parallel git worktrees) can simulate parallelism externally, but that depends on external orchestration, not OpenCode's own agent system. The vague "Зависит от реализации" is technically defensible (parallelism is possible via external scripting), but misleadingly implies the feature is present in some implementations when it is actually absent natively.
  - **Recommendation:** "Нет (агенты — профили конфигурации, параллельный запуск требует внешней оркестровки)"

### Criterion: Изоляция

- **Claim:** "Worktree (git-копия)" for Claude Code (part4_subagents_hooks.md:302)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The `isolation` frontmatter field is documented: "Set to `worktree` to run the subagent in a temporary git worktree, giving it an isolated copy of the repository. The worktree is automatically cleaned up if the subagent makes no changes." This is a real, supported feature, not aspirational.
  - **Recommendation:** —

- **Claim:** "Отдельный контекст (общая FS)" for Kilo Code (part4_subagents_hooks.md:302)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://kilo.ai/docs/code-with-ai/using-modes
  - **Evidence:** Kilo Code docs do not describe any worktree-style isolation — agents run against the project's shared filesystem. Per-agent permission is governed by the `groups` field (tool-category gating), not by a separate copy of the repository. The claim "Отдельный контекст (общая FS)" remains accurate in spirit, though the lecture's justification (Roo Code ancestry) no longer holds cleanly since Kilo has diverged.
  - **Recommendation:** —

- **Claim:** "Отдельный контекст (общая FS)" for OpenCode (part4_subagents_hooks.md:302)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/agents
  - **Evidence:** OpenCode docs do not describe any worktree-style isolation; agents share the project filesystem. Access is mediated through the per-agent `permission` field (`ask` / `allow` / `deny` per tool/path), not through a separate copy of the repository.
  - **Recommendation:** —

### Criterion: Кастомные типы

- **Claim:** "`AGENTS.md`" for Claude Code (part4_subagents_hooks.md:303)
  - **Status:** INACCURATE
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** Custom subagent types are defined as Markdown files inside `.claude/agents/` (project), `~/.claude/agents/` (user), or a plugin's `agents/` directory — not in `AGENTS.md`. The file name (without `.md`) becomes the subagent's `name`, which is what the Agent tool's `subagent_type` parameter references. Same root cause as the "Конфигурация" inaccuracy: the lecture conflates `AGENTS.md` (project memory file) with the `.claude/agents/` directory (subagent registry).
  - **Recommendation:** "`.claude/agents/<name>.md`"

- **Claim:** "`.roomodes`" for Kilo Code (part4_subagents_hooks.md:303)
  - **Status:** OUTDATED
  - **Source:** https://kilo.ai/docs/customize/custom-modes
  - **Evidence:** Same root cause as the "Конфигурация" row — Kilo Code uses `.kilocodemodes` (YAML) for project, `custom_modes.yaml` for global, or `.kilo/agents/<name>.md` for the newer Markdown format. `.roomodes` is the Roo Code file name and is not authoritative for Kilo Code.
  - **Recommendation:** "`.kilocodemodes` или `.kilo/agents/*.md`"

- **Claim:** "Агенты в конфиге" for OpenCode (part4_subagents_hooks.md:303)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/agents
  - **Evidence:** Custom agent definitions are first-class — added to the `agents` block of `opencode.json` or as additional Markdown files in `.opencode/agents/`. There are no built-in subagent "types" per se; every agent is user-defined and labelled by `mode: primary | subagent | all`.
  - **Recommendation:** —

### Criterion: Фоновый запуск

- **Claim:** "`run_in_background: true`" (part4_subagents_hooks.md:304)
  - **Status:** INACCURATE
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The current frontmatter field name is `background: true`, not `run_in_background: true`. The docs list `background` as an optional field: "Set to `true` to always run this subagent as a background task. Default: `false`." The `--agents` JSON flag documentation also uses `background`. No mention of `run_in_background` anywhere in current docs.
  - **Recommendation:** "`background: true`"

- **Claim:** "Нет" for Kilo Code (part4_subagents_hooks.md:304)
  - **Status:** VERIFIED
  - **Source:** https://kilo.ai/docs/code-with-ai/using-modes ; https://kilo.ai/docs/automate
  - **Evidence:** Kilo Code docs describe only synchronous, interactive agent switching and delegation. The Automate section covers code reviews, MCP, shell integration, and an Agent Manager, but no background/async/detached agent execution mode is documented anywhere. The claim "Нет" is accurate.
  - **Recommendation:** —

- **Claim:** "Нет" for OpenCode (part4_subagents_hooks.md:304)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/agents
  - **Evidence:** OpenCode agent documentation describes only synchronous, interactive, session-based workflows. No background execution, async mode, or fire-and-forget agent feature is mentioned. The General subagent can run multiple units of work in parallel within a session, but there is no background/detached execution model comparable to Claude Code's `background: true`.
  - **Recommendation:** —

### Post-table paragraph

- **Claim:** "Самая продвинутая модель субагентности — у Claude Code: полная изоляция через worktree, параллелизм и фоновый режим. Roo Code / Kilo Code берут удобством Orchestrator-паттерна и визуализацией в Kilo. OpenCode — минимальный, но достаточный подход для проектов, где нужны предустановленные профили без динамического порождения агентов." (part4_subagents_hooks.md:306)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** Composite — https://code.claude.com/docs/en/sub-agents (CC), https://opencode.ai/docs/agents (OpenCode), https://kilo.ai/docs/code-with-ai/using-modes (Kilo)
  - **Evidence:** Claude Code part holds (`isolation: worktree`, multiple Agent calls, `background: true`). OpenCode part holds (configuration-profile agents). The Kilo Code framing is OUTDATED: Orchestrator is deprecated, there is no documented Task Manager UI, and the "визуализация" argument no longer has a source. Kilo's current positioning is "agents with native subagent support" but without worktree isolation, parallelism, or background mode.
  - **Recommendation:** Rewrite to drop Roo Code and reframe Kilo Code as a CLI/VSCode agent with delegation via deprecated Orchestrator. Suggested replacement: "Claude Code предлагает наиболее зрелую модель субагентности: изоляцию через worktree, параллельный запуск и фоновый режим. OpenCode идёт по пути предустановленных профилей — минимально, но достаточно для проектов, где субагенты не порождаются динамически. Kilo Code — промежуточный вариант: агенты `code` / `plan` / `ask` / `debug` с нативной поддержкой субагентов, но без worktree-изоляции, параллелизма и фонового режима."

## § 5 — Hooks

### § 5.1 Brief reminder

- **Claim:** "Hook — реакция на событие жизненного цикла агента, выполняемая **хостом** (средой исполнения), а не моделью." (part4_subagents_hooks.md:428)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Claude Code docs state: "Hooks are user-defined shell commands, HTTP endpoints, or LLM prompts that execute automatically at specific points in Claude Code's lifecycle… Hooks execute on the host machine (where Claude Code runs), not in the model." The host-vs-model framing is exact.
  - **Recommendation:** —

- **Claim:** "Модель не решает, запускать ли hook, — он срабатывает автоматически при наступлении события." (part4_subagents_hooks.md:428)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Docs confirm hooks "execute automatically at specific points"; the model has no control over whether a hook fires. This is the documented distinguishing property vs. rules and system-prompt instructions, which the model can disregard.
  - **Recommendation:** —

### § 5.2 Advanced scenarios (7 scenarios)

- **Claim block:** "Семь сценариев hooks" (part4_subagents_hooks.md:436-450)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks ; https://opencode.ai/docs/plugins
  - **Evidence:** Per-scenario event mapping:
    1. Автоформатирование после Edit (prettier, black, gofmt) → `PostToolUse` (matcher: `Edit|Write`) — VERIFIED.
    2. Линтинг после записи файла (ESLint, ruff, golangci-lint), результат через stderr → `PostToolUse` — VERIFIED. Note: lecture says "результат возвращается через stderr"; correct pattern is exit 0 with JSON `{"output": "..."}` for warnings, or exit 2 with stderr to force a fix. The stderr-only framing is slightly imprecise.
    3. Автокоммит после завершения задачи (`git add -A && git commit`) → `Stop` — VERIFIED.
    4. Уведомление при ошибке (Slack webhook, системный звук) — the lecture does not name an event. Claude Code's `Notification` event fires for internal UI notifications (permission_prompt, idle_prompt, auth_success, elicitation_dialog), NOT on agent errors. For "notify on error" the appropriate events are `PostToolUseFailure` or `StopFailure`.
    5. Запуск тестов после изменения кода → `PostToolUse` (matcher `Edit|Write`) — VERIFIED.
    6. Инъекция контекста при старте сессии (git status, open issues, последние коммиты) → `SessionStart` — VERIFIED. Docs note: "Useful for loading development context like existing issues or recent changes to your codebase." Also implementable in OpenCode via `session.created` event.
    7. Блокировка опасных операций (`PreToolUse`, exit code 2) — VERIFIED. Docs: "Exit 2 … `PreToolUse` blocks the tool call" and stderr is fed back to Claude as an error message.
  - **Recommendation:** For scenario 4, clarify that the appropriate event is `PostToolUseFailure` or `StopFailure`, not `Notification`. For scenario 2, clarify that lint output is best returned as JSON stdout (exit 0) for non-blocking feedback or stderr with exit 2 for blocking feedback — not stderr alone.

### § 5.3 Claude Code — advanced configs

- **Claim:** "Конфигурация — JSON в `.claude/settings.json` (проектные) или `~/.claude/settings.json` (глобальные)." (part4_subagents_hooks.md:454)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/settings
  - **Evidence:** Docs confirm: Project scope = `.claude/settings.json` (shareable, committed to git); User scope = `~/.claude/` directory (all projects, not shareable). A third scope `.claude/settings.local.json` (local, gitignored) also exists but is not listed in the lecture — that omission is acceptable.
  - **Recommendation:** —

- **Claim:** JSON snippet at lines 458-477 — Claude Code hooks settings.json structure
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Field validation results — top-level `hooks`: ✅ | `PostToolUse` and `Stop` as event keys: ✅ | array of matcher groups: ✅ | `matcher` field: ✅ | nested `hooks` array with `type: "command"` and `command`: ✅ | empty matcher string `""` matching all: ✅
  - **Recommendation:** —

- **Claim:** "**20+ событий**" (part4_subagents_hooks.md:483)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Current docs list 26 distinct hook events (SessionStart, UserPromptSubmit, PreToolUse, PermissionRequest, PermissionDenied, PostToolUse, PostToolUseFailure, Notification, SubagentStart, SubagentStop, TaskCreated, TaskCompleted, Stop, StopFailure, TeammateIdle, InstructionsLoaded, ConfigChange, CwdChanged, FileChanged, WorktreeCreate, WorktreeRemove, PreCompact, PostCompact, Elicitation, ElicitationResult, SessionEnd). "20+" is accurate.
  - **Recommendation:** —

- **Claim:** "PreToolUse, PostToolUse, Stop, SessionStart, UserPromptSubmit, PermissionRequest, Notification, PostCompact и другие" (part4_subagents_hooks.md:483)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** All 8 named events confirmed present in the current event list: PreToolUse ✅, PostToolUse ✅, Stop ✅, SessionStart ✅, UserPromptSubmit ✅, PermissionRequest ✅, Notification ✅, PostCompact ✅.
  - **Recommendation:** —

- **Claim:** "**4 типа обработчиков** — `command` (shell), `prompt` (однопроходная оценка моделью), `agent` (запуск субагента с инструментами), `http` (POST-запрос на URL)." (part4_subagents_hooks.md:484)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Docs confirm exactly 4 handler types: `command` (shell command), `http` (HTTP POST), `prompt` (single-turn model evaluation), `agent` (spawns subagent with tools). Descriptions in the lecture match the docs precisely.
  - **Recommendation:** —

- **Claim:** "**Matcher** — регулярное выражение для фильтрации по имени инструмента. `\"Edit|Write\"` — только для этих двух. Пустой matcher — для всех." (part4_subagents_hooks.md:485)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Docs confirm matcher is a regex string filtering when hooks fire; for tool events (PreToolUse, PostToolUse, etc.) it matches against `tool_name`. Example `"Edit|Write"` matches only those tools. Empty string `""` (or `"*"` or omitting matcher) matches all occurrences.
  - **Recommendation:** —

- **Claim:** "**Переменные окружения** — `$FILEPATH`, `$TOOL_NAME`, `$TOOL_INPUT` и другие. Доступны внутри shell-команды." (part4_subagents_hooks.md:486)
  - **Status:** INACCURATE
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** `$FILEPATH`, `$TOOL_NAME`, and `$TOOL_INPUT` are NOT shell environment variables. Tool-specific data (tool name, input, file path) is delivered as JSON on stdin and must be read with e.g. `jq -r '.tool_name'`. The actual shell env vars are `$CLAUDE_PROJECT_DIR`, `$CLAUDE_PLUGIN_ROOT`, `$CLAUDE_PLUGIN_DATA`, `$CLAUDE_ENV_FILE`, `$CLAUDE_CODE_REMOTE`.
  - **Recommendation:** "**Входной контекст** — данные хука передаются как JSON в stdin: `tool_name`, `tool_input`, `tool_response` и другие. Shell-переменные хоста: `$CLAUDE_PROJECT_DIR` и др."

- **Claim:** Exit codes: `0` — OK, продолжить; `1` — ошибка (неблокирующая, подробности в verbose-режиме); `2` — заблокировать действие, stderr передаётся модели (part4_subagents_hooks.md:488-490)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Exit 0 = success (stdout parsed for JSON output) ✅. Exit 2 = blocking error (stderr fed to Claude as error message) ✅. Exit 1 = non-blocking error: transcript shows a one-line `<hook name> hook error` notice and execution continues, full stderr goes to debug log — not "verbose mode" per se, but functionally non-blocking ✅. The description "подробности в verbose-режиме" is an approximation; actual mechanism is debug log + one-line transcript notice. The baseline (2026-03-31) concern about exit 1 incorrectly feeding stderr to the model is NOT a problem in the current lecture — the lecture correctly states exit 1 is non-blocking.
  - **Recommendation:** Optionally clarify "подробности в debug-логе и одной строкой в transcript" вместо "verbose-режиме".

- **Claim:** "Тип `agent` — уникальная возможность Claude Code. Hook может запустить полноценного субагента с доступом к инструментам для глубокой проверки." (part4_subagents_hooks.md:492)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** The `agent` handler type spawns a subagent with tool access (Read, Grep, Glob, Bash) for multi-step verification. This is unique to Claude Code — OpenCode uses JS/TS plugin functions and Roo Code has no native hook system. The characterization of `agent` hooks as uniquely powerful is accurate.
  - **Recommendation:** —

### § 5.4 OpenCode — plugins

- **Claim:** "hooks реализованы через систему плагинов — JS/TS-модули вместо JSON-конфигов" (part4_subagents_hooks.md:496)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins
  - **Evidence:** Documentation confirms plugins are JavaScript or TypeScript modules placed in plugin directories or loaded via npm — not JSON configuration files.
  - **Recommendation:** —

- **Claim:** TypeScript plugin snippet at lines 501-515 — AutoFormatPlugin signature `async ({ $ }) =>`
  - **Status:** INACCURATE
  - **Source:** https://github.com/sst/opencode/blob/HEAD/packages/plugin/src/index.ts
  - **Evidence:** The canonical `PluginInput` type from the official `@opencode-ai/plugin` package is `{ client, project, directory, worktree, serverUrl, $ }`. The full context is passed, not just `$`. Valid signatures are `async (ctx) =>` or `async ({ project, client, $, directory, worktree }) =>`. Additionally, `output.args?.filePath` does not exist on the `tool.execute.after` handler — the after-hook's output shape is `{ title, output, metadata }`. The `args` field lives on `input` (which carries `{ tool, sessionID, callID, args }`). The lecture's code uses `output.args?.filePath`, which would always be undefined.
  - **Recommendation:** Correct signature to `async ({ $, project, client, directory, worktree }) =>`. Correct the after-hook body to read `const filePath = input.args?.filePath || ""` instead of `output.args?.filePath`.

- **Claim:** Plugin uses `$` shell template literal (lines 507, 512)
  - **Status:** VERIFIED
  - **Source:** https://github.com/sst/opencode/blob/HEAD/packages/plugin/src/index.ts
  - **Evidence:** `$` in `PluginInput` is typed as `BunShell` — Bun's tagged template literal shell API. Usage `await $\`command\`` is correct.
  - **Recommendation:** —

- **Claim:** Event names `tool.execute.after` and `session.idle` in TS snippet (part4_subagents_hooks.md:503, 511)
  - **Status:** VERIFIED
  - **Source:** https://github.com/sst/opencode/blob/HEAD/packages/plugin/src/index.ts
  - **Evidence:** Both `"tool.execute.after"` and `"session.idle"` are defined as typed keys in the official `Hooks` interface. `session.idle` appears as a bus event passed through the generic `event` hook.
  - **Recommendation:** —

- **Claim:** "Основные события — `tool.execute.before`, `tool.execute.after`, `session.created`, `session.idle`, `session.error`, `file.edited`, `permission.asked`, `shell.env` и другие" (part4_subagents_hooks.md:522)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://opencode.ai/docs/plugins ; https://github.com/sst/opencode/blob/HEAD/packages/plugin/src/index.ts
  - **Evidence:** All 8 named events exist. `tool.execute.before`, `tool.execute.after`, `shell.env`, and `permission.ask` (note: the API uses `permission.ask`, not `permission.asked`) are direct Hooks keys. `session.created`, `session.idle`, `session.error`, and `file.edited` are bus events received via the generic `event` hook. The "и другие" qualifier is accurate: the actual total is 25 bus event types plus 18 named hook slots in the Hooks interface, far exceeding the 8 listed.
  - **Recommendation:** Change `permission.asked` to `permission.ask` to match the actual Hooks type key. Optionally update the framing to note that the real count is 40+ distinct event/hook slots.

- **Claim:** "Полный доступ к SDK — `$` (shell), `client` (API), `project` (метаданные проекта)" (part4_subagents_hooks.md:523)
  - **Status:** VERIFIED
  - **Source:** https://github.com/sst/opencode/blob/HEAD/packages/plugin/src/index.ts
  - **Evidence:** The `PluginInput` type explicitly includes `$: BunShell` (shell), `client: ReturnType<typeof createOpencodeClient>` (SDK API client), and `project: Project` (project metadata). Two additional fields also exist: `directory: string` and `worktree: string`.
  - **Recommendation:** —

- **Claim:** "npm-пакеты — можно импортировать любые зависимости, вызывать HTTP API, писать в БД" (part4_subagents_hooks.md:524)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins
  - **Evidence:** Documentation confirms that local plugins can declare npm dependencies via `.opencode/package.json`, which are installed automatically via Bun at startup. npm-sourced plugins (declared in `opencode.json`) are also installed automatically.
  - **Recommendation:** —

- **Claim:** "Подключение — локально (`.opencode/plugins/`) или через npm (`\"plugin\": [\"package-name\"]` в `opencode.json`)" (part4_subagents_hooks.md:525)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins ; https://github.com/sst/opencode/blob/HEAD/packages/opencode/src/config/config.ts
  - **Evidence:** The npm loading mechanism via `"plugin": ["package-name"]` in `opencode.json` is confirmed. Source code scans `{plugin,plugins}/*.{ts,js}` within each `.opencode/` config directory — both `.opencode/plugin/` and `.opencode/plugins/` (plural) work. The lecture's `.opencode/plugins/` is correct.
  - **Recommendation:** —

### § 5.5 Kilo Code — emulation (formerly "Roo Code / Kilo Code")

- **Claim:** "Roo Code и Kilo Code **не имеют нативных hooks**. Единственный путь — эмуляция через комбинацию кастомного MCP-инструмента и строгих правил в инструкциях режима." (part4_subagents_hooks.md:531)
  - **Status:** VERIFIED (for Kilo Code only, Roo Code is being dropped)
  - **Source:** https://kilo.ai/docs ; https://kilo.ai/docs/automate
  - **Evidence:** Kilo Code documentation does not contain a hooks page or any event-trigger mechanism. The Automate section covers code reviews, MCP, shell integration, and agent manager — but nothing comparable to Claude Code's lifecycle hooks or OpenCode's plugin hook points. The "no native hooks → emulate via custom MCP tool + rules" framing remains correct.
  - **Recommendation:** Rewrite "Roo Code и Kilo Code" → "Kilo Code". Drop the "inherited from Roo" framing since Kilo has diverged.

- **Claim:** "Шаг 1. MCP-инструмент `lint_check`" — TypeScript snippet using `node:child_process` exec to run `npm run lint` (part4_subagents_hooks.md:535-555)
  - **Status:** VERIFIED
  - **Source:** MCP specification — https://modelcontextprotocol.io/specification (tool handler shape is standardized across all MCP clients, including Kilo Code)
  - **Evidence:** MCP is a standardized protocol — tool handlers of this shape (accept input object, return text/structured result) work with any MCP client. Kilo Code supports MCP per its Automate / Extending Kilo docs. The snippet is a valid MCP tool pattern regardless of the specific client.
  - **Recommendation:** —

- **Claim:** "Шаг 2. Правило в `.roo/rules-code/lint-before-write.md`" (part4_subagents_hooks.md:557)
  - **Status:** OUTDATED
  - **Source:** https://kilo.ai/docs/customize/custom-modes
  - **Evidence:** Kilo Code has diverged from Roo's `.roo/` directory layout. The current canonical locations for agent-scoped rules are `.kilocodemodes` (for mode-level `customInstructions`) or `.kilo/agents/<name>.md` (for the newer Markdown agent format, where per-agent guidance is part of the Markdown body). A `.roo/rules-code/` path is a Roo Code artifact and will not be picked up by Kilo Code.
  - **Recommendation:** Replace `.roo/rules-code/lint-before-write.md` with one of: (a) a `customInstructions` entry inside `.kilocodemodes` on the `safe-code` agent, or (b) the body of `.kilo/agents/safe-code.md` if using the Markdown format. Recommended (b) since it mirrors Claude Code's `.claude/agents/*.md` shape.

- **Claim:** "Шаг 3. Конфиг режима в `.roomodes`" — JSON snippet with `customModes` array and `groups: ["read", ["edit", { "fileRegex": "..." }], "command", "mcp"]` (part4_subagents_hooks.md:570-582)
  - **Status:** OUTDATED
  - **Source:** https://kilo.ai/docs/customize/custom-modes
  - **Evidence:** Three separate outdated pieces in this snippet:
    1. File name `.roomodes` → in Kilo Code it is `.kilocodemodes` (or `.kilo/agents/<name>.md`).
    2. Format JSON → YAML is now preferred (JSON kept for back-compat, so the snippet still parses, but it is no longer the idiomatic form).
    3. `groups` values `["read", ["edit", { "fileRegex": "..." }], "command", "mcp"]` — the nested `["edit", { "fileRegex": ... }]` tuple form is still supported per current docs ("First element of tuple… Second element is the options object"). But `"command"` and `"mcp"` are **not** among the currently-documented group values. The documented set is `read`, `edit`, `bash`, `glob`, `grep`, `list`, `task`, `webfetch`, `websearch`, `codesearch`, `todowrite`, `todoread`, and similar Claude Code-style tool names. To get shell-command execution, use `bash` instead of `command`. There is no longer a separate `mcp` group — MCP tool access is governed elsewhere.
  - **Recommendation:** Rewrite the snippet as YAML targeting `.kilocodemodes`:
    ```yaml
    customModes:
      - slug: safe-code
        name: Safe Code
        roleDefinition: Developer with mandatory lint checks before edits
        groups:
          - read
          - - edit
            - fileRegex: \.(ts|tsx|js|jsx)$
          - bash
        customInstructions: Always call lint_check before any file modification.
    ```
    Or switch to the newer `.kilo/agents/safe-code.md` Markdown format entirely.

- **Claim:** "Это **не гарантия**. Модель *может* пропустить проверку — особенно при сложных задачах или длинном контексте. На практике, если правило повторяется в нескольких местах (rules + customInstructions + roleDefinition), модель следует ему стабильно. Но для критичных сценариев (безопасность, деплой) полагаться на эмуляцию недостаточно." (part4_subagents_hooks.md:584)
  - **Status:** VERIFIED
  - **Source:** General reasoning from the "no native hooks → model-driven enforcement → non-guaranteed" chain; no Kilo-specific docs contradict this.
  - **Evidence:** With no host-level enforcement mechanism, anything enforced via prompt/rule/customInstructions relies on the model's compliance, which is probabilistic. The lecture's caveat is correct.
  - **Recommendation:** —

### § 5.6 Hooks comparison table

The table at part4_subagents_hooks.md:588-594 has three columns; Roo Code column is being dropped, Kilo Code column is verified separately below.

#### Claude Code column

- **Cell:** "Механизм — Нативные hooks (settings.json)" (row 590)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks ; https://code.claude.com/docs/en/settings
  - **Evidence:** Claude Code hooks are declared in `settings.json` under a top-level `hooks` key. Per-event keys, matcher arrays, and handler definitions all live in JSON. Confirmed in § 5.3 verification.
  - **Recommendation:** —

- **Cell:** "Гарантия срабатывания — Да" (row 591)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Hooks "execute automatically at specific points in Claude Code's lifecycle" and "execute on the host machine … not in the model." The host guarantees execution; the model cannot skip them.
  - **Recommendation:** —

- **Cell:** "Язык обработчика — Shell / prompt / agent / http" (row 592)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Four handler types confirmed: `command` (shell), `prompt` (single-turn LLM), `agent` (subagent with tools), `http` (POST). Matches § 5.3 verification.
  - **Recommendation:** —

- **Cell:** "Количество событий — 20+" (row 593)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** Current docs list 26 distinct hook events. "20+" is accurate and conservative.
  - **Recommendation:** Could be tightened to "26" but "20+" is safe against future additions/removals.

- **Cell:** "Сложность настройки — Низкая (JSON конфиг)" (row 594)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks
  - **Evidence:** The minimum viable hook is a single JSON object `{"hooks": {"PostToolUse": [{"matcher": "Edit|Write", "hooks": [{"type": "command", "command": "npm run format"}]}]}}`. No code compilation, no runtime setup — declarative only. Subjective framing but defensible.
  - **Recommendation:** —

#### OpenCode column

- **Cell:** "Механизм — Плагины (JS/TS)" (row 590)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins
  - **Evidence:** Plugins are JS/TS modules loaded from `.opencode/plugin(s)/` directories or via npm. Confirmed in § 5.4 verification.
  - **Recommendation:** —

- **Cell:** "Гарантия срабатывания — Да" (row 591)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins ; https://github.com/sst/opencode/blob/HEAD/packages/plugin/src/index.ts
  - **Evidence:** Plugin hook points are invoked by the host runtime (Bun) as part of the event bus — they fire regardless of what the model wants. Same host-enforced guarantee as Claude Code hooks.
  - **Recommendation:** —

- **Cell:** "Язык обработчика — JS/TS" (row 592)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins
  - **Evidence:** Plugins are authored in JavaScript or TypeScript and executed by Bun. Confirmed in § 5.4 verification.
  - **Recommendation:** —

- **Cell:** "Количество событий — 20+" (row 593)
  - **Status:** VERIFIED
  - **Source:** https://github.com/sst/opencode/blob/HEAD/packages/plugin/src/index.ts
  - **Evidence:** From the § 5.4 verification: the actual count is "25 bus event types plus 18 named hook slots" (total 40+). "20+" is accurate, though conservative.
  - **Recommendation:** Could be bumped to "40+" if the lecture wants to emphasize OpenCode's breadth.

- **Cell:** "Сложность настройки — Средняя (код на TS)" (row 594)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins
  - **Evidence:** Writing a plugin requires a TS/JS file, knowing the `PluginInput` signature, and understanding the Bun shell API. Higher barrier than JSON but still accessible. Subjective framing but defensible.
  - **Recommendation:** —

#### Kilo Code column

- **Cell:** "Механизм — Эмуляция (rules + custom tools)" (row 590)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://kilo.ai/docs (no hooks page)
  - **Evidence:** Kilo Code has no documented native hooks mechanism. Emulation via MCP tool + rules/customInstructions is still the only path — confirmed in § 5.5 verification. However, the specific file paths (`.roo/rules-code/`) are outdated; the modern Kilo path is `.kilocodemodes` customInstructions or `.kilo/agents/<name>.md` body.
  - **Recommendation:** Keep the "Эмуляция (rules + custom tools)" label but note in § 5.5 prose that the rules live in Kilo-specific locations, not `.roo/`.

- **Cell:** "Гарантия срабатывания — Нет (зависит от модели)" (row 591)
  - **Status:** VERIFIED
  - **Source:** General reasoning — no host-level hook enforcement exists in Kilo Code, so anything driven by customInstructions/rules is model-compliance-dependent.
  - **Evidence:** Without a native hook system, the only way to enforce a pre/post-action check is to tell the model to do it. Model compliance is probabilistic.
  - **Recommendation:** —

- **Cell:** "Язык обработчика — Shell (через MCP tool)" (row 592)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://kilo.ai/docs/automate (MCP integration is documented)
  - **Evidence:** Kilo Code supports MCP, so a custom MCP tool can shell out via `child_process` or similar. The "Shell (через MCP tool)" framing is accurate but slightly narrow — MCP tools can be written in any language with an MCP SDK (TS, Python, Go), not just shell. The lecture's framing is a shortcut for "external program via MCP", which is fair for a comparison table.
  - **Recommendation:** —

- **Cell:** "Количество событий — Нет нативных" (row 593)
  - **Status:** VERIFIED
  - **Source:** https://kilo.ai/docs (no hooks page)
  - **Evidence:** No native event triggers are documented for Kilo Code. The cell text is accurate.
  - **Recommendation:** —

- **Cell:** "Сложность настройки — Высокая (workaround)" (row 594)
  - **Status:** VERIFIED
  - **Source:** § 5.5 verification
  - **Evidence:** Three-step setup (MCP tool + rules/customInstructions + custom agent config) is objectively more work than dropping a JSON block into `settings.json`. Subjective but defensible.
  - **Recommendation:** —

### § 5.7 Selection strategy

- **Claim:** "Выбор механизма hooks зависит от двух факторов: нужна ли **гарантия** срабатывания и насколько сложна **логика** обработчика." (part4_subagents_hooks.md:598)
  - **Status:** VERIFIED
  - **Source:** Composite — § 5.3, § 5.4, § 5.5 verifications
  - **Evidence:** Both axes map cleanly onto the three tools: Claude Code and OpenCode have host-enforced guarantees, Kilo Code does not. Claude Code hooks max out at single shell/prompt/http/agent invocations; OpenCode plugins can do arbitrary JS/TS logic; Kilo emulation depends on model compliance. Framing is accurate.
  - **Recommendation:** —

- **Claim:** "**Нужна гарантия → Claude Code hooks или OpenCode plugins.** Хост выполняет hook безусловно — модель не может его пропустить. Для блокировки опасных операций, обязательного форматирования, автокоммитов — только этот путь." (part4_subagents_hooks.md:600)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/hooks ; https://opencode.ai/docs/plugins
  - **Evidence:** Both Claude Code hooks and OpenCode plugins are host-executed, meaning the model cannot skip them. `PreToolUse` (Claude Code) and `tool.execute.before` (OpenCode) can both block tool invocations via exit codes / thrown errors. The recommendation is correct.
  - **Recommendation:** —

- **Claim:** "**Нужна гибкость (условная логика, API-вызовы, сложные проверки) → OpenCode plugins.** Полноценный TypeScript с доступом к npm, SDK и shell. Можно написать hook, который проверяет coverage, отправляет метрики в Grafana и блокирует коммит при падении ниже порога." (part4_subagents_hooks.md:602)
  - **Status:** VERIFIED
  - **Source:** https://opencode.ai/docs/plugins ; § 5.4 verification
  - **Evidence:** Plugins are full TS/JS modules with access to `$` (Bun shell), `client` (SDK), `project`, and any npm dependency. The Grafana example is illustrative but realistic — nothing prevents a plugin from calling fetch(), reading coverage reports, and throwing to block execution. The recommendation holds. Note: Claude Code's `agent` handler type also supports arbitrary logic via a subagent, but that's more indirect than writing plain TS.
  - **Recommendation:** —

- **Claim:** "**Roo Code / Kilo Code → эмуляция с оговорками.** Работает для «мягких» сценариев (напоминание о линтере, рекомендация запустить тесты). Для критичных — комбинируйте с CI/CD: пусть эмулированный hook ловит 90% случаев, а CI/CD pipeline гарантирует остальные 10%." (part4_subagents_hooks.md:604)
  - **Status:** VERIFIED (framing accurate for Kilo Code after Roo Code is dropped)
  - **Source:** § 5.5 verification
  - **Evidence:** The practical advice is sound: non-guaranteed hooks are fine for soft reminders, and critical enforcement should fall back to CI/CD. This holds regardless of whether the label is "Roo / Kilo" or just "Kilo".
  - **Recommendation:** Rewrite the label "Roo Code / Kilo Code" → "Kilo Code". The rest of the bullet can stay verbatim. Suggested: "**Kilo Code → эмуляция с оговорками.** Работает для «мягких» сценариев (напоминание о линтере, рекомендация запустить тесты). Для критичных — комбинируйте с CI/CD: пусть эмулированный hook ловит 90% случаев, а CI/CD pipeline гарантирует остальные 10%."

## Summary

### Counts (§ 3.4 + § 5 only)

- **Total claims checked:** 52
- **VERIFIED:** 31
- **PARTIALLY VERIFIED:** 9
- **INACCURATE:** 5
- **OUTDATED:** 5
- **OUT OF SCOPE (Roo Code — to be removed):** 2
- **DEFERRED:** 0 (all previously deferred Kilo Code claims were verified on 2026-04-09)

### Issues requiring attention

#### § 3 — Subagents

1. **[HIGH] § 3.2 — Kilo Code description is substantially OUTDATED.** The lecture frames Kilo as a cosmetic Roo Code fork with Modes, Orchestrator, Boomerang Tasks, `.roomodes`, and a Task Manager UI. Current Kilo docs (kilo.ai/docs, verified 2026-04-09) show:
   - Modes renamed to Agents.
   - Built-in set: `code`, `plan`, `ask`, `debug`, `orchestrator` (deprecated). "Architect" is renamed to "plan".
   - Orchestrator deprecated — "Agents with full tool access (Code, Plan, Debug) now support subagents natively."
   - No Boomerang Tasks feature page (`kilo.ai/docs/features/boomerang-tasks` → 404).
   - No Task Manager UI documented. Only residual `/newtask` slash command in legacy VSCode extension.
   - Config file: `.kilocodemodes` (YAML preferred, JSON back-compat) or `.kilo/agents/<name>.md`, NOT `.roomodes`.
   - `groups` field uses Claude Code-style tool names (`bash`, `grep`, `glob`, `webfetch`, `task`, ...), not `command`/`mcp`.
   - **Recommendation:** Full rewrite of § 3.2. Drop Roo Code framing entirely. Reframe Kilo as: "CLI/VSCode agent with built-in agents (code, plan, ask, debug) supporting native subagents. Custom agents via `.kilo/agents/*.md` or YAML `.kilocodemodes`."

2. **[HIGH] § 3.4 row "Конфигурация" — Claude Code cell is INACCURATE.** "`AGENTS.md` + `subagent_type`" conflates the project memory file (`AGENTS.md`) with the subagent registry (`.claude/agents/*.md`).
   - **Recommendation:** "`.claude/agents/<name>.md` (Markdown с YAML frontmatter) + параметр `subagent_type` Agent-тула"

3. **[HIGH] § 3.4 row "Кастомные типы" — Claude Code cell is INACCURATE.** Same root cause: lists `AGENTS.md` instead of `.claude/agents/`.
   - **Recommendation:** "`.claude/agents/<name>.md`"

4. **[HIGH] § 3.4 row "Фоновый запуск" — Claude Code cell is INACCURATE.** Lecture says `run_in_background: true`; current frontmatter field is `background: true`.
   - **Recommendation:** "`background: true`"

5. **[HIGH] § 3.4 all Kilo Code cells — OUTDATED.** Column values like `.roomodes`, "Orchestrator / Boomerang", "Режимы" no longer match Kilo's current terminology and config.
   - **Recommendation:** Rewrite Kilo Code column using current Kilo terminology (see row-by-row recommendations in § 3.4).

6. **[MEDIUM] § 3.4 row "Модель субагентности" — Claude Code cell is PARTIALLY VERIFIED.** "Явный spawn через Agent tool" is narrow: Agent tool exists, but the primary invocation model is automatic delegation, with explicit invocation as opt-in.
   - **Recommendation:** "Автоматическая делегация через Agent tool (явный вызов через @-mention)"

7. **[MEDIUM] § 3.4 row "Параллельность" — OpenCode cell is PARTIALLY VERIFIED.** "Зависит от реализации" is misleading — OpenCode has no native parallelism; only external orchestration can simulate it.
   - **Recommendation:** "Нет (агенты — профили конфигурации, параллельный запуск требует внешней оркестровки)"

#### § 5 — Hooks

8. **[HIGH] § 5.2 scenario 4 "Уведомление при ошибке" — event mapping is wrong.** The lecture text says "при ошибке (Slack, звук)" which would map to the `Notification` event, but `Notification` actually fires for internal UI events (permission prompts, idle, auth). For error-driven notifications the correct events are `PostToolUseFailure` or `StopFailure`.
   - **Recommendation:** Clarify that for error notifications, the events are `PostToolUseFailure` or `StopFailure`, not `Notification`.

9. **[HIGH] § 5.3 "Переменные окружения" — INACCURATE.** Lecture claims `$FILEPATH`, `$TOOL_NAME`, `$TOOL_INPUT` as shell env vars. These are NOT env vars — tool-specific data arrives as JSON on stdin (`tool_name`, `tool_input`, `tool_response`) and must be parsed with e.g. `jq`. Actual shell env vars are `$CLAUDE_PROJECT_DIR`, `$CLAUDE_PLUGIN_ROOT`, etc.
   - **Recommendation:** "**Входной контекст** — данные хука передаются как JSON в stdin: `tool_name`, `tool_input`, `tool_response`. Shell-переменные хоста: `$CLAUDE_PROJECT_DIR` и др."

10. **[HIGH] § 5.4 TypeScript plugin snippet — INACCURATE.** Two bugs:
    - Destructures only `{ $ }` from `PluginInput`; the full context is `{ client, project, directory, worktree, serverUrl, $ }`.
    - Uses `output.args?.filePath` in the after-hook, which is always undefined. `args` lives on `input`, not `output`. The after-hook output shape is `{ title, output, metadata }`.
    - **Recommendation:** Change signature to `async ({ $, project, client, directory, worktree }) =>`. Change `output.args?.filePath` to `input.args?.filePath || ""`.

11. **[MEDIUM] § 5.4 event name `permission.asked` — INACCURATE.** Actual API key is `permission.ask`, not `permission.asked`.
    - **Recommendation:** Change `permission.asked` to `permission.ask`.

12. **[MEDIUM] § 5.5 file path `.roo/rules-code/lint-before-write.md` — OUTDATED.** Kilo Code does not read `.roo/` paths. Modern locations: `customInstructions` inside `.kilocodemodes`, or body of `.kilo/agents/<name>.md`.
    - **Recommendation:** Rewrite step 2 to put the rule text into `customInstructions` or into a Markdown agent body.

13. **[MEDIUM] § 5.5 `.roomodes` snippet — OUTDATED.** File name, format, and `groups` values are all wrong for current Kilo Code.
    - **Recommendation:** Rewrite as YAML `.kilocodemodes` with `groups: [read, [edit, {fileRegex: ...}], bash]`. Or switch to `.kilo/agents/safe-code.md` Markdown format.

14. **[LOW] § 5.2 scenario 2 "Линтинг через stderr" — PARTIALLY VERIFIED.** Lint output is best returned as JSON stdout (exit 0) for non-blocking feedback or stderr with exit 2 for blocking. Stderr-alone is imprecise.
    - **Recommendation:** Optional clarification of the exit-code-2 vs exit-0-JSON patterns.

15. **[LOW] § 5.3 exit code 1 description — PARTIALLY VERIFIED.** "Подробности в verbose-режиме" is an approximation; the actual mechanism is debug log + one-line transcript notice.
    - **Recommendation:** Optional clarification "подробности в debug-логе и одной строкой в transcript".

## Delta vs 2026-03-31 baseline

Only baseline issues that fall in § 3.4 or § 5 scope are tracked here. Issues in other sections remain unchanged.

### Baseline issues → current status

1. **[HIGH baseline] OpenCode agent config field names wrong (`systemPrompt` → `prompt`, `tools` → `permission`)** — § 3.3, out of this v2 scope (§ 3.3 is not § 3.4). Still open, to be fixed in a separate pass.

2. **[HIGH baseline] Exit code 1 feeds stderr to Claude — INCORRECT in baseline.** CURRENT STATUS: **RESOLVED in lecture text.** The current § 5.3 content says exit 1 is non-blocking (see part4_subagents_hooks.md:488-490). This v2 fact-check confirms the lecture's current wording is correct; the framing "подробности в verbose-режиме" is slightly imprecise (actual mechanism is debug log + transcript one-liner) but substantively correct. Was INACCURATE in the 2026-03-31 conspect, now PARTIALLY VERIFIED.

3. **[MEDIUM baseline] Claude Code hooks: 3 handler types, missing `http`.** CURRENT STATUS: **RESOLVED in lecture text.** § 5.3 now lists all 4 types: command, prompt, agent, http (part4_subagents_hooks.md:484). VERIFIED.

4. **[MEDIUM baseline] OpenCode: 8 events total (actual 28+).** CURRENT STATUS: **PARTIALLY RESOLVED.** § 5.4 now says "Основные события — ... и другие" (line 522), with the "и другие" qualifier making the 8-events framing a selection rather than a total. Still, the actual number is 25 bus events + 18 hook slots = 40+, and `permission.asked` should be `permission.ask`. See issue #11 in Summary. PARTIALLY VERIFIED.

5. **[LOW baseline] Claude Code: 12 events → now 21.** CURRENT STATUS: **RESOLVED in lecture text.** § 5.3 now says "20+ событий" (part4_subagents_hooks.md:483), and current docs list 26 events, so "20+" is safe. VERIFIED.

6. **[LOW baseline] AGENTS.md framing — custom subagents live in `.claude/agents/`, not AGENTS.md.** CURRENT STATUS: **STILL OPEN and escalated to HIGH.** § 3.4 rows "Конфигурация" and "Кастомные типы" still say `AGENTS.md`, which is INACCURATE. See issues #2 and #3 in Summary. The baseline noted this as a "minor inaccuracy in framing" but for a comparison table the precision matters more — upgraded to HIGH priority.

### New issues surfaced in v2 (not in baseline)

1. **[HIGH v2] § 3.4 `run_in_background: true` → actual field is `background: true`.** Not flagged in baseline (baseline only checked the Agent tool parameter, not the § 3.4 table cell). See Summary issue #4.

2. **[HIGH v2] Kilo Code has substantially diverged from Roo Code.** The baseline verified Kilo as a direct fork with Task Manager UI and `.roomodes` compatibility. As of 2026-04-09, Kilo has diverged: `.kilocodemodes` file, YAML preferred, new Agent terminology, Architect renamed to plan, Orchestrator deprecated, no documented Task Manager UI. See Summary issues #1 and #5.

3. **[HIGH v2] § 5.3 `$FILEPATH` / `$TOOL_NAME` / `$TOOL_INPUT` are not shell env vars.** Tool data arrives as JSON on stdin. Baseline did not check hook environment variable accuracy. See Summary issue #9.

4. **[HIGH v2] § 5.4 TypeScript plugin snippet — wrong destructuring and wrong `output.args` reference.** Baseline marked the TS snippet as "VALID syntax" but did not cross-check the actual `PluginInput` / `AfterHookOutput` types. See Summary issue #10.

5. **[MEDIUM v2] § 5.2 scenario 4 event mapping — `Notification` does not fire on errors.** Baseline did not enumerate the 7 scenarios against specific events. See Summary issue #8.

6. **[MEDIUM v2] § 5.4 `permission.asked` → `permission.ask`.** Baseline listed this event as "verified" but did not check the exact key spelling. See Summary issue #11.

7. **[MEDIUM v2] § 5.5 `.roo/rules-code/` path and `.roomodes` snippet are outdated for Kilo Code.** Baseline verified these against Roo Code docs; with Roo Code being dropped and Kilo diverged, the paths no longer apply. See Summary issues #12 and #13.

### Net delta

- **3 baseline issues RESOLVED** (exit code 1, 4th handler type, 12→21 events)
- **1 baseline issue PARTIALLY RESOLVED** (OpenCode events count — "и другие" added, but exact spelling and total still off)
- **1 baseline issue STILL OPEN and escalated** (AGENTS.md framing in § 3.4)
- **7 new HIGH/MEDIUM issues surfaced** (run_in_background, Kilo divergence, env vars, plugin destructuring, Notification event mapping, permission.ask, outdated Roo paths)
- **2 baseline issues OUT OF v2 SCOPE** (OpenCode agent field names in § 3.3 — still need to be addressed separately; slides TOC card 3 — already fixed)
