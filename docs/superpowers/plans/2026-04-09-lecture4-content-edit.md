# Lecture 4 Content Edit Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply the 15 fact-check fixes from the 2026-04-08 v2 report and add three new content blocks (model choice, tools/MCP scoping, oh-my-opencode) to Lecture 4 — first in `content/part4_subagents_hooks.md`, then mirror to `presentations/part4/index.html`.

**Architecture:** Content file is the source of truth, edited first in logical section order (§ 1 → § 7) with commits per section. After user review approves the markdown, slides are updated in a separate mirror pass. All edits use unique text anchors (not line numbers) so order-of-operations on the same file is safe.

**Tech Stack:** Markdown (content), HTML + Reveal.js (slides), git for commits.

**Source documents:**
- Spec: `docs/superpowers/specs/2026-04-09-lecture4-content-edit-design.md`
- Fact-check report: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`
- Memory: `.claude/projects/-Users-sazonov-Desktop-AIpresentation/memory/project_lecture4_drop_roocode.md`

---

## Phase 0: Research and verification

### Task 1: Verify oh-my-opencode repository

**Files:**
- Read: (none)
- Output: Notes captured in the task comment for Task 15

- [ ] **Step 1: Fetch the GitHub repo README**

Run WebFetch on `https://github.com/code-yeongyu/oh-my-openagent` with prompt:
"Return the project tagline, installation command, list of default agents, and any example MCP/hook configurations. Ignore star counts."

Expected: README content returned.

- [ ] **Step 2: If the URL 404s, try fallbacks**

Try in order:
1. `https://github.com/opensoft/oh-my-opencode`
2. `https://www.npmjs.com/package/oh-my-opencode`
3. `https://opencode.ai/docs/plugins`

Goal: confirm an authoritative source with a real tagline and install command.

- [ ] **Step 3: Record findings**

Capture:
- Verified GitHub URL
- Install command (e.g., `bunx oh-my-opencode install`)
- 3-5 agent names that ship by default
- 2-3 plugin / hook examples

If nothing can be verified, the § 5.4.1 subsection degrades to a short paragraph pointing to `https://opencode.ai/docs/plugins` with a general description.

### Task 2: Verify current model IDs and context sizes

**Files:**
- Output: Notes for Task 10 (§ 3.6 model table)

- [ ] **Step 1: Capture Claude model IDs from the session context**

From the system-level environment you know:
- `claude-opus-4-6`
- `claude-sonnet-4-6`
- `claude-haiku-4-5-20251001`

All three have 200K token context.

- [ ] **Step 2: WebFetch GLM-4.6 facts**

WebFetch `https://docs.z.ai/guides/llm/glm-4.6` with prompt:
"Return the context window size, model ID string used via API, and two-sentence description of strengths for coding tasks."

- [ ] **Step 3: WebFetch Qwen-Coder-Next facts**

WebFetch `https://qwen.ai/docs` or search results for "Qwen3 Coder" with prompt:
"Return the context window size (256K+ expected), the exact model name string, and two-sentence description of strengths for coding tasks."

- [ ] **Step 4: Record findings**

Build a small lookup table for Task 10:
```
Opus 4.6   | 200K   | architectural reasoning, complex refactoring
Sonnet 4.6 | 200K   | balance of speed and precision
Haiku 4.5  | 200K   | fast, cheap, good for search/exploration
GLM-4.6    | 200K+  | open-source coding, good cost-performance
Qwen-Coder | 256K+  | long context, code-specialized, open-source
```

Any cell you cannot verify → use the label `(проверить)` — do NOT invent numbers.

### Task 3: Verify Claude Code `.claude/agents/<name>.md` frontmatter

**Files:**
- Output: Notes for Task 9 (§ 3.5 CC example)

- [ ] **Step 1: WebFetch CC subagent docs**

WebFetch `https://docs.claude.com/en/docs/claude-code/sub-agents` with prompt:
"Return the YAML frontmatter fields supported in `.claude/agents/<name>.md`, specifically: name, description, tools, model, isolation, background, permissionMode, hooks. For each field, return one-line usage. Also return the exact syntax for restricting MCP tools (e.g., `mcp__<server>__<tool>`)."

- [ ] **Step 2: Verify MCP tool syntax**

Confirm:
- Individual MCP tool: `mcp__<server>__<tool>`
- All tools from one server: `mcp__<server>__*`

If the docs use different syntax, record it verbatim.

### Task 4: Verify Kilo Code `.kilocodemodes` current format

**Files:**
- Output: Notes for Tasks 6, 9, 13 (Kilo rewrites)

- [ ] **Step 1: WebFetch Kilo custom modes docs**

WebFetch `https://kilo.ai/docs/features/custom-modes` with prompt:
"Return the exact YAML schema of `.kilocodemodes`: field names at the root, fields inside each custom mode entry (slug, name, roleDefinition, groups, customInstructions, model). Return the list of valid `groups` values. Show a minimal working example."

- [ ] **Step 2: Verify Markdown agent format**

WebFetch `https://kilo.ai/docs/features/agents` or `https://kilo.ai/docs` with prompt:
"Return the structure of `.kilo/agents/<name>.md` Markdown agents — what YAML frontmatter fields are required, how the body becomes the role definition."

- [ ] **Step 3: Record the canonical examples**

Confirm (or correct) these expected values from the v2 fact-check report:
- File: `.kilocodemodes` (YAML preferred, JSON back-compat)
- Built-in agents: `code`, `plan`, `ask`, `debug`
- `groups` values: `read`, `edit`, `bash`, `grep`, `glob`, `list`, `task`, `webfetch`, `websearch`, `codesearch`, `todowrite`, `todoread`

If any of these changed since 2026-04-09, use the fresh values.

### Task 5: Verify current OpenCode `opencode.json` agent format

**Files:**
- Output: Notes for Tasks 7, 9 (§ 3.3 fix and § 3.5 OpenCode example)

- [ ] **Step 1: WebFetch OpenCode agents doc**

WebFetch `https://opencode.ai/docs/agents` with prompt:
"Return the exact JSON schema for defining an agent in `opencode.json`: field names (prompt vs systemPrompt, permission field — is it an array of strings like `['read', 'edit']` or an object like `{read: 'allow', edit: 'deny'}`?). Show a minimal working example. Also return how MCP servers are attached per agent."

- [ ] **Step 2: Verify model ID format**

Confirm the `model` field uses `provider:model-id` syntax (e.g., `anthropic:claude-sonnet-4-6`). If the format differs, record it.

- [ ] **Step 3: Record canonical example**

Build one reference snippet that matches the current docs; it will be reused in Tasks 7 and 9.

---

## Phase 1: Content edits (`content/part4_subagents_hooks.md`)

All Phase 1 tasks work on `/Users/sazonov/Desktop/AIpresentation/content/part4_subagents_hooks.md`. Each task uses unique text anchors, so order doesn't matter for correctness, but the recommended execution order follows the lecture flow (§ 1 → § 7) for cleaner commits.

### Task 6: Fix § 1 intro — drop Sonnet-specific mention

**Files:**
- Modify: `content/part4_subagents_hooks.md` (around line 26)

- [ ] **Step 1: Apply the Edit**

```
old_string:
У Claude Sonnet — 200K токенов, но эффективность падает задолго до лимита. На практике:

new_string:
У современных frontier-моделей (Claude Opus/Sonnet 4.6, Haiku 4.5, GLM-4.6, Qwen-Coder-Next) — 200K+ токенов, но эффективность падает задолго до лимита. На практике:
```

- [ ] **Step 2: Verify**

Run Grep for `Claude Sonnet —` in the content file. Expected: 0 matches.

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: replace Sonnet-specific context mention with frontier models"
```

### Task 7: Fix § 3.1 Claude Code — AGENTS.md framing

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 3.1, around lines 165-192)

- [ ] **Step 1: Fix the subagent types table row**

```
old_string:
| Кастомный (из AGENTS.md) | Определяется в конфигурации | Специализированные задачи |

new_string:
| Кастомный (из `.claude/agents/<name>.md`) | Определяется в YAML frontmatter | Специализированные задачи |
```

- [ ] **Step 2: Rewrite the AGENTS.md prose block and example**

```
old_string:
**AGENTS.md** — файл в корне репозитория (или `.claude/agents/*.md` для отдельных агентов), описывающий кастомных агентов. Claude Code подхватывает его автоматически:

