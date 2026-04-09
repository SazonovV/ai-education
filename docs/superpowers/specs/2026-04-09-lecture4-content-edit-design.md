# Лекция 4 — правки контента (fact-check v2 + todo.md) — дизайн

**Дата:** 2026-04-09
**Автор:** Claude (main)
**Статус:** Draft, ожидает ревью пользователя

## Цель

Привести Лекцию 4 «Субагенты, оркестрация и Hooks» в актуальное состояние: исправить 15 неточностей из fact-check отчёта v2 и добавить три новых содержательных блока, запрошенных в `todo.md` (выбор модели для субагента, ограничение tools/MCP, oh-my-opencode).

## Источники требований

1. **`docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`** — 15 правок (7 HIGH, 6 MEDIUM, 2 LOW) по § 3.2, § 3.4, § 5.2, § 5.3, § 5.4, § 5.5 лекции.
2. **`todo.md`** — 6 пунктов по Лекции 4:
   1. Убрать упоминание Claude Sonnet, добавить GLM / Qwen-Coder-Next / актуальные Opus-Sonnet.
   2. Фактчек сравнительной таблицы — закрывается отчётом v2.
   3. Добавить информацию про oh-my-opencode.
   4. Hooks «ИИшное описание» — **отложено, отдельный пасс**.
   5. Фактчек hooks — закрывается отчётом v2.
   6. Добавить информацию, как конкретным субагентам давать доступ к MCP и tools.
3. **`.claude/projects/-Users-sazonov-Desktop-AIpresentation/memory/project_lecture4_drop_roocode.md`** — решение от 2026-04-08/09 убрать Roo Code из лекции, оставить Kilo Code standalone.

## Архитектура

### Два файла, один поток

- **`content/part4_subagents_hooks.md`** — источник истины, markdown, читается через плеер. Правится **первым** полностью.
- **`presentations/part4/index.html`** — презентационный слой Reveal.js. Правится **после** одобрения контента, механическим зеркалированием.

### Поток работ (Подход 3 — content → review → slides)

```
Правки в content/part4_subagents_hooks.md (несколько коммитов по смысловым блокам)
        ↓
Пауза на ревью markdown пользователем
        ↓
Одобрено?
  ├── нет → корректировка content
  └── да → зеркалирование в presentations/part4/index.html (один проход)
        ↓
Коммит slides-mirror
        ↓
Готово
```

Обоснование: контент — источник истины; слайды — производный слой. Ревью markdown проще и компактнее, чем одновременное ревью двух файлов; зеркалирование становится механической задачей с минимумом риска пропустить что-то.

## Карта изменений в `content/part4_subagents_hooks.md`

### § 1 Введение

- **Строка 26.** Одна фраза про контекстное окно. Замена:
  - Было: `У Claude Sonnet — 200K токенов, но эффективность падает задолго до лимита.`
  - Станет: `У современных frontier-моделей (Claude Opus/Sonnet 4.6, Haiku 4.5, GLM-4.6, Qwen-Coder-Next) — 200K+ токенов, но эффективность падает задолго до лимита.`
- Подробный разбор моделей уходит в новый § 3.6.

### § 3 Субагенты в инструментах — существующие подразделы

- **§ 3.1 Claude Code** — правка `AGENTS.md` framing (расширение fix из § 3.4 на корневой источник).
  - **Строка 170.** Таблица типов субагентов, строка `Кастомный (из AGENTS.md)` → `Кастомный (из .claude/agents/<name>.md)`.
  - **Строки 172-192.** Прозовый блок «**AGENTS.md** — файл в корне репозитория…» — переписать: кастомные субагенты живут в `.claude/agents/<name>.md` (Markdown + YAML frontmatter); `AGENTS.md` — это отдельный файл проектной памяти, не реестр субагентов. Пример YAML frontmatter субагента (с полями `name`, `description`, `tools`, `model`) заменяет текущий прозовый формат `Tools: Read, Grep, Glob`.

- **§ 3.2** — полный переписанный раздел, переименованный в **«Kilo Code»**. Drop Roo Code framing полностью.
  - Подача: современный CLI/VSCode-агент со встроенными агентами `code`, `plan`, `ask`, `debug`; нативная поддержка субагентов без отдельного Orchestrator.
  - Конфигурация: `.kilocodemodes` (YAML предпочтителен, JSON совместим) для проектных агентов, `.kilo/agents/<name>.md` (Markdown + YAML frontmatter) как альтернативный формат, `custom_modes.yaml` для глобальных.
  - Убрать упоминания: Orchestrator (deprecated), Boomerang Tasks (нет feature-страницы), Task Manager UI (не задокументирован).
  - Обновить значения `groups` на Claude Code-style: `read`, `edit`, `bash`, `grep`, `glob`, `list`, `task`, `webfetch`, `websearch`, `codesearch`, `todowrite`, `todoread`.
  - «Architect» теперь «plan» — отразить в описании встроенных агентов.

