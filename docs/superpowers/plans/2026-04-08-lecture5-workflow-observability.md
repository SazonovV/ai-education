# Lecture 5: Workflow и наблюдаемость агентов — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create lecture 5 content (markdown conspect + Reveal.js slides) covering process discipline (spec/plan/TDD/verification) and observability (agent loop reading, traces, diff review, CI as independent verifier). Also swap lectures 5 ↔ 8 cards on the homepage.

**Architecture:** Static site, no build step. Conspect is a markdown file loaded by the SPA reader at `/reading/?part=part5_workflow_observability`. Slides are a standalone Reveal.js HTML file at `/presentations/part5/index.html`. Both follow the exact patterns of lectures 1-3 (and lecture 4, which is being authored in parallel — DO NOT touch its files).

**Tech Stack:** Markdown, HTML/CSS/JS, Reveal.js 5.2.1, Manrope + Unbounded fonts.

**Spec:** `docs/superpowers/specs/2026-04-08-lecture5-workflow-observability-design.md`

---

## Critical constraints

1. **Don't touch lecture 4.** Files under `/content/part4_*`, `/presentations/part4/`, and any `*lecture4*` plans/specs/reports are owned by another agent in a parallel branch. The "next lecture" pointer in lecture 4 will be updated separately after that work merges.
2. **Reading pages need HTTP serving.** Browser checks must be done over a local HTTP server (e.g., `python3 -m http.server 8000` from project root), not via `file://`. The reader fetches markdown via `fetch()` which fails on `file://`.
3. **Dark background for code blocks** is already part of the existing design system in `/reading/index.html` and the slides — just follow the existing patterns; don't introduce custom code-block CSS.
4. **Commit per sub-step.** Each task ends with a commit. Do not batch up multiple tasks into one commit.

---

## File Map

| Action | File | Responsibility |
|--------|------|----------------|
| Create | `/content/part5_workflow_observability.md` | Markdown conspect (~640-760 lines, 6 sections) |
| Create | `/presentations/part5/index.html` | Reveal.js slides (~28-36 slides, ~1500-1700 lines) |
| Modify | `/content/lectures.json` | Add lecture 5 entry |
| Modify | `/index.html` (lines 411-433) | Swap cards: slot 5 ← workflow/observability, slot 8 ← security |

**References while writing:**
- `/content/part4_subagents_hooks.md` — closest in structure and audience (read-only — do not edit it)
- `/presentations/part4/index.html` — closest visual template for slides (read-only — do not edit it)
- `/content/part3_tools_mcp.md` — fall-back template if part4 is mid-edit
- `/presentations/part3/index.html` — fall-back slide template

---

### Task 1: Add lecture 5 entry to lectures.json

**Files:**
- Modify: `/content/lectures.json`

- [ ] **Step 1: Insert the new entry**

After the `part4_subagents_hooks` object (currently the last array element), add a 5th element. The full file should look like:

```json
[
  {
    "id": "part1_vibecoding",
    "title": "Лекция 1 · Введение в вайбкодинг",
    "description": "Что такое vibe coding, инструменты, живой пример, практические рекомендации",
    "file": "part1_foundations.md",
    "slides": "../presentations/part1/"
  },
  {
    "id": "part2_prompt_engineering",
    "title": "Лекция 2 · Prompt Engineering, Skills & Rules",
    "description": "От чата к Agentic Engineering: system prompt, chain-of-thought, rules, skills, антипаттерны",
    "file": "part2_prompt_engineering.md",
    "slides": "../presentations/part2/"
  },
  {
    "id": "part3_tools_mcp",
    "title": "Лекция 3 · Tools и MCP",
    "description": "Кастомные инструменты, tool calling, MCP-протокол, подключение внешних сервисов, критерии выбора",
    "file": "part3_tools_mcp.md",
    "slides": "../presentations/part3/"
  },
  {
    "id": "part4_subagents_hooks",
    "title": "Лекция 4 · Субагенты, оркестрация и Hooks",
    "description": "Делегирование субагентам, паттерны оркестрации, hooks и автоматизация в Claude Code, Roo Code, Kilo Code, OpenCode",
    "file": "part4_subagents_hooks.md",
    "slides": "../presentations/part4/"
  },
  {
    "id": "part5_workflow_observability",
    "title": "Лекция 5 · Workflow и наблюдаемость агентов",
    "description": "Дисциплина процесса (spec/plan/TDD/verification) и анализ работы агента (трейсы, ревью, CI как независимый верификатор)",
    "file": "part5_workflow_observability.md",
    "slides": "../presentations/part5/"
  }
]
```

- [ ] **Step 2: Verify JSON parses**

Run: `python3 -m json.tool content/lectures.json > /dev/null`
Expected: no output, exit 0. (Any syntax error here will be loud.)

- [ ] **Step 3: Verify reading page picks up the new id**