```markdown
# AGENTS.md

## code-reviewer
You are a senior code reviewer. Focus on:
- Security vulnerabilities (injection, auth bypass, data exposure)
- Performance anti-patterns (N+1 queries, missing indexes)
- API contract violations

Tools: Read, Grep, Glob
Constraints: Never modify files. Always cite file:line in feedback.
Output format: list of findings with severity (critical/warning/info).

## test-writer
You are a test engineer. Given a source file, write comprehensive tests.
Tools: Read, Grep, Glob, Edit, Bash
Constraints: Use existing test framework in the project. Run tests before returning.
Output format: summary of tests added and their pass/fail status.
```

new_string:
**Кастомные субагенты** живут в `.claude/agents/<name>.md` — отдельный файл на каждого. Не путать с `AGENTS.md` в корне репозитория: это проектная память, а не реестр субагентов. Формат — Markdown с YAML frontmatter:

```markdown
---
name: code-reviewer
description: Senior code reviewer — security, performance, API contracts
tools: [Read, Grep, Glob]
model: claude-sonnet-4-6
---

Ты — senior code reviewer. Фокусируйся на:
- security vulnerabilities (injection, auth bypass, data exposure)
- performance anti-patterns (N+1 queries, missing indexes)
- API contract violations

Никогда не редактируй файлы. Всегда указывай file:line в комментариях.
Формат вывода: список находок с severity (critical/warning/info).
```

Ключевые поля frontmatter: `name` (slug субагента), `description` (критерий автоматического выбора), `tools` (массив доступных инструментов — об этом подробно в § 3.5), `model` (какая модель запускает субагента — об этом в § 3.6).
```

- [ ] **Step 3: Verify**

Run Grep for `из AGENTS.md` in the content file. Expected: 0 matches.
Run Grep for `## code-reviewer$` in the content file. Expected: 0 matches (the header-style example is gone).

- [ ] **Step 4: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: fix § 3.1 AGENTS.md framing — custom agents live in .claude/agents/"
```

### Task 8: Rewrite § 3.2 as Kilo Code standalone (drop Roo Code)

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 3.2, lines 200-259)

**Context:** The current § 3.2 covers "Roo Code / Kilo Code" as one unit, with Modes, Orchestrator, Boomerang Tasks, `.roomodes`, and Task Manager UI. Per the fact-check report, Kilo Code has diverged from Roo Code (new terminology, `.kilocodemodes`, Orchestrator deprecated, no Task Manager UI docs). Roo Code is being removed from the lecture entirely.

- [ ] **Step 1: Replace the entire § 3.2 block**

```
old_string:
### 3.2 Roo Code / Kilo Code

Roo Code реализует субагентность через систему **режимов (Modes)**. Kilo Code — форк Roo Code с идентичной архитектурой режимов и дополнительным UI для управления задачами, поэтому рассматриваем их вместе.

**Встроенные режимы:**

- **Code** — полный доступ к редактированию и выполнению команд
- **Architect** — только чтение, фокус на проектировании и планировании
- **Ask** — ответы на вопросы, без доступа к инструментам изменения
- **Debug** — анализ ошибок, чтение логов, диагностика
- **Orchestrator** — специальный режим для делегации задач другим режимам

**Orchestrator + Boomerang Tasks.** Orchestrator — ключевой механизм субагентности в Roo Code. Он анализирует задачу, разбивает на подзадачи и делегирует их другим режимам через Boomerang Tasks. Каждый режим выполняет свою часть и возвращает результат обратно Orchestrator'у.

```
Пользователь: "Добавить авторизацию в API"
         │
    Orchestrator
    ├── → Architect: спроектировать схему auth
    │   ← архитектурный план
    ├── → Code: реализовать middleware
    │   ← код готов
    ├── → Code: написать тесты
    │   ← тесты написаны
    └── → Debug: проверить интеграцию
        ← всё работает

    Orchestrator: собирает итог, отвечает пользователю
```

Boomerang Tasks работают **последовательно** — Orchestrator ждёт завершения текущей подзадачи, прежде чем отправить следующую. Это проще в реализации, но медленнее параллельного запуска.

**Кастомные режимы через `.roomodes`.** Файл `.roomodes` в корне проекта позволяет определять свои режимы:

```json
{
  "customModes": [
    {
      "slug": "doc-writer",
      "name": "Documentation Writer",
      "roleDefinition": "You are a technical writer. Generate clear, structured documentation for the codebase. Use JSDoc/TSDoc style for code comments. Write README sections in markdown.",
      "groups": ["read", "command"],
      "customInstructions": "Always include usage examples. Follow the existing documentation style in the project."
    },
    {
      "slug": "security-audit",
      "name": "Security Auditor",
      "roleDefinition": "You are a security engineer. Analyze code for vulnerabilities, check dependencies for known CVEs, review auth and data handling.",
      "groups": ["read"],
      "customInstructions": "Classify findings by OWASP Top 10. Output a structured report."
    }
  ]
}
```

Поле `groups` определяет, какие группы инструментов доступны режиму: `read` (чтение файлов), `edit` (редактирование), `command` (выполнение команд), `browser` (браузер), `mcp` (MCP-серверы).

**Переключение режимов:** Orchestrator переключает режимы автоматически при делегации. Пользователь тоже может переключиться вручную — например, из Code в Debug, если что-то пошло не так.

**Особенность Kilo Code:** встроенный **Task Manager UI** — визуальная панель, показывающая дерево подзадач, их статус (в очереди / выполняется / завершено / ошибка) и результаты. Конфигурация `.roomodes` полностью совместима между Roo Code и Kilo Code.

new_string:
### 3.2 Kilo Code

Kilo Code — CLI/VSCode агент с собственной системой встроенных агентов и нативной поддержкой субагентов. Исторически это форк Roo Code, но на 2026 год архитектура существенно разошлась: терминология «modes» заменена на «agents», Orchestrator как отдельный режим объявлен deprecated, конфигурационный файл сменился с `.roomodes` на `.kilocodemodes`.

**Встроенные агенты:**

- **code** — полный доступ к редактированию и выполнению команд
- **plan** — проектирование и планирование (бывший Architect)
- **ask** — ответы на вопросы без инструментов изменения
- **debug** — анализ ошибок и диагностика

Каждый из этих агентов поддерживает **нативный вызов субагентов** — не нужен отдельный Orchestrator. Агент с полным доступом к tools сам делегирует подзадачи специализированным субагентам, когда это нужно.

```
Пользователь: "Добавить авторизацию в API"
         │
       code
    ├── → subagent (plan): спроектировать схему auth
    │   ← архитектурный план
    ├── → subagent (code): реализовать middleware
    │   ← код готов
    ├── → subagent (code): написать тесты
    │   ← тесты написаны
    └── → subagent (debug): проверить интеграцию
        ← всё работает

    code: собирает итог, отвечает пользователю
```

Делегация последовательная — Kilo Code ждёт завершения текущего субагента перед запуском следующего. Параллельный запуск не поддерживается нативно.

**Кастомные агенты — два формата:**

Файл `.kilocodemodes` в корне проекта (YAML предпочтителен, JSON совместим):

```yaml
customModes:
  - slug: doc-writer
    name: Documentation Writer
    roleDefinition: |
      You are a technical writer. Generate clear, structured documentation
      for the codebase. Use JSDoc/TSDoc style for code comments.
    groups:
      - read
      - edit
      - grep
      - glob
    customInstructions: Always include usage examples.

  - slug: security-audit
    name: Security Auditor
    roleDefinition: |
      You are a security engineer. Analyze code for vulnerabilities.
    groups:
      - read
      - grep
      - glob
    customInstructions: Classify findings by OWASP Top 10.
```

Альтернативный, более новый формат — отдельный Markdown-файл на агента в `.kilo/agents/<slug>.md`:

```markdown
---
name: doc-writer
groups: [read, edit, grep, glob]
---

You are a technical writer. Generate clear, structured documentation...
```