- **§ 3.3 OpenCode** — верификация и точечные правки.
  - Проверить через WebFetch актуальные имена полей (`prompt`, `permission`) в `opencode.json`.
  - Обновить устаревший ID модели `anthropic:claude-sonnet-4-20250514` на актуальный (`claude-sonnet-4-6` или эквивалент).
  - Проверить форму значений `permission`: массив строк `["read", "glob", ...]` vs словарь `{edit: "allow", bash: "ask", ...}`.
  - При наличии расхождений с current docs — поправить snippet.

- **§ 3.4 Сводная таблица** — правки по ячейкам:
  - Row `Модель субагентности` CC → `Автоматическая делегация через Agent tool (явный вызов @-mention)`
  - Row `Конфигурация` CC → `` `.claude/agents/<name>.md` (Markdown + YAML frontmatter) + `subagent_type` ``
  - Row `Кастомные типы` CC → `` `.claude/agents/<name>.md` ``
  - Row `Фоновый запуск` CC → `` `background: true` ``
  - Row `Параллельность` OpenCode → `Нет (профили; параллелизм — внешняя оркестровка)`
  - Колонка `Roo Code / Kilo Code` → `Kilo Code`. Все ячейки переписаны:
    - Модель субагентности: `Агенты с нативными субагентами`
    - Конфигурация: `.kilocodemodes / .kilo/agents/<name>.md`
    - Параллельность: `Последовательная делегация`
    - Изоляция: `Отдельный контекст (общая FS)`
    - Кастомные типы: `.kilocodemodes / .kilo/agents/<name>.md`
    - Фоновый запуск: `Нет`
  - Пост-табличный абзац — убрать фразы про Orchestrator/Boomerang/Task Manager UI, переформулировать summary под новые ячейки.

### § 3 — два новых подраздела (после § 3.4)

- **§ 3.5 Ограничение доступа: tools и MCP для субагентов** (NEW)
  - Тезис: принцип минимальных прав — субагент получает ровно то, что нужно для его задачи. Избыточные права → расширенная attack surface для prompt injection, менее предсказуемое поведение, больший расход контекста.
  - **Claude Code.** Пример `.claude/agents/researcher.md` с YAML frontmatter: `tools: [Read, Grep, Glob, mcp__sentry__get_issue]`, `model`, `description`. Синтаксис `mcp__<server>__<tool>` для отдельных MCP-инструментов, `mcp__<server>__*` для всего сервера.
  - **Kilo Code.** Пример `.kilocodemodes` с `groups` как массивом tool-имён в Claude Code-style: `[read, edit, grep, glob, webfetch]`. Включение MCP — через раздел `mcp_servers` в том же файле или `.kilo/mcp.yaml`.
  - **OpenCode.** Пример `opencode.json` с `permission` per-agent + включением конкретных MCP-серверов. Синтаксис верифицируется через WebFetch.
  - Короткая сводка «Что ограничивать»: чтение vs запись, bash, webfetch, конкретные MCP-серверы.
  - Закрывающий абзац: почему минимальные права важны именно для субагентов — они работают автономно, ошибка модели не перехватывается человеком между шагами.

- **§ 3.6 Выбор модели для субагента** (NEW)
  - Тезис: субагент — естественное место для осознанной экономии модели, потому что он работает в изолированном контексте с узкой задачей.
  - **Таблица «Модель — сильные стороны — контекст — стоимость».** Строки: Opus 4.6, Sonnet 4.6, Haiku 4.5, GLM-4.6, Qwen-Coder-Next.
  - **Плюсы лёгких моделей:** выше скорость ответа, ниже стоимость за задачу, больше параллельных субагентов в том же бюджете, меньше расход контекста оркестратора на ожидание.
  - **Минусы:** слабее рассуждение в многошаговых задачах, хуже справляются со сложными инструкциями, чаще ошибаются в нетривиальной логике, сильнее чувствительны к нечётким промптам.
  - **Рекомендации по ролям:**
    - Исследование / поиск по коду → Haiku 4.5, GLM-4.6, Qwen-Coder-Next.
    - Генерация тестов, написание кода по спеке → Sonnet 4.6.
    - Рефакторинг ядра, архитектурные решения, планирование → Opus 4.6.
    - Документация, комментарии → Haiku / Qwen.
  - Где это конфигурируется: CC — `model:` в YAML frontmatter; Kilo — `model:` в `.kilocodemodes`; OpenCode — `model:` в `opencode.json` per-agent.

