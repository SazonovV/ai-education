# Lecture 4 Fact-Check v2 — § 3.4 & § 5

**Date:** 2026-04-08
**Scope:** § 3.4 (tools comparison table) + § 5 (hooks)
**Baseline:** 2026-03-31-lecture4-factcheck.md
**Status:** IN PROGRESS

## Scope change — 2026-04-08

User decision during execution:

1. **Roo Code is being removed from the lecture entirely.** All Roo Code claims are marked `OUT OF SCOPE — to be removed from lecture` instead of being verified. The content edit phase will drop § 3.2's "Roo Code" framing, the "Roo Code / Kilo Code" column header, and any other Roo Code references.

2. **Kilo Code stays in the lecture as a standalone tool.** The current "Roo Code / Kilo Code" combined column gets relabeled to "Kilo Code". The technical claims (Boomerang, `.roomodes`, Modes, Orchestrator) carry over since Kilo Code inherits them from its Roo Code ancestry — but they need to be re-verified against `kilo.ai/docs` (not `docs.roocode.com`).

3. **Kilo Code verification is deferred.** During this fact-check, fetches against both `docs.roocode.com` and `kilo.ai/docs` consistently hung. Each Kilo Code claim is recorded in a separate file: `2026-04-08-lecture4-factcheck-v2-kilo-deferred.md`. That file is the source of truth for what still needs to be verified before the content edit phase can proceed on the Kilo Code column.

This report (the parent) only contains verified claims for **Claude Code** and **OpenCode**.

## § 3.4 — Tools comparison table

### Criterion: Модель субагентности

- **Claim:** "Явный spawn через Agent tool" (part4_subagents_hooks.md:299)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The underlying tool is confirmed as "Agent tool" (renamed from "Task tool" in v2.1.63 — old `Task(...)` references still work as aliases). However, the primary invocation model is *automatic delegation* by Claude based on subagent descriptions, not explicit spawn; explicit invocation is an opt-in via @-mention or `--agent` flag, not the default mode.
  - **Recommendation:** "Автоматическая делегация через Agent tool (явный вызов через @-mention)"

- **Claim:** "Режимы + Orchestrator / Boomerang" for Roo Code / Kilo Code (part4_subagents_hooks.md:299)
  - **Status:** OUT OF SCOPE
  - **Notes:** Roo Code is being removed from the lecture. Kilo Code claim is deferred to `2026-04-08-lecture4-factcheck-v2-kilo-deferred.md`.

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

- **Claim:** "`.roomodes` (JSON)" for Roo Code / Kilo Code (part4_subagents_hooks.md:300)
  - **Status:** OUT OF SCOPE
  - **Notes:** Roo Code dropped; Kilo Code deferred.

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

- **Claim:** "Worktree (git-копия)" for Claude Code (part4_subagents_hooks.md:302)
  - **Status:** VERIFIED
  - **Source:** https://code.claude.com/docs/en/sub-agents
  - **Evidence:** The `isolation` frontmatter field is documented: "Set to `worktree` to run the subagent in a temporary git worktree, giving it an isolated copy of the repository. The worktree is automatically cleaned up if the subagent makes no changes." This is a real, supported feature, not aspirational.
  - **Recommendation:** —

- **Claim:** "Отдельный контекст (общая FS)" for Roo Code / Kilo Code (part4_subagents_hooks.md:302)
  - **Status:** OUT OF SCOPE
  - **Notes:** Roo Code dropped; Kilo Code deferred.

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

- **Claim:** "`.roomodes`" for Roo Code / Kilo Code (part4_subagents_hooks.md:303)
  - **Status:** OUT OF SCOPE
  - **Notes:** Roo Code dropped; Kilo Code deferred.

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

- **Claim:** "Самая продвинутая модель субагентности — у Claude Code: полная изоляция через worktree, параллелизм и фоновый режим. Roo Code / Kilo Code берут удобством Orchestrator-паттерна и визуализацией в Kilo. OpenCode — минимальный, но достаточный подход для проектов, где нужны предустановленные профили без динамического порождения агентов." (part4_subagents_hooks.md:306)
  - **Status:** PARTIALLY VERIFIED
  - **Source:** Composite — https://code.claude.com/docs/en/sub-agents (CC), https://opencode.ai/docs/agents (OpenCode)
  - **Evidence:** The technical claims hold up against the per-row verification above — CC has worktree isolation (`isolation: worktree`), parallelism (multiple Agent calls / agent teams), background mode (`background: true`); OpenCode uses pre-defined config-driven profiles. The framing words ("самая продвинутая", "минимальный, но достаточный") are subjective but defensible as opinion.
  - **Recommendation:** Rewrite to drop the Roo / Kilo sentence (Roo Code is being removed; Kilo Code framing belongs in the rewritten § 3.2). Suggested replacement: "Claude Code предлагает наиболее зрелую модель субагентности: изоляцию через worktree, параллельный запуск и фоновый режим. OpenCode идёт по пути предустановленных профилей — минимально, но достаточно для проектов, где субагенты не порождаются динамически. Kilo Code занимает середину — Boomerang-оркестратор и визуализация задач, но без worktree-изоляции и параллелизма."

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