Поле `groups` содержит имена tool-групп: `read`, `edit`, `bash`, `grep`, `glob`, `list`, `task`, `webfetch`, `websearch`, `codesearch`, `todowrite`, `todoread`. MCP-серверы подключаются отдельно — об этом подробнее в § 3.5.

**Переключение агентов:** вручную из UI или автоматически при субделегации.
```

- [ ] **Step 2: Verify**

Run Grep for `Roo Code` in the content file within § 3.2. Expected: 0 matches in lines 200-260.
Run Grep for `Orchestrator` in the file. Expected: 0 matches (the term is deprecated).
Run Grep for `Boomerang` in the file. Expected: 0 matches.
Run Grep for `.roomodes` in the file (expect it to remain only in § 5.5 until Task 17 is done).

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: rewrite § 3.2 as Kilo Code standalone, drop Roo Code framing"
```

### Task 9: Fix § 3.3 OpenCode — verify snippet, update model ID

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 3.3, lines 261-293)

- [ ] **Step 1: Apply findings from Task 5**

If the current snippet uses `prompt` + `permission` (array of strings) and Task 5 confirmed the modern format is either an array-of-strings or an object-map, keep the current format if valid. Otherwise replace with the verified format.

- [ ] **Step 2: Update the model ID**

```
old_string:
      "model": "anthropic:claude-sonnet-4-20250514",
      "prompt": ".opencode/agents/reviewer.md",
      "permission": ["read", "glob", "grep"]

new_string:
      "model": "anthropic:claude-sonnet-4-6",
      "prompt": ".opencode/agents/reviewer.md",
      "permission": ["read", "glob", "grep"]
```

Repeat the model ID fix for the other two entries (`implementer`, `planner`) — all three currently use the outdated `claude-sonnet-4-20250514`.

- [ ] **Step 3: Verify**

Run Grep for `claude-sonnet-4-20250514` in the content file. Expected: 0 matches.

- [ ] **Step 4: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: update OpenCode snippet model IDs to claude-sonnet-4-6"
```

### Task 10: Fix § 3.4 comparison table — cell-level corrections

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 3.4, lines 295-306)

- [ ] **Step 1: Replace the whole table + post-paragraph**

```
old_string:
### 3.4 Сводная таблица

| Критерий | Claude Code | Roo Code / Kilo Code | OpenCode |
|---|---|---|---|
| Модель субагентности | Явный spawn через Agent tool | Режимы + Orchestrator / Boomerang | Агенты в конфигурации |
| Конфигурация | `AGENTS.md` + `subagent_type` | `.roomodes` (JSON) | `opencode.json` / `.opencode/agents/` |
| Параллельность | Да (несколько Agent-вызовов) | Последовательная (Boomerang) | Зависит от реализации |
| Изоляция | Worktree (git-копия) | Отдельный контекст (общая FS) | Отдельный контекст (общая FS) |
| Кастомные типы | `AGENTS.md` | `.roomodes` | Агенты в конфиге |
| Фоновый запуск | `run_in_background: true` | Нет | Нет |

Самая продвинутая модель субагентности — у Claude Code: полная изоляция через worktree, параллелизм и фоновый режим. Roo Code / Kilo Code берут удобством Orchestrator-паттерна и визуализацией в Kilo. OpenCode — минимальный, но достаточный подход для проектов, где нужны предустановленные профили без динамического порождения агентов.

new_string:
### 3.4 Сводная таблица

| Критерий | Claude Code | Kilo Code | OpenCode |
|---|---|---|---|
| Модель субагентности | Автоматическая делегация через Agent tool (явный вызов @-mention) | Агенты с нативными субагентами | Агенты-профили в конфигурации |
| Конфигурация | `.claude/agents/<name>.md` (Markdown + YAML frontmatter) + `subagent_type` | `.kilocodemodes` (YAML) / `.kilo/agents/<name>.md` | `opencode.json` / `.opencode/agents/` |
| Параллельность | Да (несколько Agent-вызовов) | Последовательная делегация | Нет (профили; параллелизм — внешняя оркестровка) |
| Изоляция | Worktree (git-копия) | Отдельный контекст (общая FS) | Отдельный контекст (общая FS) |
| Кастомные типы | `.claude/agents/<name>.md` | `.kilocodemodes` / `.kilo/agents/<name>.md` | Агенты в `opencode.json` |
| Фоновый запуск | `background: true` | Нет | Нет |

Самая продвинутая модель субагентности — у Claude Code: полная изоляция через worktree, параллелизм и фоновый режим. Kilo Code компактнее: нативная субагентность без отдельного оркестратора, но без параллелизма. OpenCode — минимальный подход для проектов, где нужны предустановленные профили без динамического порождения агентов.
```

- [ ] **Step 2: Verify**

Run Grep for `Roo Code / Kilo Code` in the content file. Expected: 0 matches in § 3.4 (other sections handled later).
Run Grep for `run_in_background: true` in the content file. Expected: 0 matches.
Run Grep for `AGENTS.md` (table cell). Expected: 0 matches in § 3.4.

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: fix § 3.4 comparison table (CC fields, Kilo column, OpenCode parallelism)"
```

### Task 11: Add NEW § 3.5 — Ограничение доступа: tools и MCP

**Files:**
- Modify: `content/part4_subagents_hooks.md` (insert after § 3.4, before `## 4`)

- [ ] **Step 1: Insert the new section**

Find the anchor `\n---\n\n## 4. Декомпозиция задач` and replace with the new § 3.5, § 3.6, then the `---` separator, then `## 4`.

```
old_string:

---

## 4. Декомпозиция задач и паттерно оркестрации

new_string:

### 3.5 Ограничение доступа: tools и MCP для субагентов

Когда субагент запускается со всеми инструментами родителя «на всякий случай», это проблема. Во-первых, это расширяет attack surface для prompt injection — любое вредоносное содержимое в читаемом файле может попытаться спровоцировать запись куда не надо или вызов внешнего API. Во-вторых, это делает поведение менее предсказуемым: модель с правом `bash` будет пытаться решать задачи через shell, даже когда это лишнее. В-третьих, избыточные инструменты тратят контекст — описания всех tools занимают место в системном промпте.

Принцип: **каждому субагенту — только те tools и MCP-серверы, которые нужны для его конкретной задачи.** Ревьюеру — только чтение. Тестеру — чтение плюс запуск тестов. Исследователю Sentry — чтение плюс MCP к Sentry.

**Claude Code** — поле `tools` в YAML frontmatter файла `.claude/agents/<name>.md`:

```markdown
---
name: researcher
description: Explores codebase, surveys similar projects, summarizes findings
tools: [Read, Grep, Glob, WebFetch, mcp__context7__search, mcp__sentry__get_issue]
model: claude-haiku-4-5-20251001
---

Ты — research agent. Твоя задача — исследовать, а не изменять...
```

Синтаксис `mcp__<server>__<tool>` включает отдельный инструмент из MCP-сервера. Для всего сервера сразу — `mcp__<server>__*`. Если поле `tools` отсутствует, субагент наследует все инструменты родителя — обычно нежелательное поведение.

**Kilo Code** — поле `groups` в `.kilocodemodes`:

```yaml
customModes:
  - slug: researcher
    name: Researcher
    roleDefinition: |
      You are a research agent focused on codebase exploration.
    groups:
      - read
      - grep
      - glob
      - webfetch