### § 4 Паттерны оркестрации

- Строки 355, 384, 407, 413, 416: `Roo Code / Kilo Code` → `Kilo Code`. Там, где упоминается «Boomerang — последовательный паттерн», заменить на «Kilo Code — последовательная делегация».

### § 5 Hooks — фактчек-правки

- **§ 5.2 сценарий 4** — уточнить mapping событий: для ошибок — `PostToolUseFailure` / `StopFailure`, не `Notification`. `Notification` относится к внутренним UI-событиям (permission prompts, idle, auth).
- **§ 5.2 сценарий 2** — короткая ремарка про варианты возврата: JSON stdout (exit 0) для неблокирующей обратной связи; stderr + exit 2 для блокирующей. Текущая формулировка «через stderr» — упрощение.
- **§ 5.3** — переписать блок «Переменные окружения»:
  - Было: `$FILEPATH`, `$TOOL_NAME`, `$TOOL_INPUT` как shell env vars.
  - Станет: данные хука приходят как JSON в stdin (`tool_name`, `tool_input`, `tool_response`), парсятся через `jq`. Shell-переменные хоста: `$CLAUDE_PROJECT_DIR`, `$CLAUDE_PLUGIN_ROOT`.
  - Короткий пример: `jq -r '.tool_input.file_path'`.
- **§ 5.3 exit code 1** — уточнить описание: «подробности в debug-логе и одной строкой в transcript», вместо «в verbose-режиме».
- **§ 5.4 TypeScript snippet** — две правки:
  - Сигнатура: `async ({ $ }) =>` → `async ({ $, project, client, directory, worktree }) =>`.
  - Внутри хука: `output.args?.filePath` → `input.args?.filePath || ""`. Поле `args` живёт на `input`, не `output`; после-хук получает output-объект формы `{ title, output, metadata }`.
- **§ 5.4 событие** `permission.asked` → `permission.ask`.

### § 5 — новый мини-подраздел

- **§ 5.4.1 oh-my-opencode — готовые плагины** (NEW, вложен в § 5.4 как H4)
  - Что: курируемый набор готовых плагинов и агентов для OpenCode, публикуется через репозиторий.
  - Как подключить: через npm в `opencode.json` или локально через клонирование репозитория в `.opencode/plugins/`.
  - 2-3 конкретных примера плагинов: автоформат, линт-блокировка, notification-plugin.
  - **Фактическое содержание (имя репо, точный синтаксис установки, конкретные примеры) уточняется через WebFetch на этапе реализации плана.** Если WebFetch не даёт достаточно информации — подраздел сокращается до краткого абзаца с ссылкой на репозиторий и общим описанием.

### § 5.5 Kilo Code — эмуляция hooks (переписываем)

- Drop Roo Code framing — раздел остаётся, но полностью про Kilo Code.
- MCP-инструмент `lint_check` — TS snippet остаётся (он корректен как generic MCP tool).
- Убрать путь `.roo/rules-code/lint-before-write.md` → правило живёт в `customInstructions` внутри `.kilocodemodes` или в теле `.kilo/agents/safe-code.md`.
- Переписать `.roomodes` JSON → `.kilocodemodes` YAML с `groups: [read, [edit, {fileRegex: ...}], bash]`. Альтернативный вариант — Markdown `.kilo/agents/safe-code.md`.

### § 5.6 Сводная таблица hooks

- Колонка `Roo Code / Kilo Code` → `Kilo Code`.
- Ячейки большей частью остаются («эмуляция», «нет гарантии», «нет нативных событий», «высокая сложность»), но формулировки пересмотреть под Kilo-терминологию.

### § 5.7 Стратегия выбора

- Буллет `Roo Code / Kilo Code → эмуляция с оговорками…` → `Kilo Code → эмуляция с оговорками…`. Остальной текст буллета остаётся.

### § 6 Pitfalls

- Проверить все упоминания Roo/Kilo в ловушках и примерах → заменить на Kilo.

### § 7 Summary + практика

- Строки 665, 679, 689: `Roo/Kilo Code` → `Kilo Code`.
- Задание на практику про создание кастомного режима — заменить `.roo/rules-code/` на `customInstructions` в `.kilocodemodes`.