Serve the project locally over HTTP and open `http://localhost:8000/reading/?part=part5_workflow_observability`. Expected: page loads with header chrome but reports a missing markdown file (because content file doesn't exist yet). This confirms the new id is wired up.

- [ ] **Step 4: Commit**

```bash
git add content/lectures.json
git commit -m "feat: add lecture 5 entry to lectures.json"
```

---

### Task 2: Conspect — header, ToC, Section 1 (Зачем нужен процесс и наблюдение)

**Files:**
- Create: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Create the file with header, ToC, and Section 1**

Create `/content/part5_workflow_observability.md` with the following structure (write it out — do not paste placeholders):

```markdown
# Лекция 5 · Workflow и наблюдаемость агентов

> Контекст: пятая лекция курса AI Engineering.
> Аудитория: разработчики и техлиды, знакомые с агентами, prompts, tools и субагентами (лекции 1-4).
> Фокус: дисциплина процесса (spec/plan/TDD/verification) и анализ работы агента (трейсы, ревью, CI как независимый верификатор).

---

## Содержание
1. [Зачем нужен процесс и наблюдение](#1-зачем-нужен-процесс-и-наблюдение)
2. [Столп 1 · Дисциплина процесса](#2-столп-1--дисциплина-процесса)
3. [Столп 2 · Наблюдаемость работы агента](#3-столп-2--наблюдаемость-работы-агента)
4. [Поддержка в инструментах](#4-поддержка-в-инструментах)
5. [Подводные камни](#5-подводные-камни)
6. [Итоги и практика](#6-итоги-и-практика)

---

## 1. Зачем нужен процесс и наблюдение

В лекциях 1-4 мы собрали инструментарий: prompts, rules, skills, tools, MCP, субагенты, hooks. Этого достаточно, чтобы агент *мог* делать работу. Но достаточно ли, чтобы агент делал её *правильно*? Эта лекция — про то, как превратить набор техник в надёжный процесс.

### 1.1 Знакомая история

Дал агенту задачу. Он подумал, поправил несколько файлов, написал «всё готово, тесты проходят». Закоммитил. Через день — открыл код и увидел: тесты, оказывается, никто не запускал; один из файлов агент даже не открыл; в другом — лишний `try/except`, который маскирует баг.

Это не лень модели и не «галлюцинации». Это симптом отсутствующего процесса и отсутствующего наблюдения. Лечится не «лучшим промптом», а структурой работы.

### 1.2 Две ловушки доверия

**Ложная продуктивность.** Агент пишет много, но не то, что нужно. Симптом: после каждой итерации — «ещё чуть-чуть, и готово», но конца не видно. Корень: задача не определена в виде проверяемых критериев — агент сам додумывает scope и каждый раз додумывает иначе.

**Ложное «готово».** Агент уверенно объявляет успех без доказательств. Симптом: ответ «всё работает», но нет ни запуска тестов, ни вывода линта, ни сборки. Корень: LLM оптимизирована быть *уверенной*, а не *верной*. Заявление «готово» для модели — это естественный завершающий ход, а не результат проверки.

### 1.3 Карта лекции

Противоядие — два столпа:

- **Столп 1 · Дисциплина процесса.** Каждая задача проходит через этапы, и на каждом этапе появляется *читаемый артефакт*: спека, план, тесты, итог. Эти артефакты — чекпоинты, в которых ты можешь остановиться, прочитать и одобрить (или развернуть).
- **Столп 2 · Наблюдаемость.** Ты должен *видеть*, что реально сделал агент: какие tools вызывал, в каком порядке, с каким результатом. И критически читать диф. Финальный фильтр — CI/CD, который не умеет верить агенту на слово.

Дисциплина создаёт правила. Наблюдаемость проверяет, что правила выполнены. Без одного второе бесполезно.

---
```

Section 1 should land at ~80-100 lines. Refer to the spec (`docs/superpowers/specs/2026-04-08-lecture5-workflow-observability-design.md`, section 1) for the exact narrative.

- [ ] **Step 2: Render check in browser**

Serve the project over HTTP and open `http://localhost:8000/reading/?part=part5_workflow_observability`. Expected:
- H1 title renders
- Blockquote metadata renders with proper styling
- ToC list renders, anchors point at not-yet-existing sections (clicks will fail — expected at this stage)
- Section 1 with subsections 1.1, 1.2, 1.3 renders cleanly
- No raw markdown leaks

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: add lecture 5 conspect — header, ToC, section 1 (problem statement)"
```

---

### Task 3: Conspect — Section 2 part 1 (Дисциплина: brainstorm/spec/plan, subsections 2.1-2.3)

**Files:**
- Modify: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Append section 2 opening + subsections 2.1, 2.2, 2.3**

Append after the existing `---` separator:

```markdown
## 2. Столп 1 · Дисциплина процесса

Цель столпа — превратить работу с агентом в *проверяемые шаги*. Каждый шаг производит артефакт, который можно прочитать и одобрить *до* того, как агент потратит контекст и токены на следующий шаг. Это не бюрократия — это контракт между тобой и агентом.

### 2.1 Цикл одной задачи

Brainstorm → Spec → Plan → Implement → Review. Не пять отдельных команд, а пять чекпоинтов внутри одной задачи. На каждом чекпоинте у тебя есть выбор: продолжить или развернуть.

| Шаг | Артефакт | Что проверяем |
|---|---|---|
| Brainstorm | набор уточнений и ответов | Понял ли агент намерение |
| Spec | `specs/<date>-<topic>.md` | Согласны ли с тем, что делаем |
| Plan | `plans/<date>-<topic>.md` | Согласны ли с тем, как делаем |
| Implement | код + тесты + коммиты | Соответствует ли плану |
| Review | отчёт/диф | Что сломалось, что пропущено |

Ключевой момент: артефакты *должны существовать на диске*. Не «в чате», не «в голове агента». Файл, который пережил перезапуск сессии — единственный надёжный носитель решения.

### 2.2 Spec-first: чтобы агент не «дорисовал»

Спека — это контракт. Двусмысленность в спеке = риск «дорисовки»: агент честно пытается угадать, что ты имел в виду, и часто угадывает не то.

**Чеклист спеки:**

- **Цель** — одно предложение, что мы решаем
- **Критерии приёмки** — проверяемые утверждения, не общие слова
- **Out-of-scope** — что *не* делаем (явный список)
- **Ограничения** — производительность, совместимость, зависимости

**Анти-паттерн:** «добавь валидацию форм».
Что услышит агент: «обвешать всё, что похоже на форму, чем-то похожим на валидацию». Результат — 50 проверок, половина не нужна, половина не работает.

**Лучше:** «Валидация поля email и пароля на форме `/login`. Критерии: email — RFC 5322 базовая проверка; пароль — минимум 8 символов, минимум одна цифра. Ошибки показываются под полем красным текстом. Submit-кнопка disabled пока есть ошибки. Не трогаем форму регистрации — для неё отдельная задача.»

### 2.3 Plan-first: разбиение перед кодом

План — это последовательность маленьких *проверяемых* шагов. Каждый шаг должен быть таким, чтобы про него можно было сказать «сделано» или «не сделано», без долгих разбирательств.

**Зачем разбивать:**

- Если что-то сломалось — откатываешь *один* шаг, а не всю задачу
- Можно остановиться в любой момент с осмысленным промежуточным состоянием
- Видно прогресс — не «80% готово месяц подряд»

**Пример плана** на простую задачу «добавить кэширование результатов API»:

1. Написать тест: повторный вызов с тем же аргументом не должен идти в сеть (mock проверяет 1 вызов вместо 2)
2. Запустить тест → красный
3. Добавить in-memory словарь в модуль `api_client`
4. Обернуть `fetch_user(id)` в lookup по словарю
5. Запустить тест → зелёный
6. Добавить тест на TTL: после `time.sleep(ttl+1)` повторный вызов снова идёт в сеть
7. Реализовать TTL через timestamp в значении словаря
8. Запустить оба теста → зелёные
9. Коммит

Девять маленьких шагов вместо одного «сделай кэширование». Каждый шаг занимает минуты и оставляет проверяемое состояние.

```

This part lands at ~120-140 lines. Stay close to the spec wording.

- [ ] **Step 2: Render check**

Reload `http://localhost:8000/reading/?part=part5_workflow_observability`. Expected:
- Section 2 header and 2.1-2.3 render
- The artifact table renders with all 5 rows
- Code block with the example plan renders
- Anchors `#21-цикл-одной-задачи` etc. work (click ToC item 2)

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: add lecture 5 section 2.1-2.3 (cycle, spec-first, plan-first)"
```

---

### Task 4: Conspect — Section 2 part 2 (Дисциплина: TDD, verification, mini-case, subsections 2.4-2.6)

**Files:**
- Modify: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Append subsections 2.4, 2.5, 2.6**

Append:

```markdown
### 2.4 TDD с агентом

Red-green-refactor: агент сначала пишет тест, тест *падает на красной фазе* в реальном выводе, *затем* пишет код, тест зелёный.

Почему это критично именно с агентом: тест — единственный внешний критерий истины, которому модель не может «подчиниться словами». Текст «всё работает» легко генерируется. Зелёный pytest-output, в котором видно имя теста и `PASSED`, — нет.

**Анти-паттерн — фальшивый TDD.** Агент пишет тест и реализацию одновременно. Тест с первого запуска зелёный. Это значит, тест подогнан под уже задуманный код, а не наоборот. Симптом: test-функции, которые проверяют «что код вернул то, что код вернул», без явной бизнес-логики.

**Правило:** *красная фаза должна быть в выводе*. Если её нет — TDD фальшивый.

### 2.5 Verification before completion — «evidence before assertions»

Прежде чем сказать «готово», агент должен *показать* доказательства: вывод тестов, линта, билда. Текстом, из реального вывода tool'а, не «в общих чертах».

**Правило:** *нет вывода — нет готово*.

Это правило можно сделать машинным:

- **Claude Code:** `Stop` или `PostToolUse` hook, который проверяет наличие свежего зелёного build-log'а; если нет — блокирует завершение.
- **OpenCode:** плагин на событие `session.idle`, который требует артефакт перед закрытием.
- **Roo / Kilo:** правило режима + custom MCP tool `verify_completion`, который агент обязан вызывать.

В деталях про hooks/правила см. лекцию 4. Здесь важна сама идея: дисциплина не должна держаться только на доброй воле модели — она поддерживается инфраструктурой инструмента.

### 2.6 Mini-case: до и после дисциплины

**Без дисциплины.** Задача: «отрефактори обработчик загрузки файлов, чтобы он не падал на больших файлах». Агент за одну итерацию переписал функцию, добавил streaming, сказал «готово, тесты проходят». Через день — выяснилось: тестов на эту функцию вообще не было, агент имел в виду «другие тесты в проекте всё ещё проходят». Большие файлы продолжают падать, потому что streaming реализован неправильно.

**С дисциплиной.** Та же задача:
1. **Brainstorm:** «Что значит "большие"? — Файлы > 100MB. Какой текущий лимит — 50MB, OOM. Окружение — Node, формат — multipart.»
2. **Spec:** Цель — обработчик не должен падать на файлах до 1GB. Критерии: загрузка файла на 500MB завершается за < 60s без OOM; меньшие файлы продолжают работать. Out-of-scope: ограничение по диску, CDN.
3. **Plan:** 6 шагов. Сначала — failing test с моком, проверяющим, что данные обрабатываются стримом, а не загружаются целиком в память.
4. **Implement:** TDD-цикл по плану. Каждый шаг — отдельный коммит.
5. **Review:** диф, запуск всех тестов, проверка симптома на реальном файле 500MB.

В обоих случаях агент потратил примерно одинаковое количество токенов. Но во втором случае результат можно нести в прод, а в первом — нет.

---
```

This adds ~100-130 lines. Total section 2 ≈ 240-270 lines.

- [ ] **Step 2: Render check**

Reload the page. Expected:
- Subsections 2.4, 2.5, 2.6 render
- Numbered list in the mini-case renders
- Bold/italic emphases land correctly
- Section 2 anchor in ToC navigates to its top

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: add lecture 5 section 2.4-2.6 (TDD, verification, mini-case)"
```

---

### Task 5: Conspect — Section 3 part 1 (Наблюдаемость: loop, traces, tools, subsections 3.1-3.3)

**Files:**
- Modify: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Append section 3 opening + 3.1, 3.2, 3.3**

Append:

```markdown
## 3. Столп 2 · Наблюдаемость работы агента

Дисциплина задаёт правила. Наблюдаемость отвечает на вопрос «а правила-то выполнены?». Без второго первое — церемония, которая не защищает.

### 3.1 Что такое agent loop и зачем его читать

Напоминание из лекции 1: цикл агента — `think → tool call → observe → think → …`, повторяется до условия завершения. В нормальной работе loop спрятан за UI: ты видишь сообщения и итог. Когда что-то идёт не так — нужно уметь раскатать loop как лог и прочитать его глазами.

**Три уровня видимости:**

- **Live** — то, что видишь по ходу работы: текущее сообщение агента, ongoing tool calls, их вывод
- **Session history** — последняя сессия целиком: можно открыть после того, как агент закончил, и пройтись по шагам
- **Длинная история** — предыдущие сессии, пересечения между ними, эволюция решений по проекту

Каждый уровень нужен в своём сценарии: live — для остановки в реальном времени, session history — для разбора «что вот сейчас произошло», длинная история — для аудита и повторяющихся проблем.

### 3.2 Что искать в трейсе (чеклист чтения)

- **Tool calls.** Правильные ли инструменты, в правильном ли порядке, с правильными ли аргументами? Часто видно: агент вызвал `Read` с offset, который пропустил нужный фрагмент, а потом действовал на основе неполной картины.
- **Повторения.** Одно и то же действие дважды — почти всегда симптом. Либо зацикливание (агент чинит А → ломает Б → чинит Б → ломает А), либо потерянный контекст (агент забыл, что уже это делал).
- **Молчаливые срезы углов.** Ожидал чтение всего файла — а агент прочитал первые 50 строк. Ожидал запуск всех тестов — а запущен один по имени. Эти срезы редко видны в финальном сообщении, только в трейсе.
- **Признаки путаницы.** Агент говорит про файл `X`, а правит `Y`. Ссылается на функцию, которой нет. Имя класса меняется от шага к шагу.
- **Token-spikes.** Резкий рост потребления токенов в одной секции — обычно агент зашёл в болото и его контекст забит мусором.

### 3.3 Инструменты наблюдаемости в четвёрке

Что есть из коробки и что приходится достраивать:

**Claude Code** — самый богатый стек:
- `/statusline` и кастомные statusline-скрипты — индикатор состояния поверх UI
- `settings.json` → `hooks` (`PreToolUse` / `PostToolUse`) для логирования каждого вызова инструмента в jsonl-файл
- Session history лежит в `~/.claude/projects/<project>/` — это plain jsonl, читается грепом
- `output-styles` для кастомного отображения сообщений

**Roo Code / Kilo Code:**
- История режимов и подзадач — VS Code panel (в Kilo есть Task Manager UI с таймлайном)
- Логи через VS Code output panel
- Нативных hook-based трассировок нет — обходной путь: MCP tool, который пишет аргументы и результаты в собственный лог-файл

**OpenCode:**
- Плагины с событиями `tool.execute.before` / `tool.execute.after` → пишем свой логгер на TS
- Session state доступен программно через SDK
- Гибче Claude Code в плане кастомизации, но порог входа выше

**Внешние observability-платформы.** Если нужен полноценный дашборд по проекту/команде — Langfuse, LangSmith. Они садятся поверх API и собирают трейсы централизованно. В рамках лекции — упоминание; deep-dive — отдельная тема.

Главный практический вывод: даже если инструмент даёт мало из коробки, *завести лог-файл tool calls* можно почти всегда. Это первый шаг — без него остальные практики наблюдения слепые.

```

This adds ~140-160 lines.

- [ ] **Step 2: Render check**

Reload. Expected:
- Section 3 header and 3.1-3.3 render
- Bullet hierarchies in 3.3 render correctly (nested items per tool)
- ToC link 3 navigates to section top

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: add lecture 5 section 3.1-3.3 (loop, trace checklist, tooling)"
```

---

### Task 6: Conspect — Section 3 part 2 (Наблюдаемость: diff review, CI, mini-case, subsections 3.4-3.6)

**Files:**
- Modify: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Append subsections 3.4, 3.5, 3.6**

Append:

```markdown
### 3.4 Критическое ревью диффа

Ревью кода, написанного агентом, ≠ обычное ревью. Свои риски, которых у человеческого PR обычно нет.

**Чеклист ревьюера агентского кода:**

- **Лишнее.** Агент любит дорисовывать: лишние аргументы «на будущее», fallback'и для случаев, которых не бывает, docstring'и на каждой однострочной функции, `try/except: pass` «на всякий случай». Каждое такое добавление — кандидат на удаление.
- **Хардкод.** Магические числа, строки вроде `"example.com"` или `"test-user"`, которые выглядят «для примера», но остались в коммите. Особенно опасны в коде, а не в тестах.
- **Мёртвый код.** Неиспользуемые импорты, закомментированные блоки «на случай отката», переменные, которые присваиваются, но не читаются.
- **Ложное «готово».** Изменения сделаны в моках/тестах, а не в основном коде. Тесты помечены как `@skip` / `xfail` / `it.skip`. `assert True` вместо настоящей проверки. «Тесты обновлены под новое поведение» — без объяснения, *почему* поведение изменилось.
- **Туманные коммит-сообщения.** «fix stuff», «improvements», «update». Это сигнал, что агент сам не понимает, что он сделал — а значит, и ты не поймёшь через неделю.
- **Регрессии.** Агент «починил» одно и сломал соседнее. Защита: запусти полный тестран *сам*, не доверяй фразе «все тесты проходят».

Полезно держать этот чеклист как `~/.claude/skills/review-agent-diff.md` (или эквивалент в твоём инструменте) и применять перед каждым merge.

### 3.5 CI/CD как независимый верификатор

Агент и CI — два независимых наблюдателя. Агент может убедить себя (и тебя), что всё работает. CI запускает реальные тесты в чистом окружении и не умеет врать.

**Принцип:** не доверяй success-сообщениям агента — доверяй зелёному pipeline'у на отдельной машине. Локально агент может «забыть» запустить часть тестов, использовать кешированный билд, не пересобрать зависимости. CI этого не умеет.

**Что должно быть в pipeline:**

- **Полный test run** — не выборочный. Тест-фреймворк проекта прогоняется целиком.
- **Lint и type-check** — отдельной стадией. Если упадёт — видно *что именно* (lint-rule X в файле Y), а не «что-то сломалось».
- **Build / type-check без эмита.** Даже если основная цель проекта не билдится в артефакт, проверка типа `tsc --noEmit`, `mypy`, `cargo check` ловит то, что агент пропустил локально.
- **Diff-coverage.** Упал ли coverage на *новых* строках? Агент любит писать код без тестов и говорить «тесты есть» — diff-coverage делает это видимым.

**Защита от агентских анти-паттернов:**

- **Запретить `--no-verify` и подобные обходы.** Server-side hook или branch protection. Агент любит обходить упавшие хуки именно этим флагом — в момент, когда сдаётся.
- **Skipped тесты — отдельный шаг.** Pipeline падает, если в diff появились новые `skip` / `xfail` / `it.skip`. Без этой защиты «зелёный CI» легко получить, выключив тест.
- **Маркеры незавершённости** (`TODO`, `FIXME`, `XXX`) в diff'е — линтер по вкусу команды.

**Branch protection как контракт.** Merge в основную ветку только при зелёном CI. Даже у тебя как у автора: pull request → CI → merge. Никаких прямых push'ей. Правило: *если CI упал — это факт, а не «флаки». Сначала разбираемся, потом ругаем CI.*

**Anti-pattern.** Агент работает в worktree, локально всё зелёное, прямой push в `master`. Через час кто-то другой клонирует репозиторий — и ничего не работает: забытая зависимость, кэш, переменная окружения. CI поймал бы это сразу.

**Связка со столпом 1.** Verification-before-completion работает локально (правило «нет вывода — нет готово»). CI работает глобально. Локально — для скорости итерации, CI — для финальной гарантии. Они *не заменяют* друг друга: локально ты ловишь ошибки за секунды, CI ловит то, что ты не подумал проверить.

### 3.6 Mini-case: чтение реального трейса

Выдержка из session log'а (упрощённая):

```
[14:02:11] tool: Read("src/auth/jwt.py", limit=50)
[14:02:11] tool result: <первые 50 строк>
[14:02:14] message: "Я вижу функцию verify_token. Изменю её, чтобы..."
[14:02:18] tool: Edit("src/auth/jwt.py", old="def verify_token...", new="def verify_token...")
[14:02:18] tool result: edit failed — old_string not unique
[14:02:21] tool: Edit("src/auth/jwt.py", old="def verify_token", new="def verify_token") (с другим контекстом)
[14:02:21] tool result: ok
[14:02:25] message: "Готово, функция исправлена"
```

Что не так:

- **Шаг 1.** `Read` с `limit=50` — а в файле 200 строк. Агент мог не увидеть второго определения `verify_token` ниже.
- **Шаг 4.** Первый Edit упал именно по этой причине — `old_string not unique`. Это сигнал «в файле есть похожий блок, я его не учёл».
- **Шаг 5.** Вместо того чтобы прочитать файл целиком и понять структуру, агент *подобрал* контекст для обхода not-unique. Он мог попасть не в ту функцию.
- **Шаг 6.** «Готово» без проверки. Где запуск тестов? Где verification?

Точка, после которой нужно было прерывать: шаг 4. `Edit failed — old_string not unique` — это всегда повод остановиться и переоткрыть файл целиком, а не подбирать новый контекст.

---
```

This adds ~140-160 lines. Total section 3 ≈ 280-320 lines.

- [ ] **Step 2: Render check**

Reload. Expected:
- 3.4, 3.5, 3.6 render cleanly
- The pseudo-trace code block in 3.6 renders inside `<pre>` with monospace
- All bullet lists keep correct nesting

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: add lecture 5 section 3.4-3.6 (diff review, CI, trace mini-case)"
```

---

### Task 7: Conspect — Section 4 (Поддержка в инструментах)

**Files:**
- Modify: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Append section 4**

Append:

```markdown
## 4. Поддержка в инструментах

Одни и те же практики реализованы в каждом инструменте по-своему. Где-то — первый класс, где-то — workaround. Подробные конфиги были в лекциях 3-4; здесь — *как именно инструменты помогают (или мешают) практикам из секций 2-3*.

### 4.1 Сводная таблица

| Практика | Claude Code | Roo / Kilo | OpenCode |
|---|---|---|---|
| Plan mode (отделение плана от impl) | Нативный `EnterPlanMode` / `ExitPlanMode` | Architect mode | Через системный промпт агента |
| Skills / методология как код | `~/.claude/skills/`, `Skill` tool | Custom modes (`.roomodes`) | Agents в `opencode.json` |
| Чекпоинты / approval gates | Plan approval, `AskUserQuestion` | Подтверждение шагов в Orchestrator | Approval prompts через плагин |
| Verification-before-completion | `Stop` / `PostToolUse` hooks | Эмуляция через rules + custom tool | Плагины с событиями |
| Statusline / live observability | `/statusline`, кастомные скрипты | VS Code output panel | Plugin events |
| Session history / трейсы | `~/.claude/projects/` (jsonl) | Task Manager (Kilo), VS Code logs | Программно через session API |
| Логирование tool calls | Hook → файл | Custom MCP tool-логгер | Plugin event handler |
| Output styles | `output-styles` | Нет нативного | Через плагины |

### 4.2 Что выбрать под практику

- **«Хочу строгий процесс с гарантиями»** → Claude Code. Нативные hooks + plan mode + skills дают самый прямой путь к verification-before-completion.
- **«Хочу гибкий tooling, готов писать TS»** → OpenCode плагины. Чуть выше порог входа, зато всё, что не закрыли из коробки, можно дописать самим.
- **«Хочу визуализацию задач, не хочу шелл-конфиги»** → Kilo Code. Task Manager UI помогает видеть подзадачи и их статус глазами.
- **«Уже в Roo Code»** → подходит, но для строгости нужны workaround'ы (правила режима + custom tools). Закладывайся на то, что часть гарантий придётся компенсировать инфраструктурой команды.

### 4.3 Что приходится достраивать

- **Унифицированной observability для всех 4 инструментов нет.** Если хочешь смотреть трейсы агентов из разных IDE в одном дашборде — это либо ручные hook-логгеры с общим форматом, либо внешний слой типа Langfuse / LangSmith поверх API.
- **Командная аналитика** (сколько задач, сколько токенов, типичные ошибки) — отдельная задача, выходящая за рамки IDE. На уровне одного разработчика хватает локального лог-файла; на уровне команды нужен централизованный сбор.

---
```

This adds ~70-90 lines.

- [ ] **Step 2: Render check**

Reload. Expected: section 4 renders, the comparison table looks readable, no alignment artifacts.

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: add lecture 5 section 4 (tool support comparison)"
```

---

### Task 8: Conspect — Section 5 (Подводные камни)

**Files:**
- Modify: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Append section 5**

Append:

```markdown
## 5. Подводные камни

Дисциплина и наблюдаемость снимают много рисков, но не все. Шесть типичных ловушек — каждая в формате «проблема → симптом → решение».

### 5.1 Театр процесса (cargo cult)

- **Проблема:** агент пишет спеку и план, ты их одобряешь, но они написаны «под код, который агент уже мысленно представил». Дисциплина превращается в ритуал, а не контракт.
- **Симптом:** план-документ есть, но реализация отличается от плана; спека написана общими словами вроде «улучшить производительность» без проверяемых критериев.
- **Решение:** требовать конкретику в спеке — критерии приёмки в виде проверяемых утверждений, а не «сделать систему лучше». Если спека помещается в один абзац без цифр — это не спека.

### 5.2 Фальшивый TDD

- **Проблема:** агент пишет тест и реализацию одновременно, тест подгоняется под уже задуманный код.
- **Симптом:** тесты всегда зелёные с первого запуска; тест проверяет «что функция вернула то, что функция вернула», без связи с бизнес-логикой.
- **Решение:** требовать «красную фазу» в явном виде — показать упавший тест в реальном выводе, *затем* писать код. В правилах режима / hook'ах можно даже жёстко проверять наличие красного запуска перед commit'ом теста.

### 5.3 Ложное «готово» без evidence

- **Проблема:** агент пишет «всё работает», не запустив проверки.
- **Симптом:** в ответе нет вывода тестов / линта / билда — только текстовые утверждения.
- **Решение:** правило «нет вывода — нет готово». Verification hooks локально (`Stop` / `PostToolUse`), CI как финальный фильтр глобально.

### 5.4 Слепое доверие трейсу

- **Проблема:** трейс показывает, что агент «вызвал тесты», — но ты не проверил, *какие именно* и с каким результатом.
- **Симптом:** в логе видно `tool: run_tests`, в финальном сообщении — «tests passed». На CI — красно.
- **Решение:** читать не только *что* вызвано, но и *вывод* tool'а. CI как независимый верификатор отлавливает то, что трейс пропустил.

### 5.5 Паралич планирования

- **Проблема:** обратная крайность — на каждое чихание полная цепочка spec → plan → TDD → review. Накладные расходы больше пользы.
- **Симптом:** для задачи «переименовать переменную» агент создаёт три документа и ждёт твоего одобрения после каждого.
- **Решение:** правило sizing'а — spec/plan только для нетривиальных задач (>1 файл, >1 шаг, неочевидный подход). Тривиальное — делать сразу. Сторого следить, чтобы дисциплина не превращалась в самоцель.

### 5.6 Молчаливое забывание контекста между сессиями

- **Проблема:** новая сессия не знает, что было в предыдущей; агент переоткрывает уже решённые вопросы.
- **Симптом:** агент во второй сессии переписывает то, что уже одобрено в первой. «А почему мы не используем X?» — а потому что в прошлой сессии решили не использовать.
- **Решение:** хэндофф-артефакты на диске. План и спека лежат в репо. Коммиты с осмысленными сообщениями. `AGENTS.md` / `CLAUDE.md` как «инструкция для будущих сессий»: что было сделано, какие решения приняты, чего не делать.

---
```

This adds ~80-100 lines.

- [ ] **Step 2: Render check**

Reload. Expected: 6 pitfall blocks render with consistent problem/symptom/solution structure.

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: add lecture 5 section 5 (pitfalls)"
```

---

### Task 9: Conspect — Section 6 (Итоги и практика)

**Files:**
- Modify: `/content/part5_workflow_observability.md`

- [ ] **Step 1: Append section 6**

Append:

```markdown
## 6. Итоги и практика

### 6.1 Ключевые выводы

1. **Без процесса агент производит правдоподобный мусор.** Дисциплина превращает скорость в результат.
2. **Артефакты — чекпоинты доверия.** Spec → Plan → Implement → Review: каждый шаг производит читаемый файл, который можно одобрить или развернуть.
3. **TDD с агентом обязателен.** Тест — единственный верификатор, который не подкупается словами. Красная фаза должна быть в выводе.
4. **«Готово» без evidence — не готово.** Verification-before-completion локально, CI как независимый верификатор глобально. Они не заменяют друг друга.
5. **Читать трейс важно так же, как читать диф.** Tool calls, повторения, молчаливые срезы углов, token-spikes — всё это сигналы.
6. **Ревью агентского кода ≠ обычное ревью.** Свои анти-паттерны: лишнее, хардкод, фальшивые тесты, туманные коммиты.

### 6.2 Практическое задание

**Задание 1 — Дисциплина**

Возьми небольшую фичу в своём проекте (1-2 файла, не больше). С твоим инструментом:

- Прогони brainstorm → spec → plan → implement → review.
- Сохрани артефакты (`spec.md`, `plan.md`) в репо рядом с кодом.
- В реализации покажи минимум один полный TDD-цикл: красный тест → код → зелёный тест.

**Задание 2 — Наблюдаемость**

Для своего инструмента:

- **Claude Code:** настрой `PostToolUse` hook, который дописывает каждое срабатывание tool'а в `~/.claude/agent-trace.jsonl` (поля: время, имя инструмента, успех/ошибка). Бонус: добавь в statusline индикатор последней ошибки.
- **OpenCode:** напиши плагин, логирующий `tool.execute.before` / `tool.execute.after` в файл проекта.
- **Roo / Kilo:** создай custom MCP tool `log_action` и правило в `.roomodes`, обязывающее режим вызывать его перед каждым `Edit`.

После настройки — поработай день. Открой свой лог и найди два случая, где агент свернул не туда. Опиши, что увидел в трейсе и в какой момент его стоило остановить.

### 6.3 Следующая лекция

**Лекция 6 · Context Management** — контекстное окно, что съедает токены, summarization, pruning, compact-режимы. Берём то, что узнали про дисциплину и наблюдаемость, и учимся видеть, *сколько* контекста стоит каждое действие.
```

This adds ~70-90 lines. Total file should land around ~640-760 lines.

- [ ] **Step 2: Final conspect render check**

Reload. Full pass:
- Section 6 renders
- All 6 sections present in ToC and reachable via clicking
- All numbered subsections render their internal headers
- Code blocks have dark background
- Tables in sections 2.1, 4.1 render readably
- No raw markdown leaks anywhere
- File length (`wc -l content/part5_workflow_observability.md`) is between 600 and 800 lines

- [ ] **Step 3: Commit**

```bash
git add content/part5_workflow_observability.md
git commit -m "feat: complete lecture 5 conspect — summary and practice assignments"
```

---

### Task 10: Presentation — skeleton + sections 1-2 slides

**Files:**
- Create: `/presentations/part5/index.html`

- [ ] **Step 1: Create the file with full skeleton + opening slides**

Use `/presentations/part3/index.html` as the structural template (lecture 4's slides are off-limits per the constraint at the top of this plan). Copy the full HTML/CSS structure, then:

1. Update `<title>` to `Workflow и наблюдаемость · Лекция 5`
2. Keep the entire `<style>` block — same design system
3. Keep the Yandex.Metrika script (same counter id `107242055`)
4. Keep the Reveal.js init script at the bottom of body
5. Update the `back-home` and `open-notes` anchor links:
   - `<a class="back-home" href="../../">← На главную</a>` (unchanged)
   - `<a class="open-notes" href="../../reading/?part=part5_workflow_observability">📖 Материалы</a>`
6. Replace ALL slide `<section>` elements with the new content described below.

**Title slide (`chapter-shell`):**

```html
<section>
  <div class="chapter-shell">
    <span class="kicker">AI Engineering Course</span>
    <h1 class="hero-title">Workflow и наблюдаемость</h1>
    <p class="hero-sub">Лекция 5</p>
    <div class="meta-row">
      <span class="pill">Фокус: дисциплина и анализ работы агента</span>
    </div>
  </div>
</section>
```

**Содержание slide:** card grid with 6 entries:

```html
<section>
  <div class="slide-shell">
    <h2 class="section-title">Содержание</h2>
    <div class="cards c3">
      <div class="card teal fragment"><h3>1. Зачем процесс и наблюдение</h3><p>Ловушки доверия</p></div>
      <div class="card amber fragment"><h3>2. Столп 1 · Дисциплина</h3><p>Spec, plan, TDD, verification</p></div>
      <div class="card rose fragment"><h3>3. Столп 2 · Наблюдаемость</h3><p>Loop, трейсы, ревью, CI</p></div>
      <div class="card green fragment"><h3>4. Поддержка в инструментах</h3><p>Claude Code, Roo, Kilo, OpenCode</p></div>
      <div class="card amber fragment"><h3>5. Подводные камни</h3><p>6 типичных ловушек</p></div>
      <div class="card teal fragment"><h3>6. Итоги и практика</h3><p>Чеклист и задания</p></div>
    </div>
  </div>
</section>
```

**Section 1 slides (≈3-4 slides):**
- Chapter divider: «Зачем процесс и наблюдение»
- Slide «Знакомая история» — short narrative + visual contrast
- Slide «Две ловушки доверия» — two cards (Ложная продуктивность / Ложное «готово»)
- Slide «Карта лекции» — two pillars introduced

**Section 2 slides (≈6-7 slides):**
- Chapter divider: «Столп 1 · Дисциплина процесса»
- Slide «Цикл одной задачи» — table from conspect 2.1
- Slide «Spec-first» — checklist + before/after example
- Slide «Plan-first» — example plan steps
- Slide «TDD с агентом» — red-green-refactor + анти-паттерн
- Slide «Verification before completion» — правило «нет вывода — нет готово» + tool examples
- Slide «Mini-case: до и после» — split layout

Target: ~13-15 slides for skeleton + sections 1-2.

**For each slide use the existing components from the design system:**
- `slide-shell` for content slides
- `chapter-shell` for chapter dividers
- `cards`/`card teal|amber|rose|green` for card grids
- `grid-2` for two-column layouts
- `table-wrap` + `table.mini` for tables
- `fragment` class for incremental reveals where appropriate

- [ ] **Step 2: Verify presentation opens**

Serve the project over HTTP and open `http://localhost:8000/presentations/part5/`. Check:
- Title slide renders with the gradient background
- Слайды navigation works (arrow keys)
- Slide counter at the bottom-right shows the correct total
- Code blocks (if any in section 2) render with monokai highlighting

- [ ] **Step 3: Commit**

```bash
git add presentations/part5/index.html
git commit -m "feat: add lecture 5 slides — skeleton + sections 1-2"
```

---

### Task 11: Presentation — Section 3 slides (Наблюдаемость)

**Files:**
- Modify: `/presentations/part5/index.html`

- [ ] **Step 1: Append section 3 slides before the closing `</div></div></body>` block**

Insert these slides after the section 2 slides:

**Section 3 slides (≈7-8 slides):**

1. Chapter divider: «Столп 2 · Наблюдаемость»
2. Slide «Что такое agent loop» — schematic of `think → tool → observe → think`, three visibility levels (live, session history, long history)
3. Slide «Что искать в трейсе» — checklist of 5 items as a card grid or bullet list
4. Slide «Инструменты наблюдаемости» — 4-quadrant or table layout for Claude Code, Roo/Kilo, OpenCode, External
5. Slide «Критическое ревью диффа» — checklist of 6 items (лишнее, хардкод, мёртвый код, ложное «готово», туманные коммиты, регрессии)
6. Slide «CI/CD как независимый верификатор» — principle + что в pipeline (4 items)
7. Slide «CI: защита от анти-паттернов» — `--no-verify`, skipped tests, branch protection
8. Slide «Mini-case: чтение трейса» — pseudo-trace in a `<pre>` block + annotated callouts

For the pseudo-trace in slide 8, use the `<pre><code>` styling from part3 and add visual highlights for the problematic steps (e.g., a styled `<span class="warn">` with red/amber tint, or fragment-by-fragment reveal).

- [ ] **Step 2: Verify section 3 slides**

Reload `/presentations/part5/`. Expected:
- 7-8 new slides between section 2 and the (still missing) section 4
- Code block in mini-case slide is readable, monospace, dark background
- Slide counter increases by ~7-8

- [ ] **Step 3: Commit**

```bash
git add presentations/part5/index.html
git commit -m "feat: add lecture 5 slides — section 3 (observability)"
```

---

### Task 12: Presentation — Sections 4-5 slides (Tool support + Pitfalls)

**Files:**
- Modify: `/presentations/part5/index.html`

- [ ] **Step 1: Append section 4 slides**

**Section 4 slides (≈3 slides):**

1. Chapter divider: «Поддержка в инструментах»
2. Slide «Сводная таблица» — full comparison table from conspect 4.1 (8 rows × 4 columns). Use `table-wrap` + `table.mini`. May need a smaller font size than default — use the `.compact` modifier if it exists in the design system, otherwise inline `style="font-size: 0.5em;"` on this table.
3. Slide «Что выбрать под практику» — 4 bullet items (one per tool/scenario)

- [ ] **Step 2: Append section 5 slides**

**Section 5 slides (≈4 slides):**

1. Chapter divider: «Подводные камни»
2. Slide «Театр процесса + Фальшивый TDD» — two-card layout, each with problem/symptom/solution
3. Slide «Ложное "готово" + Слепое доверие трейсу» — two-card layout
4. Slide «Паралич планирования + Забывание контекста» — two-card layout

Use the same `cards c2` or `grid-2` pattern that lecture 4 uses for similar content, falling back to `grid-2` from part3 if needed.

- [ ] **Step 3: Verify slides**

Reload. Check:
- Section 4 table fits within the slide (no horizontal overflow)
- Section 5 cards render evenly
- Slide count is now ~24-26

- [ ] **Step 4: Commit**

```bash
git add presentations/part5/index.html
git commit -m "feat: add lecture 5 slides — sections 4-5 (tools, pitfalls)"
```

---

### Task 13: Presentation — Section 6 + closing slides

**Files:**
- Modify: `/presentations/part5/index.html`

- [ ] **Step 1: Append section 6 slides**

**Section 6 slides (≈3 slides):**

1. Chapter divider: «Итоги и практика»
2. Slide «Ключевые выводы» — numbered list of 6 takeaways
3. Slide «Практическое задание» — Задание 1 (Дисциплина) + Задание 2 (Наблюдаемость), each with tool-specific bullets

- [ ] **Step 2: Append closing slides**

**Closing slides (2 slides) — reuse the structure from `/presentations/part3/index.html` closing:**

1. **«Дальше → Лекция 6»** slide:
   - `chapter-shell` style
   - Heading: «Дальше · Лекция 6»
   - Subtitle: «Context Management — что съедает токены, summarization, compact-режимы»
   - Pill row with navigation links to lectures 1, 2, 3 + «Главная»
   - **Do NOT add a pill for lecture 4** — leave that linkage for whoever finishes lecture 4 in the parallel branch
   - **Do NOT add a self-link to lecture 5**

2. **«Остаёмся на связи»** slide:
   - Copy the exact promo layout from `/presentations/part3/index.html`'s last slide
   - Author photo on the left (`assets/avatar.jpg`)
   - QR code + Telegram link on the right
   - Same wording

- [ ] **Step 3: Final presentation review**

Reload `/presentations/part5/`. Full check:
- Total slide count: ~28-36
- Navigation works end-to-end (arrow keys, Esc overview)
- All code blocks have syntax highlighting
- All chapter dividers visible and consistent in style
- Last two slides match part3's closing pattern
- No console errors in browser devtools
- No overflow or layout issues at standard resolutions

- [ ] **Step 4: Commit**

```bash
git add presentations/part5/index.html
git commit -m "feat: complete lecture 5 slides — summary, practice, closing"
```

---

### Task 14: Swap homepage cards (lecture 5 ↔ lecture 8)

**Files:**
- Modify: `/index.html` (specifically lines 411-433 — the first and fourth cards in the «Планируются» grid)

- [ ] **Step 1: Replace card content for the slot 5 position with the new lecture 5**

Find the planned card currently labelled «Лекция 5 · Безопасность агентов» (around lines 411-415). Replace its content. Since the new lecture 5 is no longer planned but in-work, also move it from `.planned` to a clickable card. Replace lines 411-415 with:

```html
      <article class="card clickable" data-href="./presentations/part5/">
        <span class="complexity mid">Средняя сложность</span>
        <h3>Лекция 5 · Workflow и наблюдаемость агентов</h3>
        <p>Дисциплина процесса (spec/plan/TDD/verification) и анализ работы агента: трейсы, ревью, CI как независимый верификатор.</p>
        <div class="row">
          <a class="btn" href="./presentations/part5/">Презентация</a>
          <a class="btn secondary" href="./reading/?part=part5_workflow_observability">Материалы лекции</a>
        </div>
      </article>
```

**Important:** the card is now in the «Планируются» grid but contains links — this is wrong. Move the entire card *out* of the «Планируются» `<section class="grid">` and into the «В работе» grid (the section that currently contains lecture 4's card, around line 397). The lecture 5 card should be the second article inside that section, after lecture 4.

After this move, the «Планируются» grid no longer has a lecture 5 entry.

- [ ] **Step 2: Replace lecture 8 card content with the security topic**

Find the planned card for «Лекция 8 · Debugging и наблюдаемость» (around lines 429-433). Replace its content with the security topic that previously occupied slot 5:

```html
      <article class="card planned">
        <span class="complexity mid">Средняя сложность</span>
        <h3>Лекция 8 · Безопасность агентов</h3>
        <p>Prompt injection, MCP-уязвимости, утечки секретов, sandboxing, план реагирования на инцидент.</p>
      </article>
```

- [ ] **Step 3: Verify the homepage in the browser**

Open `http://localhost:8000/` and check:
- «В работе» section now has two cards: lecture 4 and lecture 5 (workflow & observability), both clickable
- «Планируются» section starts with lecture 6 (Context Management) — no orphan lecture 5 card
- Lecture 8 card now reads «Безопасность агентов»
- Lectures 6, 7, 9 are unchanged
- All "Презентация" / "Материалы лекции" links on the new lecture 5 card resolve (the slides and reading page should now exist from previous tasks)
- Footer/legend unchanged

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: swap planned lectures 5 and 8, activate lecture 5 card"
```

---

### Task 15: Final cross-check

**Files:**
- (read-only verification across the project)

- [ ] **Step 1: Verify all navigation paths**

Serve the project and check in the browser:

1. `http://localhost:8000/` → click lecture 5 «Презентация» → slides load
2. `http://localhost:8000/` → click lecture 5 «Материалы лекции» → reading page loads with full conspect
3. `http://localhost:8000/reading/` → lecture 5 entry appears in the index card list
4. `http://localhost:8000/reading/?part=part5_workflow_observability` → all 6 sections render, ToC anchors work
5. `http://localhost:8000/presentations/part5/` → slides work end-to-end, last slide links to lectures 1, 2, 3 and home (not 4, not self)

- [ ] **Step 2: Content consistency check**

Skim the conspect and slides side-by-side:
- Both cover the same 6 sections
- Both mention the four tools (Claude Code, Roo, Kilo, OpenCode) consistently
- Section 3 in both has the CI/CD subsection (it was a late addition to the spec, easy to forget)
- Numbers in section 6 takeaways match between conspect (numbered list) and slides

- [ ] **Step 3: Lecture 4 untouched check**

Run:

```bash
git status
git diff HEAD~15..HEAD --stat -- 'content/part4*' 'presentations/part4/' 'docs/superpowers/specs/*lecture4*' 'docs/superpowers/plans/*lecture4*' 'docs/superpowers/reports/*lecture4*'
```

Expected: no changes to any lecture-4 file across this plan's commits. If anything shows up — STOP and revert it; lecture 4 is owned by another agent.

- [ ] **Step 4: Spec coverage spot-check**

Open the spec (`docs/superpowers/specs/2026-04-08-lecture5-workflow-observability-design.md`) and verify each section has corresponding content in the conspect and slides:
- Spec section 1 → conspect §1, slides «Зачем процесс и наблюдение» chapter
- Spec section 2 (subsections 2.1-2.6) → conspect §2.1-2.6, slides «Дисциплина» chapter (verify cycle table, spec checklist, plan example, TDD, verification, mini-case all appear)
- Spec section 3 (subsections 3.1-3.6) → conspect §3.1-3.6, slides «Наблюдаемость» chapter (loop, checklist, tools, diff review, **CI**, mini-case)
- Spec section 4 → conspect §4, slides «Поддержка в инструментах»
- Spec section 5 (6 pitfalls) → conspect §5.1-5.6, slides §5
- Spec section 6 → conspect §6, slides «Итоги»

- [ ] **Step 5: Commit if any fixes were needed**

If any small fixes were needed during cross-check, commit them per the same incremental-commit rule:

```bash
git add <files>
git commit -m "fix: lecture 5 cross-check corrections"
```

If nothing needed fixing, no commit is required for this task.

---

## Out of scope (do NOT do as part of this plan)

- Editing any file under `/content/part4_*`, `/presentations/part4/`, or any `*lecture4*` doc
- Updating the «Следующая лекция» link in lecture 4's conspect or closing slide (will be handled after the parallel branch merges)
- Changing lectures 6, 7, 9 cards on the homepage
- Adding RAG, security, context-management, or multi-agent content beyond what's in the spec
- Creating CI/CD pipelines, hooks, or skill files for the project itself (those are *examples* in the lecture, not infrastructure to ship)
- Refactoring the design system (`<style>` blocks) of the slides or reading page

---

## Notes for the executor

- **HTTP serving:** if no server is already running, `python3 -m http.server 8000` from the repo root works. Verify URLs use `http://localhost:8000/...`, not `file://`.
- **Code block dark background:** the reading page already styles `<pre><code>` blocks with a dark background and Highlight.js theme — no extra CSS needed. If a code block looks wrong, the issue is in the markdown (e.g., missing language hint), not in the design system.
- **Incremental commits:** each task ends with a commit. Don't batch tasks. The user has explicit feedback on this point in their persistent memory.
- **Russian content:** all user-facing content (conspect text, slide text, card descriptions) is in Russian. Code identifiers, file names, and config keys remain English.
- **Style references:** when in doubt about formatting/layout, look at lecture 3 (`/content/part3_tools_mcp.md`, `/presentations/part3/index.html`) — it is the closest *editable* template (lecture 4 is off-limits).