```

Значения `groups` — имена tool-групп в Claude Code-style: `read`, `edit`, `bash`, `grep`, `glob`, `list`, `task`, `webfetch`, `websearch`, `codesearch`, `todowrite`, `todoread`. MCP-серверы подключаются через отдельную секцию `mcp_servers` в том же файле или глобально через `.kilo/mcp.yaml`.

**OpenCode** — поле `permission` в `opencode.json` per-agent:

```json
{
  "agents": {
    "researcher": {
      "model": "anthropic:claude-haiku-4-5",
      "prompt": ".opencode/agents/researcher.md",
      "permission": ["read", "glob", "grep", "webfetch"]
    }
  }
}
```

Формат `permission` — массив строк с именами разрешённых tool-групп. MCP-серверы подключаются отдельно в корне `opencode.json` и становятся доступны всем агентам, которым они выданы через `permission`.

**Что ограничивать.** Минимальный чек-лист:

- **Чтение vs запись.** Ревьюеры, исследователи, документаторы — только чтение.
- **Bash.** Только тем, кому действительно нужно запускать команды (тестеры, деплойщики). Большинству субагентов bash не требуется.
- **Web (webfetch / websearch).** Только исследовательским агентам. Остальным — блокировать.
- **Конкретные MCP-серверы.** Давать только те серверы, которые релевантны задаче. Sentry-агент не нуждается в Context7, и наоборот.

Субагент с узким набором tools работает предсказуемее, быстрее и безопаснее. Модель не «фантазирует», как решить задачу через недоступные инструменты — она делает то, что может, из того, что есть.

### 3.6 Выбор модели для субагента

Запускать все субагенты на самой мощной доступной модели — дорого и избыточно. Субагент — хорошая точка экономии: у него изолированный контекст, узкая задача и понятный критерий успеха. Для многих ролей достаточно более лёгкой модели.

**Актуальные frontier-модели для кодинга (2026):**

| Модель | Контекст | Сильные стороны |
|---|---|---|
| Claude Opus 4.6 | 200K | Архитектурные решения, сложный рефакторинг, многошаговое рассуждение |
| Claude Sonnet 4.6 | 200K | Баланс скорости и точности для большинства задач |
| Claude Haiku 4.5 | 200K | Быстрый отклик, поиск по коду, лёгкие правки |
| GLM-4.6 | 200K+ | Open-source, хороший кодинг по стоимости |
| Qwen-Coder-Next | 256K+ | Длинный контекст, специализация на коде, open-source |

**Плюсы лёгких моделей для субагентов:**

- **Скорость.** Haiku отвечает в 3-5 раз быстрее Opus. Для субагента, который вызывается 10 раз за сессию, это экономит минуты.
- **Стоимость.** Haiku и лёгкие open-source модели дешевле Opus на порядок. При fan-out с 5 параллельными субагентами это существенно.
- **Параллельный бюджет.** При ограниченном rate limit можно запустить больше субагентов одновременно, если каждый использует более дешёвую модель.
- **Разгрузка контекста оркестратора.** Пока субагент работает, родитель не тратит контекст на ожидание.

**Минусы лёгких моделей:**

- **Слабее многошаговое рассуждение.** Задачи с 5+ логическими шагами или неочевидными зависимостями даются сложнее.
- **Чувствительность к промпту.** Нечёткая инструкция с большей вероятностью приводит к ошибке — лёгкие модели меньше «достраивают» намерение.
- **Хуже на редких языках / фреймворках.** Frontier-модели видели больше материала по нишевым технологиям.
- **Слабее на нетривиальной логике.** Сложные invariants, рекурсия с состоянием, многопоточные баги — домены более мощных моделей.

**Рекомендации по ролям:**

| Роль субагента | Рекомендуемая модель |
|---|---|
| Research / code search / exploration | Haiku 4.5, GLM-4.6, Qwen-Coder-Next |
| Документация / комментарии / рефакторинг имён | Haiku 4.5, Qwen-Coder-Next |
| Написание тестов по спецификации | Sonnet 4.6 |
| Реализация фичи по плану | Sonnet 4.6 |
| Архитектура / дизайн-решения | Opus 4.6 |
| Диагностика сложных багов | Opus 4.6, Sonnet 4.6 |
| Рефакторинг ядра системы | Opus 4.6 |

**Где это конфигурируется:**

- **Claude Code** — поле `model` в YAML frontmatter `.claude/agents/<name>.md`.
- **Kilo Code** — поле `model` в `.kilocodemodes` или в Markdown-агенте `.kilo/agents/<name>.md`.
- **OpenCode** — поле `model` в `opencode.json` per-agent, синтаксис `provider:model-id` (например, `anthropic:claude-haiku-4-5`).

Практическое правило: начинайте с Sonnet/Haiku как дефолта для большинства субагентов, поднимайте до Opus только там, где видите, что задача требует более мощного рассуждения.

---

## 4. Декомпозиция задач и паттерно оркестрации
```

**NOTE:** In the step above, the trailing `## 4. Декомпозиция задач и паттерно оркестрации` heading is repeated verbatim to ensure the Edit anchor is unique. The typo `паттерно` (missing `в`) is preserved from the source text — do not "fix" it in this edit. A separate cleanup pass can handle typos if needed.

- [ ] **Step 2: Verify**

Run Grep for `### 3.5 Ограничение доступа` in the content file. Expected: 1 match.
Run Grep for `### 3.6 Выбор модели` in the content file. Expected: 1 match.
Run Grep for `## 4. Декомпозиция` in the content file. Expected: 1 match (moved but still there).

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: add § 3.5 tools/MCP scoping and § 3.6 model choice"
```

### Task 12: Update § 4 patterns — Roo/Kilo references → Kilo

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 4, lines ~355-416)

- [ ] **Step 1: Replace support line for Fan-out pattern**

```
old_string:
**Поддержка:** Claude Code — нативные параллельные вызовы Agent tool. Roo Code / Kilo Code — только последовательный Boomerang, параллелизм недоступен.

new_string:
**Поддержка:** Claude Code — нативные параллельные вызовы Agent tool. Kilo Code — только последовательная делегация, параллелизм недоступен.
```

- [ ] **Step 2: Replace support line for Specialist pattern**

```
old_string:
**Поддержка:** все инструменты — паттерн реализуется через AGENTS.md (Claude Code), `.roomodes` (Roo/Kilo) или конфиг агентов (OpenCode).

new_string:
**Поддержка:** все инструменты — паттерн реализуется через `.claude/agents/` (Claude Code), `.kilocodemodes` (Kilo Code) или конфиг агентов (OpenCode).
```

- [ ] **Step 3: Replace support line for Supervisor pattern**

```
old_string:
**Поддержка:** Claude Code — родитель может вызвать Agent tool повторно с уточнённым промптом. Roo Code / Kilo Code — Orchestrator проверяет результат Boomerang Task и перенаправляет при необходимости.

new_string:
**Поддержка:** Claude Code — родитель может вызвать Agent tool повторно с уточнённым промптом. Kilo Code — родительский агент проверяет результат субагента и перенаправляет при необходимости.
```

- [ ] **Step 4: Fix the pattern summary table row**

```
old_string:
| Fan-out / fan-in | Параллельные субагенты, родитель собирает | Обновление 5 микросервисов | Claude Code (параллельно), Roo/Kilo (последовательно) |

new_string:
| Fan-out / fan-in | Параллельные субагенты, родитель собирает | Обновление 5 микросервисов | Claude Code (параллельно), Kilo Code (последовательно) |
```

- [ ] **Step 5: Fix Supervisor row in the summary table**

```
old_string:
| Supervisor | Делегация → проверка → принятие/возврат | Ревью кода субагента | Claude Code, Roo/Kilo (Orchestrator) |

new_string:
| Supervisor | Делегация → проверка → принятие/возврат | Ревью кода субагента | Claude Code, Kilo Code |
```

- [ ] **Step 6: Verify**

Run Grep for `Roo Code / Kilo Code` within § 4 (lines 310-420). Expected: 0 matches.
Run Grep for `Roo/Kilo` in the content file. Expected: 0 matches within § 4.
Run Grep for `Boomerang` within § 4. Expected: 0 matches.

- [ ] **Step 7: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: update § 4 pattern support notes to Kilo Code only"
```

### Task 13: Fix § 5.2 scenarios 2 and 4

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 5.2, lines ~434-450)

- [ ] **Step 1: Fix scenario 2 (stderr vs JSON stdout)**

```
old_string:
2. **Линтинг после записи файла** — ESLint, ruff, golangci-lint. Результат возвращается модели через stderr — она видит ошибки и исправляет их в следующем шаге.

new_string:
2. **Линтинг после записи файла** — ESLint, ruff, golangci-lint. Результат можно вернуть модели двумя способами: как неблокирующую обратную связь (JSON stdout + exit 0 — модель видит сообщения, но может продолжать) или как блокирующую (stderr + exit 2 — действие отклонено, модель должна исправить и повторить).
```

- [ ] **Step 2: Fix scenario 4 (Notification event mapping)**