## Карта изменений в `presentations/part4/index.html`

Выполняется **после** одобрения `content/part4_subagents_hooks.md`. Задача прохода — механическое зеркалирование.

### Существующие слайды, которые правятся

- **Line 706** — TOC карточка: убрать Roo Code из перечисления.
- **Line 730** — абзац про контекстное окно: замена Sonnet на frontier-модели (идентично `content:26`).
- **Line 993** — список параметров Agent tool: `run_in_background` → `background`.
- **Lines 1026, 1038-1062, 1087, 1295** — слайды Claude Code: AGENTS.md → `.claude/agents/`, `run_in_background` → `background`, trust-agent card.
- **Lines 1100-1160** — два слайда «Roo Code / Kilo Code — режимы» и «Roo Code / Kilo Code — .roomodes»:
  - Переписать как «Kilo Code» с `code/plan/ask/debug`, `.kilocodemodes` / `.kilo/agents/`, новые значения `groups`.
  - Убрать слайд/карточку Task Manager UI.
- **Lines 1195-1253** — «Сводная таблица — субагенты»: колонка `Roo/Kilo` → `Kilo Code`, все ячейки под новые значения, а также правки CC-ячеек.
- **Lines 1307-1339** — Fan-out паттерн: убрать упоминание Roo/Kilo Boomerang.
- **Lines 1369-1417** — Specialist + Supervisor: заменить `.roomodes` → `.kilocodemodes`, убрать Orchestrator framing.
- **Lines 1494-1565** — слайды Claude Code hooks: заменить `$FILEPATH` / `$TOOL_NAME` на блок про JSON stdin + `$CLAUDE_PROJECT_DIR`.
- **Lines 1568-1607** — OpenCode plugins slide: исправить TS snippet (destructuring + `input.args`), `permission.asked` → `permission.ask`.
- **Lines 1609-1652** — Roo/Kilo emulation hooks slide: drop Roo framing, обновить под `.kilocodemodes`, убрать `.roo/rules-code/`.
- **Lines 1654-1706** — Hooks comparison table: колонка Roo/Kilo → Kilo Code.
- **Lines 1708-1736** — Strategy slide: Kilo Code карточка.
- **Lines 1816-1846** — Summary: убрать Roo.
- **Lines 1849-1874** — практические задания: Roo/Kilo → Kilo.

### Новые слайды (для новых подразделов контента)

- **§ 3.5 Ограничение доступа: tools и MCP** — 1-2 слайда: карточки CC / Kilo / OpenCode, сводка «минимальные права».
- **§ 3.6 Выбор модели для субагента** — 1-2 слайда: таблица моделей + плюсы/минусы + рекомендации по ролям.
- **§ 5.4.1 oh-my-opencode** — 1 слайд: что это, как подключить, примеры.

## Вне скоупа этого спека

- **Переписывание «ИИшного» тона § 5** (todo item 4) — отдельный пасс после этого.
- **Правки в других лекциях** (`content/part1_foundations.md`, лекция 5, и т.д.) — не трогаем.
- **Фактчек § 3.1 / § 3.2 / § 4 (секции, не покрытые v2)** — верифицируется только то, что затрагивается изменениями из этого спека.
- **Оригинальные фактчек issues из baseline 2026-03-31, закрытые в v2** — уже исправлены, не повторяем.

## Допущения

- `kilo.ai/docs` и `opencode.ai/docs` доступны для WebFetch на момент реализации (иначе соответствующие подразделы деградируют до общего описания с пометкой «уточнить позже»).
- Пользователь доступен для промежуточного ревью после фазы «content edit», до того как начнётся «slides mirror».
- Модельные имена (`claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`, `glm-4.6`, `qwen-coder-next`) — стабильны на момент правок и не изменятся во время работы.

## Критерии успеха

1. Все 15 issues из `2026-04-08-lecture4-factcheck-v2.md` имеют соответствующие правки в `content/part4_subagents_hooks.md`.
2. В content- и slides-файлах нет упоминаний Roo Code как отдельного инструмента в составе сравнения.
3. Новые § 3.5 и § 3.6 подразделы присутствуют в контенте и зеркалированы в слайдах.
4. Новый § 5.4.1 oh-my-opencode присутствует в контенте и зеркалирован.
5. Фраза про Sonnet в § 1 заменена на нейтральную formulation.
6. § 3.3 OpenCode snippet верифицирован и при необходимости обновлён.
7. Markdown-плеер и Reveal.js-презентация открываются без ошибок рендеринга.