```
old_string:
4. **Уведомление при ошибке** — Slack webhook, системный звук, push-нотификация. Если агент работает в фоне 20 минут — вы узнаете о проблеме сразу.

new_string:
4. **Уведомление при ошибке** — Slack webhook, системный звук, push-нотификация. Для ошибок подходят события `PostToolUseFailure` / `StopFailure`, а не `Notification` (оно срабатывает на внутренние UI-события — запрос permission, idle, auth). Если агент работает в фоне 20 минут, вы узнаете о проблеме сразу.
```

- [ ] **Step 3: Verify**

Run Grep for `через stderr — она видит` in the file. Expected: 0 matches.
Run Grep for `PostToolUseFailure` in the file. Expected: at least 1 match.

- [ ] **Step 4: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: fix § 5.2 scenarios 2 (stderr/JSON stdout) and 4 (Notification event mapping)"
```

### Task 14: Fix § 5.3 — env vars and exit code 1

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 5.3, lines ~481-491)

- [ ] **Step 1: Replace the env vars bullet**

```
old_string:
- **Переменные окружения** — `$FILEPATH`, `$TOOL_NAME`, `$TOOL_INPUT` и другие. Доступны внутри shell-команды.

new_string:
- **Входной контекст** — данные хука приходят как JSON в stdin: `tool_name`, `tool_input`, `tool_response`. Парсятся через `jq`. Пример: `jq -r '.tool_input.file_path'` извлекает путь к файлу. Shell-переменные хоста (`$CLAUDE_PROJECT_DIR`, `$CLAUDE_PLUGIN_ROOT`) доступны внутри команды.
```

- [ ] **Step 2: Refine exit code 1 description**

```
old_string:
  - `1` — ошибка (неблокирующая, подробности в verbose-режиме)

new_string:
  - `1` — ошибка (неблокирующая, подробности в debug-логе и одной строкой в transcript)
```

- [ ] **Step 3: Verify**

Run Grep for `$FILEPATH` in the content file. Expected: 0 matches.
Run Grep for `\$CLAUDE_PROJECT_DIR` in the content file. Expected: at least 1 match.
Run Grep for `verbose-режиме` in the file. Expected: 0 matches.

- [ ] **Step 4: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: fix § 5.3 — hook input is JSON on stdin, not shell env vars"
```

### Task 15: Fix § 5.4 — TS plugin snippet destructuring and permission.ask

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 5.4, lines ~500-522)

- [ ] **Step 1: Fix the destructuring and input.args bug**

```
old_string:
export const AutoFormatPlugin = async ({ $ }) => {
  return {
    "tool.execute.after": async (input, output) => {
      if (input.tool === "write" || input.tool === "edit") {
        const filePath = output.args?.filePath || ""
        if (filePath.match(/\.(ts|tsx|js|jsx)$/)) {
          await $`npx prettier --write ${filePath}`
        }
      }
    },
    "session.idle": async () => {
      await $`osascript -e 'display notification "Done!" with title "OpenCode"'`
    }
  }
}

new_string:
export const AutoFormatPlugin = async ({ $, project, client, directory, worktree }) => {
  return {
    "tool.execute.after": async (input, output) => {
      if (input.tool === "write" || input.tool === "edit") {
        const filePath = input.args?.filePath || ""
        if (filePath.match(/\.(ts|tsx|js|jsx)$/)) {
          await $`npx prettier --write ${filePath}`
        }
      }
    },
    "session.idle": async () => {
      await $`osascript -e 'display notification "Done!" with title "OpenCode"'`
    }
  }
}
```

- [ ] **Step 2: Fix permission.asked → permission.ask in the event list**

```
old_string:
- **Основные события** — `tool.execute.before`, `tool.execute.after`, `session.created`, `session.idle`, `session.error`, `file.edited`, `permission.asked`, `shell.env` и другие

new_string:
- **Основные события** — `tool.execute.before`, `tool.execute.after`, `session.created`, `session.idle`, `session.error`, `file.edited`, `permission.ask`, `shell.env` и другие
```

- [ ] **Step 3: Verify**

Run Grep for `async ({ $ })` in the content file. Expected: 0 matches.
Run Grep for `output.args\?\.filePath` in the file. Expected: 0 matches.
Run Grep for `permission.asked` in the file. Expected: 0 matches.
Run Grep for `permission.ask\b` in the file. Expected: at least 1 match.

- [ ] **Step 4: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: fix § 5.4 TS plugin snippet — full destructuring, input.args, permission.ask"
```

### Task 16: Add NEW § 5.4.1 — oh-my-opencode mini-subsection

**Files:**
- Modify: `content/part4_subagents_hooks.md` (insert inside § 5.4, before `### 5.5`)

**Depends on Task 1** — actual content uses verified data from the oh-my-opencode research.

- [ ] **Step 1: Insert the new mini-subsection before § 5.5**

```
old_string:
**Сравнение с Claude Code:** больше гибкости (условная логика, асинхронные вызовы, API-интеграции), но выше порог входа — нужно писать код на TypeScript вместо JSON-конфига.

### 5.5 Roo Code / Kilo Code — эмуляция

new_string:
**Сравнение с Claude Code:** больше гибкости (условная логика, асинхронные вызовы, API-интеграции), но выше порог входа — нужно писать код на TypeScript вместо JSON-конфига.

#### oh-my-opencode — готовый набор плагинов

Если не хочется собирать конфигурацию с нуля, есть курируемые наборы. **oh-my-opencode** — сообщественный плагин-пак для OpenCode, добавляющий сразу коллекцию специализированных агентов, подключённые MCP-серверы, готовые hooks и утилитарные tools. Устанавливается одной командой:

```bash
bunx oh-my-opencode install
```

После установки конфигурация появляется в `.opencode/oh-my-opencode.json` (проектный уровень) или `~/.config/opencode/oh-my-opencode.json` (глобальный).

**Что получает пользователь из коробки:**

- **Готовые агенты** под типовые роли: оркестратор, deep worker, researcher, code reviewer и другие.
- **Подключённые MCP-серверы** для веб-поиска, поиска по документации и поиска по публичному коду.
- **Встроенные hooks** — автоформат, уведомления, git-утилиты.
- **Команда `ultrawork`** — запускает полный коллаборативный режим нескольких агентов для сложных задач.

Это быстрый способ попробовать многогагентную оркестрацию в OpenCode, не настраивая всё вручную. Для обучения полезно сначала пройти через ручную настройку из основной части § 5.4 — тогда oh-my-opencode воспринимается как набор best practices, а не как замена пониманию механизма.

Репозиторий: https://github.com/code-yeongyu/oh-my-openagent (фактический URL проверяется на этапе реализации — см. Task 1 этого плана).

### 5.5 Roo Code / Kilo Code — эмуляция
```

**NOTE:** the `### 5.5 Roo Code / Kilo Code — эмуляция` heading is preserved as-is in this step because Task 17 will rewrite it. Don't touch it here.

- [ ] **Step 2: Verify**

Run Grep for `oh-my-opencode` in the content file. Expected: at least 2 matches (the subsection heading and the install command).
Run Grep for `#### oh-my-opencode` in the file. Expected: 1 match.

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: add § 5.4.1 oh-my-opencode mini-subsection"
```

### Task 17: Rewrite § 5.5 as Kilo Code emulation (drop Roo framing)

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 5.5, lines ~529-584)

- [ ] **Step 1: Replace the section title and intro**

```
old_string:
### 5.5 Roo Code / Kilo Code — эмуляция

Roo Code и Kilo Code **не имеют нативных hooks**. Единственный путь — эмуляция через комбинацию кастомного MCP-инструмента и строгих правил в инструкциях режима.

new_string:
### 5.5 Kilo Code — эмуляция

Kilo Code **не имеет нативных hooks**. Единственный путь — эмуляция через комбинацию кастомного MCP-инструмента и строгих правил в инструкциях агента.
```

- [ ] **Step 2: Update Step 2 rule path**

```
old_string:
**Шаг 2. Правило для режима `code`** (`.roo/rules-code/lint-before-write.md`):

new_string:
**Шаг 2. Правило для агента `code`** (внутри `customInstructions` в `.kilocodemodes` или в теле `.kilo/agents/code.md`):
```

- [ ] **Step 3: Rewrite `.roomodes` JSON snippet as `.kilocodemodes` YAML**

```
old_string:
**Шаг 3. Кастомный режим `safe-code`** (`.roomodes`):

```json
{
  "customModes": [{
    "slug": "safe-code",
    "name": "Safe Code",
    "roleDefinition": "Developer with mandatory lint checks before edits",
    "groups": ["read", ["edit", { "fileRegex": "\\.(ts|tsx|js|jsx)$" }], "command", "mcp"],
    "customInstructions": "Always call lint_check before any file modification."
  }]
}
```

new_string:
**Шаг 3. Кастомный агент `safe-code`** (`.kilocodemodes`, YAML):

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
```

- [ ] **Step 4: Verify**

Run Grep for `Roo Code` within lines 500-600. Expected: 0 matches.
Run Grep for `.roomodes` in § 5.5. Expected: 0 matches.
Run Grep for `.roo/rules-code` in the file. Expected: 0 matches.
Run Grep for `.kilocodemodes` in § 5.5. Expected: at least 1 match.

- [ ] **Step 5: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: rewrite § 5.5 as Kilo Code emulation (drop Roo framing)"
```

### Task 18: Fix § 5.6 hooks comparison table — column relabel

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 5.6, lines ~588-594)

- [ ] **Step 1: Rename the column header**

```
old_string:
| Критерий | Claude Code | OpenCode | Roo Code / Kilo Code |

new_string:
| Критерий | Claude Code | OpenCode | Kilo Code |
```

- [ ] **Step 2: Verify**

Run Grep for `Roo Code / Kilo Code` in the content file. Expected: 0 matches (now all tables relabeled; next tasks handle prose).

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: rename § 5.6 column Roo/Kilo → Kilo Code"
```

### Task 19: Fix § 5.7 strategy bullet — Roo/Kilo → Kilo

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 5.7, line ~604)

- [ ] **Step 1: Replace the bullet**

```
old_string:
- **Roo Code / Kilo Code** → эмуляция с оговорками

new_string:
- **Kilo Code** → эмуляция с оговорками
```

- [ ] **Step 2: Verify**

Run Grep for `**Roo` in the file. Expected: 0 matches.

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: § 5.7 — rename strategy bullet to Kilo Code"
```

### Task 20: Cleanup § 6-7 Roo references and practice assignment

**Files:**
- Modify: `content/part4_subagents_hooks.md` (§ 6-7, lines ~660-693)

- [ ] **Step 1: Find all remaining Roo references**

Run Grep for `Roo` in the file. Expected: some matches in § 6-7.

- [ ] **Step 2: Fix practice assignment line 689**

```
old_string:
- **Roo/Kilo Code:** создайте MCP-инструмент `pre_edit_check` и правило в `.roo/rules-code/`, которое обязывает агента вызывать проверку перед каждым редактированием. Протестируйте: попросите агента изменить файл и убедитесь, что проверка вызывается.

new_string:
- **Kilo Code:** создайте MCP-инструмент `pre_edit_check` и правило в `customInstructions` внутри `.kilocodemodes` (или в теле `.kilo/agents/code.md`), которое обязывает агента вызывать проверку перед каждым редактированием. Протестируйте: попросите агента изменить файл и убедитесь, что проверка вызывается.
```

- [ ] **Step 3: Fix any remaining § 6-7 Roo mentions**

For each remaining Grep match in § 6-7 (summary, pitfalls, practice), replace `Roo/Kilo Code` or `Roo Code / Kilo Code` with `Kilo Code`. Where the text specifically describes a Roo-only feature (Orchestrator, Boomerang, `.roomodes`, Task Manager UI), either remove the reference or rewrite under Kilo's current terminology.

- [ ] **Step 4: Verify**

Run Grep for `Roo` in the content file. Expected: 0 matches.

- [ ] **Step 5: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "lecture4: § 6-7 cleanup — drop remaining Roo references, update practice assignment"
```

### Task 21: Content pass complete — user review checkpoint

**Files:**
- Modify: (none)
- Verify: `content/part4_subagents_hooks.md` opens in the markdown reader without errors

- [ ] **Step 1: Full-file Grep sanity checks**

Run these and verify expected results:

- `Grep "Roo Code"` — expect **0**.
- `Grep "Roo/Kilo"` — expect **0**.
- `Grep "Orchestrator"` — expect **0**.
- `Grep "Boomerang"` — expect **0**.
- `Grep ".roomodes"` — expect **0**.
- `Grep ".roo/rules-code"` — expect **0**.
- `Grep "run_in_background: true"` — expect **0**.
- `Grep "permission.asked"` — expect **0**.
- `Grep "claude-sonnet-4-20250514"` — expect **0**.
- `Grep "### 3.5 Ограничение доступа"` — expect **1**.
- `Grep "### 3.6 Выбор модели"` — expect **1**.
- `Grep "#### oh-my-opencode"` — expect **1**.
- `Grep "Claude Sonnet —"` — expect **0** (old mention gone).

- [ ] **Step 2: Open in the markdown reader**

Open `content/part4_subagents_hooks.md` in the project's markdown reader (HTTP, not file://). Verify:
- No rendering errors
- All code blocks have correct language tags
- Tables render correctly
- New sections (§ 3.5, § 3.6, § 5.4.1) are visible in the TOC if the reader has one

- [ ] **Step 3: Pause for user review**

Notify the user:
> "Content edits to `content/part4_subagents_hooks.md` are complete. All 15 fact-check fixes plus three new subsections (§ 3.5, § 3.6, § 5.4.1) are in place. Please review the markdown and let me know if anything needs to change before I start mirroring to the slides."

Wait for the user's response. If they request changes, apply them in a new set of commits, then repeat this verification. Only proceed to Phase 2 once the user approves.

---

## Phase 2: Slides mirror (`presentations/part4/index.html`)

Phase 2 runs **only after** Phase 1 is approved by the user. It is a mechanical mirror of the content edits into the Reveal.js presentation.

### Task 22: Fix TOC card and intro slide

**Files:**
- Modify: `presentations/part4/index.html` (lines ~700-735)

- [ ] **Step 1: Drop Roo Code from the TOC card**

Find the TOC card text mentioning the tool names (around line 706). Replace `Claude Code, Roo Code, Kilo Code, OpenCode` with `Claude Code, Kilo Code, OpenCode`.

- [ ] **Step 2: Fix intro paragraph about context window**

Find the paragraph at line ~730 containing `Claude Sonnet — 200K`. Replace with the same frontier-models phrasing used in content:

```
old_string:
<p><strong>Контекстное окно агента — ресурс конечный.</strong> У Claude Sonnet — 200K токенов, но эффективность падает задолго до лимита.</p>

new_string:
<p><strong>Контекстное окно агента — ресурс конечный.</strong> У современных frontier-моделей (Claude Opus/Sonnet 4.6, Haiku 4.5, GLM-4.6, Qwen-Coder-Next) — 200K+ токенов, но эффективность падает задолго до лимита.</p>
```

- [ ] **Step 3: Verify**

Run Grep for `Claude Sonnet — 200K` in the slides file. Expected: 0 matches.
Run Grep for `Roo Code` in lines 700-720. Expected: 0 matches.

- [ ] **Step 4: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: drop Roo Code from TOC, neutralize context window mention"
```

### Task 23: Fix Claude Code slides — AGENTS.md framing and `background`

**Files:**
- Modify: `presentations/part4/index.html` (lines ~985-1100)

- [ ] **Step 1: Fix Agent tool params list**

Find the line listing `run_in_background` in the Agent tool params block (around line 993). Replace with `background: true`:

```
old_string:
<li><code>run_in_background</code> — запуск в фоновом режиме</li>

new_string:
<li><code>background</code> — запуск в фоновом режиме (<code>true</code>/<code>false</code>)</li>
```

- [ ] **Step 2: Fix "Из AGENTS.md / .claude/agents/" line**

Find the text at line ~1026 with `Из AGENTS.md / .claude/agents/`. Replace with `Из .claude/agents/<name>.md`.

- [ ] **Step 3: Rewrite the "Claude Code — AGENTS.md" slide (lines 1038-1062)**

Locate the slide titled `Claude Code — AGENTS.md` (or equivalent). Rewrite the slide to:
- Title: `Claude Code — .claude/agents/<name>.md`
- Body: YAML frontmatter example matching the content file (from Task 7, Step 2)
- Key points: `name`, `description`, `tools`, `model` fields

Use the same YAML example drafted in Task 7 of this plan. If the slide uses a code block with Highlight.js, keep the `markdown` or `yaml` language tag.

- [ ] **Step 4: Fix `run_in_background: true` card at line ~1087**

Find the card showing `run_in_background: true`. Replace the code text with `background: true`.

- [ ] **Step 5: Fix trust-agent card at line ~1295**

Find the card mentioning `AGENTS.md` in the trust-agent context. Update to `.claude/agents/<name>.md`.

- [ ] **Step 6: Verify**

Run Grep for `run_in_background` in the slides file. Expected: 0 matches.
Run Grep for `AGENTS.md` in the slides file, excluding any intentional "this is NOT the same as AGENTS.md" explanatory notes. Expected: 0 matches (or only intentional ones).

- [ ] **Step 7: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: fix Claude Code slides — .claude/agents/ and background field"
```

### Task 24: Rewrite Kilo Code slides (drop Roo, new content)

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1100-1160)

- [ ] **Step 1: Rewrite "Roo Code / Kilo Code — режимы" slide**

Locate the slide (around line 1100). Change:
- Title: `Kilo Code — встроенные агенты`
- List of modes: change to `code`, `plan`, `ask`, `debug` (drop Orchestrator, rename Architect → plan)
- Remove any card describing "Orchestrator + Boomerang Tasks" as the primary subagency mechanism

Use the text from Task 8 of this plan (Step 1) as the source.

- [ ] **Step 2: Rewrite "Roo Code / Kilo Code — .roomodes" slide**

Locate the slide (around line 1136). Change:
- Title: `Kilo Code — .kilocodemodes / .kilo/agents/`
- Code block: use the YAML `.kilocodemodes` example from Task 8, Step 1
- Remove the Task Manager UI card entirely (not documented in current Kilo docs)
- Replace `groups` values `["read", "edit", "command", "browser", "mcp"]` with the Claude Code-style list from Task 8

- [ ] **Step 3: Verify**

Run Grep for `Roo Code` in lines 1080-1180 of the slides file. Expected: 0 matches.
Run Grep for `.roomodes` in the slides file. Expected: 0 matches (or only historical note, if any).
Run Grep for `Orchestrator` in the slides file. Expected: 0 matches.
Run Grep for `Task Manager UI` in the slides file. Expected: 0 matches.
Run Grep for `.kilocodemodes` in the slides file. Expected: at least 1 match.

- [ ] **Step 4: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: rewrite Kilo Code slides (drop Roo, new agents, .kilocodemodes)"
```

### Task 25: Update § 3.3 OpenCode slide (model ID + verified format)

**Files:**
- Modify: `presentations/part4/index.html` (OpenCode section, verify actual lines via Grep for `opencode.json` in the file)

- [ ] **Step 1: Locate the OpenCode agent config slide**

Run Grep for `anthropic:claude-sonnet-4-20250514` in the slides file. Note the line numbers.

- [ ] **Step 2: Update model IDs**

Replace every occurrence of `anthropic:claude-sonnet-4-20250514` with `anthropic:claude-sonnet-4-6`.

- [ ] **Step 3: Apply findings from Task 5**

If Task 5 discovered that `prompt` or `permission` field names changed, apply those corrections here matching the content file from Task 9.

- [ ] **Step 4: Verify**

Run Grep for `claude-sonnet-4-20250514` in the slides file. Expected: 0 matches.

- [ ] **Step 5: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: update OpenCode snippet model IDs"
```

### Task 26: Fix § 3.4 comparison table slide

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1195-1253)

- [ ] **Step 1: Update the comparison table**

Find the HTML table with the columns Claude Code / Roo Code / Kilo Code / OpenCode. Apply the same cell-level corrections as in content Task 10:

- Column header `Roo Code / Kilo Code` → `Kilo Code`
- `Модель субагентности` CC: `Автоматическая делегация через Agent tool (явный вызов @-mention)`
- `Конфигурация` CC: `.claude/agents/<name>.md + subagent_type`
- `Параллельность` OpenCode: `Нет (профили; внешняя оркестровка)`
- `Кастомные типы` CC: `.claude/agents/<name>.md`
- `Фоновый запуск` CC: `background: true`
- Kilo column — all cells rewritten per Task 10

- [ ] **Step 2: Verify**

Run Grep for `run_in_background: true` in the slides file. Expected: 0 matches.
Run Grep for `Roo Code / Kilo Code` in the slides file. Expected: 0 matches.

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: fix § 3.4 comparison table cells"
```

### Task 27: Add NEW slides for § 3.5 tools/MCP scoping

**Files:**
- Modify: `presentations/part4/index.html` (insert new section after the comparison table slide)

- [ ] **Step 1: Add a section divider slide**

Insert a new `<section>` with a title slide: `3.5 Ограничение доступа: tools и MCP`.

- [ ] **Step 2: Add principle slide**

Add a `<section>` card containing the three-point principle from content Task 11:
- attack surface reduction
- predictable behavior
- context savings

Subtitle: `Каждому субагенту — только нужное`.

- [ ] **Step 3: Add three tool-specific slides (CC / Kilo / OpenCode)**

Each slide is a card with:
- Header: tool name
- Code block: the YAML/JSON snippet from content Task 11
- Below: one-line explanation

Use the Highlight.js language tags `markdown` (CC frontmatter), `yaml` (Kilo), `json` (OpenCode).

- [ ] **Step 4: Add "что ограничивать" checklist slide**

Four-item card with the bullet list from content Task 11.

- [ ] **Step 5: Verify**

Open the slides in a browser (HTTP, not file://). Navigate to the new § 3.5 section. Verify all code blocks render, no layout breakage.

- [ ] **Step 6: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: add § 3.5 tools/MCP scoping slides"
```

### Task 28: Add NEW slides for § 3.6 model choice

**Files:**
- Modify: `presentations/part4/index.html` (insert after § 3.5 slides)

- [ ] **Step 1: Add section divider slide**

Title: `3.6 Выбор модели для субагента`.

- [ ] **Step 2: Add model comparison table slide**

HTML `<table>` with columns: `Модель | Контекст | Сильные стороны`. Rows match content Task 11 (§ 3.6 draft). Use `(проверить)` in any cell that Task 2 could not verify.

- [ ] **Step 3: Add pros/cons slide**

Two-column card: `Плюсы лёгких моделей` (4 bullets) | `Минусы лёгких моделей` (4 bullets). Text from content Task 11 § 3.6.

- [ ] **Step 4: Add role recommendations slide**

HTML table with two columns: `Роль | Модель`. Rows match content Task 11 § 3.6 recommendations table.

- [ ] **Step 5: Add "где конфигурируется" card**

Three bullets: Claude Code / Kilo Code / OpenCode, each showing where `model:` is set.

- [ ] **Step 6: Verify**

Open in browser. Navigate to § 3.6 slides. Verify tables, layout, no breakage.

- [ ] **Step 7: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: add § 3.6 model choice slides"
```

### Task 29: Fix § 4 pattern slides — Roo/Kilo → Kilo

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1307-1417)

- [ ] **Step 1: Fan-out pattern slide**

Find the Fan-out card at ~line 1307. Replace the Roo/Kilo Boomerang reference with Kilo Code (sequential delegation).

```
old_string (text block in the slide):
Roo Code / Kilo Code — только последовательный Boomerang

new_string:
Kilo Code — только последовательная делегация
```

- [ ] **Step 2: Specialist pattern slide**

Find the Specialist card at ~line 1369. Replace `.roomodes` references with `.kilocodemodes` and `AGENTS.md` references with `.claude/agents/`.

- [ ] **Step 3: Supervisor pattern slide**

Find the Supervisor card at ~line 1417. Replace `Orchestrator + Boomerang` reference with plain Kilo Code delegation.

- [ ] **Step 4: Verify**

Run Grep for `Roo Code / Kilo Code` in lines 1300-1420. Expected: 0 matches.
Run Grep for `Boomerang` in lines 1300-1420. Expected: 0 matches.

- [ ] **Step 5: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: update § 4 pattern slides — Kilo Code only"
```

### Task 30: Fix § 5.3 Claude Code hooks slides — env vars

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1494-1565)

- [ ] **Step 1: Fix the env vars block in the Claude Code hooks slide**

Find the slide listing `$FILEPATH`, `$TOOL_NAME`, `$TOOL_INPUT` as shell env vars.

Replace the block with the JSON-on-stdin explanation from content Task 14, plus the `$CLAUDE_PROJECT_DIR` host env var.

- [ ] **Step 2: Refine exit code 1 description**

Find the text `verbose-режиме` in the exit codes list. Replace with `debug-логе и одной строкой в transcript`.

- [ ] **Step 3: Verify**

Run Grep for `\$FILEPATH` in the slides file. Expected: 0 matches (or only inside a ``jq`` example showing stdin parsing, in which case the match is intentional).
Run Grep for `\$CLAUDE_PROJECT_DIR` in the slides file. Expected: at least 1 match.

- [ ] **Step 4: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: fix § 5.3 hook input — JSON on stdin, not shell env vars"
```

### Task 31: Fix § 5.4 OpenCode plugins slide — TS snippet and `permission.ask`

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1568-1607)

- [ ] **Step 1: Fix the TS plugin snippet destructuring**

Find the TS code block in the OpenCode plugins slide. Apply the same fix as content Task 15: full destructuring `{ $, project, client, directory, worktree }` and `input.args?.filePath` instead of `output.args?.filePath`.

- [ ] **Step 2: Fix `permission.asked` → `permission.ask`**

Run Grep for `permission.asked` in the slides file. For each match, replace with `permission.ask`.

- [ ] **Step 3: Verify**

Run Grep for `async ({ \$ })` in the slides file. Expected: 0 matches.
Run Grep for `output.args\?\.filePath` in the slides file. Expected: 0 matches.
Run Grep for `permission.asked` in the slides file. Expected: 0 matches.

- [ ] **Step 4: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: fix § 5.4 TS snippet destructuring + permission.ask"
```

### Task 32: Add NEW § 5.4.1 oh-my-opencode slide

**Files:**
- Modify: `presentations/part4/index.html` (insert after the OpenCode plugins slide, before § 5.5)

- [ ] **Step 1: Add the mini-subsection slide**

New slide with:
- Title: `oh-my-opencode — готовый набор плагинов`
- Install code block: `bunx oh-my-opencode install`
- Bullet list: что из коробки (agents, MCP, hooks, ultrawork command)
- Footer: repository URL

Content matches content Task 16.

- [ ] **Step 2: Verify**

Open the slides in a browser. Navigate to the new slide. Verify layout.

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: add § 5.4.1 oh-my-opencode slide"
```

### Task 33: Rewrite § 5.5 Kilo Code emulation slide

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1609-1652)

- [ ] **Step 1: Rewrite the Roo/Kilo emulation slide**

Find the slide titled `Roo Code / Kilo Code — эмуляция hooks` (or similar). Change to:
- Title: `Kilo Code — эмуляция hooks`
- Drop all Roo framing
- Replace any `.roomodes` code with `.kilocodemodes` YAML (from content Task 17)
- Replace `.roo/rules-code/` path references with `customInstructions` in `.kilocodemodes` or body of `.kilo/agents/code.md`

- [ ] **Step 2: Verify**

Run Grep for `.roomodes` in lines 1600-1660. Expected: 0 matches.
Run Grep for `.roo/rules-code` in the slides file. Expected: 0 matches.

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: rewrite § 5.5 Kilo Code emulation (drop Roo)"
```

### Task 34: Fix § 5.6 and § 5.7 — column/bullet renames

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1654-1736)

- [ ] **Step 1: Fix § 5.6 hooks comparison table**

Find the table. Rename column `Roo Code / Kilo Code` → `Kilo Code`.

- [ ] **Step 2: Fix § 5.7 strategy slide**

Find the Kilo/Roo card. Rename to `Kilo Code` (drop Roo).

- [ ] **Step 3: Verify**

Run Grep for `Roo Code / Kilo Code` in lines 1640-1740. Expected: 0 matches.

- [ ] **Step 4: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: rename § 5.6 column and § 5.7 bullet — Kilo Code"
```

### Task 35: Fix § 6-7 summary and practice slides

**Files:**
- Modify: `presentations/part4/index.html` (lines ~1816-1874)

- [ ] **Step 1: Find remaining Roo mentions in the slides file**

Run Grep for `Roo` in `presentations/part4/index.html`. Expected: some matches in § 6-7 slides.

- [ ] **Step 2: For each match, apply the Kilo-only rewrite**

Use the same conventions as content Task 20: drop Roo, relabel `Roo/Kilo Code` → `Kilo Code`, replace `.roo/rules-code/` with `customInstructions` in `.kilocodemodes`.

- [ ] **Step 3: Verify**

Run Grep for `Roo` in the slides file. Expected: 0 matches.

- [ ] **Step 4: Commit**

```bash
git add presentations/part4/index.html
git commit -m "lecture4 slides: § 6-7 cleanup — drop remaining Roo references"
```

### Task 36: Final slides verification

**Files:**
- Verify: `presentations/part4/index.html` renders in Reveal.js

- [ ] **Step 1: Full-file Grep sanity checks**

Run these and verify expected results on `presentations/part4/index.html`:

- `Grep "Roo Code"` — expect **0**.
- `Grep "Roo/Kilo"` — expect **0**.
- `Grep "Orchestrator"` — expect **0**.
- `Grep "Boomerang"` — expect **0**.
- `Grep ".roomodes"` — expect **0**.
- `Grep ".roo/rules-code"` — expect **0**.
- `Grep "run_in_background"` — expect **0**.
- `Grep "permission.asked"` — expect **0**.
- `Grep "claude-sonnet-4-20250514"` — expect **0**.
- `Grep "Claude Sonnet — 200K"` — expect **0**.
- `Grep "oh-my-opencode"` — expect **at least 1**.
- `Grep "3.5 Ограничение доступа"` — expect **at least 1**.
- `Grep "3.6 Выбор модели"` — expect **at least 1**.

- [ ] **Step 2: Open the Reveal.js presentation in a browser**

Serve the file via HTTP (not file://). Navigate through all sections. Verify:
- No JavaScript errors in the console
- All code blocks render with syntax highlighting
- New slides (§ 3.5, § 3.6, § 5.4.1) are present and navigable
- No broken layouts on the rewritten Kilo slides
- Comparison tables render correctly

- [ ] **Step 3: Report to user**

Notify the user:
> "Slides mirror complete. All 15 fact-check fixes and three new subsections are now reflected in `presentations/part4/index.html`. The lecture is ready for end-to-end review."

---

## Phase 3: Final cleanup

### Task 37: Update the stale `project_lecture4_in_progress.md` memory

**Files:**
- Modify: `/Users/sazonov/.claude/projects/-Users-sazonov-Desktop-AIpresentation/memory/project_lecture4_in_progress.md`

- [ ] **Step 1: Delete the stale memory**

The `project_lecture4_in_progress.md` file says "лекция 4 ведётся параллельным агентом — не трогать её файлы". This guard became obsolete once the user explicitly authorized lecture 4 edits in this conversation.

Delete the memory file.

- [ ] **Step 2: Remove the pointer from MEMORY.md**

Edit `/Users/sazonov/.claude/projects/-Users-sazonov-Desktop-AIpresentation/memory/MEMORY.md` and remove the line pointing to `project_lecture4_in_progress.md`.

- [ ] **Step 3: Verify**

Check that `MEMORY.md` no longer references the deleted file.

- [ ] **Step 4: Commit (optional — memory files may live outside git)**

Memory directory is under `~/.claude/projects/` — typically not in this project's git. No commit needed; the edit persists locally.

---

## Out of scope

- **Rewriting the "AI-sounding" tone of § 5** (todo item 4) — deferred to a separate pass.
- **Other lectures** (`content/part1_foundations.md`, lecture 5, etc.) — not touched.
- **New fact-checks** beyond the 15 v2 issues — not performed in this pass.
