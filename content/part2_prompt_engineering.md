# От чата к Agentic Engineering

> Лекция 2 · Prompt Engineering, Skills & Rules
> Фокус: как управлять поведением агента
> Аудитория: разработчики и техлиды, работающие с AI-агентами

---

## Содержание

1. Чем агентный режим отличается от чата
2. Prompt Engineering для агентов
3. Rules: глобальные и проектные правила
4. Skills: модульные способности агента
5. Антипаттерны: топ ошибок в промптах и правилах

---

## 1. Чем агентный режим отличается от чата

### 1.1. Чат: запрос → ответ

Когда вы работаете с ChatGPT, Claude.ai или встроенным чатом IDE, взаимодействие устроено просто:

1. Вы пишете запрос
2. Модель отвечает текстом
3. Вы вручную применяете ответ (копируете код, вносите изменения)

Чат — это **советник**. Он предлагает решения, но не может их выполнить. Не видит ваш проект. Не может проверить, работает ли его предложение. Каждый шаг требует участия человека.

**Ограничения чата:**

- Нет доступа к файлам, терминалу, браузеру
- Контекст — только то, что вы вставили руками
- Не может проверить, работает ли его решение
- Контекст теряется между сообщениями (в пределах окна)

### 1.2. Агент: Think → Act → Observe → Loop

Агентный режим — принципиально другая архитектура. Агент работает в цикле:

1. **THINK** — анализирует задачу, строит план
2. **ACT** — вызывает инструмент: читает файл, запускает команду, редактирует код
3. **OBSERVE** — получает результат, анализирует
4. **LOOP** — решает: продолжить (вернуться к шагу 1) или завершить

Агент — это **исполнитель**. Он думает, действует и проверяет результат сам.

### 1.3. Три ключевых отличия

| | Чат | Агент |
|---|---|---|
| **Цикл** | Один запрос → один ответ | Многоходовый loop до завершения |
| **Инструменты** | Только текст (copy-paste) | Read, Edit, Bash, Search, Browser... |
| **Автономность** | Ноль — человек выполняет всё | Настраиваемая: от ask-every-step до full auto |
| **Контекст** | То, что вставили в чат | Весь проект + результаты tool calls |
| **Обратная связь** | Не видит, сработало ли | Видит ошибки компиляции, тесты, git diff |

Примеры агентных инструментов: Roo Code, Kilo Code, OpenCode, Zed, Claude Code.

---

## 2. Prompt Engineering для агентов

### 2.1. Промпт для чата vs промпт для агента

Промпты для чата и для агента — это разные дисциплины.

**Для чата:**
- Максимум контекста в одном сообщении
- Чёткая формулировка желаемого ответа
- Few-shot примеры
- Результат: текст, который вы копируете

**Для агента:**
- **Цель**, а не пошаговая инструкция
- Ограничения: что не менять, куда не лезть
- Критерии готовности: тесты проходят, линтер чист
- Результат: реальные изменения в проекте

> Ключевое отличие: агенту не нужен контекст вашего файла в промпте — он прочитает его сам. Ему нужна **цель** и **границы**.

### 2.2. Анатомия system prompt

System prompt — это инструкция, которую агент получает до начала работы. Из чего он состоит:

- **Роль** — кто агент и для чего он создан
- **Поведение** — как принимать решения
- **Ограничения** — что запрещено
- **Формат** — как отвечать
- **Tools** — какие инструменты доступны

**Порядок имеет значение.** LLM лучше следует инструкциям в начале и конце текста (primacy + recency bias). Критичные ограничения размещайте ближе к началу, примеры и детали — в середине, финальное напоминание — в конце.

### 2.3. Chain-of-thought для агентов

Chain-of-thought (CoT) — это явное указание агенту **думать перед действием**: анализировать, планировать, обосновывать выбор.

Пример инструкции:

```markdown
# Перед каждым изменением:
1. Прочитай текущий код целиком
2. Определи, какие файлы затронуты
3. Объясни свой план в 2-3 предложениях
4. Только потом начинай редактирование
```

Зачем это нужно:
- Снижает «галлюцинации» — модель проверяет себя
- Делает действия агента предсказуемыми
- Проще дебажить — видишь ход рассуждений
- Уменьшает «прыжки к решению» без анализа

> Extended thinking в Claude, reasoning в OpenAI — это встроенный CoT на уровне модели. Но явные инструкции в system prompt усиливают эффект.

### 2.4. CoT на практике: Plan-агенты и Loop-паттерны

Принцип chain-of-thought реализован в конкретных паттернах работы агентов:

**Plan-first агенты** — сначала план, потом реализация:

- **Roo Code / Kilo Code** — режим Architect: агент проектирует решение, затем передаёт задачу в режим Code через Boomerang-делегацию
- **Claude Code** — режим `--permission-mode plan`: агент анализирует и планирует, но не выполняет write-операции без одобрения
- **OpenCode** — можно определить агента Plan в `.opencode/agents/`, который только планирует

**Итеративная проверка (Ralph Loop)** — паттерн из экосистемы Claude Code, когда агент работает в цикле действие → проверка → исправление:

- На каждой итерации запускает тесты или линтер
- Не останавливается, пока проверки не пройдут
- Настраивается интервал и условие выхода
- Похож на TDD-цикл (red → green → refactor), но автоматизированный

**Как встроить CoT в правила любого агента:**

```markdown
# В rules проекта:
Before writing code:
1. Read existing implementation
2. Write a plan as a numbered list
3. Get user approval
4. Only then implement

After each change:
- Run tests
- If tests fail → fix and rerun
- Commit only when green
```

Такие правила работают в любом инструменте: Roo Code, OpenCode, Zed, Claude Code — все они загружают rules перед началом работы.

### 2.5. Пять правил хорошего промпта

**1. Цель, не рецепт**

«Добавь валидацию email в форму регистрации» вместо «открой файл X, найди строку Y, вставь Z». Агент сам найдёт файл и нужное место.

**2. Границы**

«Не меняй API контракт», «Не трогай миграции», «Только файлы в src/auth/». Без явных границ агент может «улучшить» то, что не нужно трогать.

**3. Критерии готовности**

«Тесты проходят», «Линтер без ошибок», «Типы совпадают с API». Агент должен знать, когда задача выполнена.

**4. Контекст задачи**

«Это баг из прода: пользователи видят 500 при пустом email» — объясните зачем, а не только что. Контекст помогает агенту принимать правильные решения в неоднозначных ситуациях.

**5. Одна задача**

Не мешайте рефакторинг с новой фичей. Один промпт — один фокус. Это снижает вероятность ошибки и облегчает ревью.

### 2.6. Демо: плохой vs хороший промпт

**Плохой промпт:**

```
Почини баг с логином
```

Что пойдёт не так: агент не знает, какой баг. Может «починить» не то. Нет способа проверить результат. Может сломать соседнюю логику.

**Хороший промпт:**

```
Баг: при логине с email без @ сервер возвращает 500 вместо 422.

Файл: src/auth/login.ts, функция validateCredentials.

Добавь валидацию формата email до обращения к БД.
Тесты: npm test -- auth.
Не меняй поведение для валидного логина.
```

**Формула:** Контекст (что сломано) + Локация (где) + Действие (что сделать) + Проверка (как убедиться) + Границы (что не трогать).

### 2.7. Ещё примеры: от размытого к точному

| Размыто | Точно |
|---|---|
| Сделай код лучше | Вынеси дублирующуюся логику парсинга дат из `utils/date.ts` в одну функцию `parseISO`. Тесты должны проходить. |
| Добавь тесты | Добавь unit-тесты для `CartService.applyDiscount()`: пустая корзина, скидка > 100%, истекший купон. Используй vitest. |
| Задеплой это | Создай Dockerfile для `api/`: node:20-alpine, multi-stage build, порт 3000. Не включай devDependencies в финальный образ. |
| Перепиши на TypeScript | Мигрируй `src/utils/helpers.js` на TypeScript. Используй strict mode. Не меняй публичный API. Запусти `tsc --noEmit` для проверки. |

---

## 3. Rules: глобальные и проектные правила

### 3.1. Что такое Rules

Rules — это текстовые инструкции, загружаемые в контекст **каждого** запроса к агенту. Они определяют стиль, ограничения и стандарты поведения.

Ключевые свойства:
- Действуют **всегда**, не требуют активации
- Работают как «конституция» для агента
- Два уровня: **глобальные** (для всех проектов) и **проектные** (для конкретного репозитория)

> Проектные правила — главный способ сделать поведение агента одинаковым для всей команды.

### 3.2. Где живут правила: карта конфигов

| Инструмент | Глобальные | Проектные |
|---|---|---|
| **Roo Code** | Custom instructions в UI | `.roo/rules/`, `.roo/rules-code/`, `.roo/rules-architect/` |
| **Kilo Code** | Custom instructions в UI | `.kilocode/rules/`, `.kilocode/rules-code/` |
| **OpenCode** | `~/.config/opencode/opencode.jsonc` | `AGENTS.md`, `.opencode/opencode.jsonc` |
| **Zed** | Rules Library (Agent Panel → Rules) | Файл `.rules` в корне проекта |
| **Claude Code** | `~/.claude/CLAUDE.md` | `CLAUDE.md` в корне проекта |
| **Cursor** | Settings → Rules for AI | `.cursor/rules/*.mdc` |
| **AGENTS.md** | — | Кросс-инструментный: Claude Code, OpenCode, Codex, Roo Code, Kilo Code, Zed |

AGENTS.md — универсальная конвенция. Если команда использует разные агенты, начните с него.

### 3.3. Живой пример: .roo/rules/project.md

```markdown
# .roo/rules/project.md

## Project
E-commerce API on Node.js + TypeScript + PostgreSQL.
Monorepo: apps/api, apps/web, packages/shared.

## Code Style
- Strict TypeScript, no `any`
- Functional style: prefer pure functions, avoid classes
- Error handling: use Result<T, E> pattern, no throw in business logic
- Tests: vitest, minimum 80% coverage for new code

## Architecture
- apps/api follows Clean Architecture: domain → use-cases → infra
- Never import from infra in domain layer
- All DB queries go through repository pattern

## Forbidden
- NEVER modify .env files or commit secrets
- NEVER use `rm -rf` or `DROP TABLE`
- NEVER disable TypeScript strict mode
- NEVER add dependencies without asking first
```

Этот файл загружается во всех режимах Roo Code / Kilo Code. Для режимоспецифичных правил используйте `rules-code/`, `rules-architect/` и т.д.

### 3.4. Живой пример: AGENTS.md

```markdown
# AGENTS.md

Кросс-инструментный файл правил. Поддерживается OpenCode, Claude Code, Codex.

## Stack
- React 18, TypeScript 5, Tailwind CSS, Zustand
- Testing: Vitest + React Testing Library

## UI Rules
- All components use functional style with hooks
- Use Tailwind utility classes, no CSS modules
- Responsive: mobile-first, breakpoints sm/md/lg
- Accessibility: every interactive element needs aria-label or visible label

## File Structure
- components/: reusable UI components
- features/: domain-specific feature modules
- Each feature has: components/, hooks/, types.ts

## Git
- Conventional commits: feat:, fix:, refactor:
- One logical change per commit
```

AGENTS.md удобен, когда команда использует разные инструменты: один файл правил — единый стандарт для всех.

### 3.5. Режимные правила (Roo Code / Kilo Code)

Roo Code и Kilo Code поддерживают правила, привязанные к конкретному режиму работы:

- `rules-code/` — только в режиме Code
- `rules-architect/` — только в режиме Architect
- `rules/` — во всех режимах

Пример правила безопасности для режима Code:

```markdown
# .roo/rules-code/security.md

- NEVER read or modify `.env*` files
- NEVER run `rm -rf` or `DROP TABLE`
- Ask user before deleting any file
- Validate all user inputs at API boundary
- Use parameterized queries, never string concatenation for SQL
```

Пример правила для режима Architect:

```markdown
# .roo/rules-architect/planning.md

- Always start with a written plan
- List affected files before any changes
- Consider backward compatibility
- Estimate blast radius of changes
```

Разделение по режимам снижает расход контекста: агент загружает только релевантные правила.

### 3.6. Условные правила по путям файлов

Cursor поддерживает правила, которые активируются только при работе с определёнными файлами:

```markdown
# .cursor/rules/api-layer.mdc
---
globs: ["apps/api/**/*.ts"]
---

- Use Express middleware pattern
- All routes return typed ResponseDTO
- Log every error with requestId
- Rate limiting on all public endpoints
```

```markdown
# .cursor/rules/tests.mdc
---
globs: ["**/*.test.ts", "**/*.spec.ts"]
---

- Use describe/it structure
- One assertion per test preferred
- Mock external services, not internal
- Test behavior, not implementation
```

Зачем условные правила: разный код — разные стандарты. API-слой ≠ фронтенд ≠ тесты ≠ миграции. Активируются только при работе с matching-файлами, экономя контекст.

### 3.7. Как писать эффективные правила

**Хорошие правила:**
- **Конкретные:** «используй vitest» вместо «пиши тесты»
- **Проверяемые:** можно проверить автоматически
- **С примерами:** покажите, как выглядит правильно
- **С причиной:** почему это правило существует
- **Короткие:** одно правило — одно предложение

**Плохие правила:**
- **Размытые:** «пиши хороший код»
- **Противоречивые:** правило A конфликтует с B
- **Избыточные:** 500 строк правил → агент начинает игнорировать
- **Без контекста:** «не используй X» — а почему?
- **Устаревшие:** правило для старой версии стека

> Практика: начните с 10-15 правил. Добавляйте новые, только когда агент ошибается в конкретном паттерне.

---

## 4. Skills: модульные способности агента

### 4.1. Rules vs Skills

| | Rules | Skills |
|---|---|---|
| **Когда активны** | Всегда, в каждом запросе | Только при активации (on-demand) |
| **Роль** | Рамки: что можно / нельзя | Процедуры: как выполнить задачу |
| **Расход контекста** | Постоянный | Только при использовании |
| **Аналогия** | Конституция компании | Должностная инструкция на задачу |
| **Пример** | «Не используй any в TypeScript» | «Как написать E2E-тест для этого проекта» |

> Rules задают **рамки**, Skills дают **способности**.

### 4.2. Анатомия skill-файла

Skill — это Markdown-файл с YAML frontmatter, описывающий процедуру выполнения конкретного типа задачи:

````yaml
---
name: create-api-endpoint
description: >
  Use when creating a new REST API endpoint.
  Covers routing, validation, service layer, and tests.
---

# Creating a new API endpoint

## Steps
1. Create route in `apps/api/src/routes/`
2. Define request/response DTOs in `types/`
3. Implement service logic in `services/`
4. Add input validation with zod schema
5. Write tests: unit for service, integration for route
6. Update OpenAPI spec in `docs/api.yaml`

## Template
```typescript
// routes/[entity].ts
router.post('/[entity]',
  validate(Create[Entity]Schema),
  async (req, res) => {
    const result = await [entity]Service.create(req.body);
    res.status(201).json(result);
  }
);
```

## Checklist
- [ ] Route follows RESTful naming
- [ ] Zod schema validates all inputs
- [ ] Service has unit tests
- [ ] Error responses use standard format
````

### 4.3. Базовый механизм: как агент узнаёт о skills

Все современные агенты используют один и тот же базовый процесс:

1. **SCAN** — при старте агент сканирует директории skills (проектные и глобальные)
2. **COLLECT** — собирает `name` + `description` каждого найденного skill
3. **PROMPT** — список описаний попадает в system prompt (или в доступные tools)
4. **SELECT** — кто-то решает, какой skill активировать для текущей задачи

Первые три шага одинаковы во всех инструментах. Разница — в четвёртом: **кто и как выбирает** skill.

### 4.4. Два подхода к выбору skills

#### LLM выбирает сама (OpenCode, Claude Code, Kilo Code)

Все descriptions попадают в промпт. LLM читает список и сама решает, вызвать ли `skill` tool с конкретным именем. Никакого внешнего движка рекомендаций нет — семантика целиком **внутри модели**. Это основной подход большинства инструментов.

Аналогия: «Я показал модели всё меню — она сама выберет блюдо».

**Плюсы:** простота, не нужна доп. инфраструктура. Работает из коробки.
**Минусы:** все описания в промпте → раздувание контекста при 50+ skills. Модель может проигнорировать skill или выбрать не тот.

> Документация Kilo Code: *"There's no keyword matching or semantic search — the agent evaluates your request against all available skill descriptions and determines if one clearly and unambiguously applies."*

#### Semantic similarity через плагин (OpenCode)

OpenCode из коробки использует LLM-выбор, но плагин `opencode-agent-skills` добавляет дополнительный слой: **semantic similarity** для автоматического обнаружения релевантных skills. Когда плагин обнаруживает совпадение, он инжектирует промпт, побуждающий агента загрузить подходящий skill. Это не замена LLM-выбора, а усиление — pre-filter, который подсказывает модели.

Плагин также предоставляет tools для работы со skills: `use_skill`, `read_skill_file`, `run_skill_script`, `get_available_skills`.

#### Режимы как механизм фильтрации (Roo Code)

Skills могут быть привязаны к режимам (Code, Architect, Debug...) через директории `skills-code/`, `skills-architect/` и т.д. Переключение режима определяет, какие skills доступны для LLM-выбора. Оркестратор может делегировать подзадачу в нужный режим через Boomerang-паттерн.

**Плюсы:** естественное разделение ответственности, предсказуемость, экономия контекста.
**Минусы:** нужно заранее продумать набор режимов.

### 4.5. Сравнение по инструментам

| Инструмент | Кто выбирает skill | Где хранятся | Масштаб |
|---|---|---|---|
| **Claude Code** | LLM по description из SKILL.md | `.claude/skills/`, `.agents/skills/` | Десятки skills |
| **OpenCode** | LLM + semantic similarity (плагин `opencode-agent-skills`) | `.opencode/skills/`, `.agents/skills/` | Десятки skills |
| **Kilo Code** | LLM по description из SKILL.md | `.kilocode/skills/`, `.agents/skills/` | Десятки skills |
| **Roo Code** | LLM + фильтрация по режимам | `.roo/skills/`, `.agents/skills/` | По режимам |
| **Zed** | Пользователь — Rules Library или slash-команда | Rules Library, файл `.rules` | Любой |

> **Тренд:** LLM-выбор — стандарт. Режимы — для дополнительной фильтрации. `.agents/skills/` — кросс-инструментный путь.

### 4.6. Примеры skills из реальных проектов

**write-e2e-test** — trigger: «напиши E2E тест»
- Прочитай существующие тесты в `e2e/`
- Используй `getByRole` / `getByText`
- Каждый тест — конкретный assert
- Никогда `waitForTimeout`
- Запусти eslint + playwright после

**database-migration** — trigger: «создай миграцию»
- Генерируй через `npx prisma migrate dev`
- Проверь обратную совместимость
- Если DROP — спроси подтверждение
- Обнови seed-данные
- Запусти `prisma generate`

**code-review** — trigger: «проверь мой код»
- Проверь типы, edge cases, безопасность
- Оцени читаемость и naming
- Предложи конкретные улучшения с кодом
- Не предлагай рефакторинг ради рефакторинга

**debug-production** — trigger: «баг в проде»
- Собери контекст: логи, stack trace, URL
- Воспроизведи локально
- Найди root cause перед фиксом
- Добавь regression test

### 4.7. Где хранить skills: от инструмента к универсальной конвенции

#### Инструмент-специфичные пути

| Инструмент | Путь | Формат |
|---|---|---|
| **Roo Code** | `.roo/skills/`, `.agents/skills/` | SKILL.md с frontmatter |
| **Kilo Code** | `.kilocode/skills/`, `.agents/skills/` | SKILL.md с frontmatter |
| **OpenCode** | `.opencode/skills/`, `.agents/skills/` | SKILL.md с frontmatter |
| **Zed** | Rules Library (глобальная), файл `.rules` | Markdown |
| **Claude Code** | `.claude/skills/`, `.agents/skills/` | SKILL.md с frontmatter |
| **Cursor** | `.cursor/rules/*.mdc`, Notepads | Markdown с globs frontmatter |

Командные skills — часть проекта, коммитятся в Git. Личные skills — в домашней директории.

#### Папка `.agents/skills/` — кросс-инструментный стандарт

Проблема инструмент-специфичных путей очевидна: если в команде кто-то использует Claude Code, кто-то OpenCode, а кто-то Kilo Code — skills приходится дублировать в `.claude/skills/`, `.opencode/skills/`, `.kilocode/skills/`.

Решение описано в открытом стандарте **Agent Skills** (agentskills.io) — спецификации формата skills, которую поддерживают 30+ инструментов, включая Claude Code, OpenCode, Kilo Code, Roo Code, Cursor, GitHub Copilot, Gemini CLI и другие. Стандарт определяет путь `.agents/skills/` как кросс-инструментный:

```
.agents/
  skills/
    write-e2e-test/        ← skill-папка
      SKILL.md
    manual-qa/             ← skill-папка с примерами
      SKILL.md
      examples/
        checkout-flow.md
        login-flow.md
```

Инструменты, поддерживающие стандарт, сканируют `.agents/skills/` наряду со своими специфичными путями. Skill, положенный в `.agents/skills/`, будет виден во всех совместимых инструментах без дублирования.

> **Примечание:** стандарт Agent Skills определяет путь именно для skills (`.agents/skills/`). Для rules кросс-инструментным решением остаётся файл `AGENTS.md` в корне проекта.

#### Skill как папка: вынос примеров

Когда skill простой — достаточно одного `.md` файла. Но если примеров много или они объёмные, полезно организовать skill как **папку**:

```
.agents/skills/manual-qa/
  SKILL.md                  ← основной файл: description, шаги, чеклист
  examples/
    checkout-flow.md        ← пример: тестирование оформления заказа
    login-flow.md           ← пример: тестирование входа в систему
  templates/
    bug-report.md           ← шаблон баг-репорта
```

**Как это работает:** при старте агент сканирует skills и читает только `name` + `description` из основного файла. Примеры и шаблоны **не загружаются в контекст**, пока skill не активирован. Когда skill срабатывает, агент читает основной файл и видит инструкции вроде:

```markdown
## Примеры
Перед началом тестирования прочитай релевантный пример из `./examples/`.
```

Агент сам прочитает нужный файл — это просто чтение файла с диска, никакой специальной поддержки не нужно. Любой агентный инструмент умеет читать файлы, поэтому паттерн работает **везде**: Claude Code, OpenCode, Kilo Code, Roo Code.

**Зачем выносить примеры:**
- Экономия контекста — примеры на 500+ строк не загружаются при каждом запросе
- Примеры можно обновлять независимо от инструкций
- Разные примеры для разных сценариев — агент подгрузит только релевантный
- Шаблоны используются как есть, без риска что агент «перескажет» их по памяти

### 4.8. Публичные репозитории skills

Skills не обязательно писать с нуля. Появились публичные коллекции готовых skills, которые можно подключить к проекту.

#### skills.sh

Открытый каталог skills для агентных инструментов. Содержит ready-to-use skills для типовых задач: code review, миграции, тестирование, документация, работа с API. Издатели включают Anthropic, Vercel, Microsoft и сообщество. Skills оформлены в стандартном формате Agent Skills.

Как использовать:
1. Найти нужный skill в каталоге на сайте
2. Установить через CLI: `npx skills add <owner/repo>` — skill скачается в `.agents/skills/`
3. Адаптировать пути, стек и примеры под свой проект
4. Закоммитить в репозиторий

#### neuraldeep.ru

Русскоязычный агрегатор skills, MCP-серверов и AI-инструментов с фокусом на интеграцию с российскими сервисами: Яндекс (Wordstat, Метрика, Вебмастер), Битрикс24, 1C, GigaChat, Wildberries, YooKassa. Полезно, если вы работаете с отечественными API и платёжными системами. Установка: `npx skillsbd add <owner/repo>`.

Адрес: https://neuraldeep.ru/

#### Как оценивать чужие skills

Прежде чем подключать skill из публичного репозитория, проверьте:
- **description** — содержит ли конкретный trigger, а не размытый «helps with code»?
- **Шаги** — есть ли пронумерованный список действий?
- **Примеры** — привязаны ли к реальному стеку или абстрактные?
- **Чеклист** — есть ли критерии завершения?
- **Безопасность** — нет ли инструкций, выполняющих деструктивные команды?

Хороший публичный skill — это **отправная точка**. Его всегда нужно адаптировать: заменить пути, добавить свои примеры, настроить под стек проекта.

### 4.9. Практический пример: skill для ручного QA-тестирования

Skills — не только про код. Агент с доступом к браузеру может выполнять задачи, которые раньше делались только руками. Вот пример skill для ручного тестирования веб-приложения:

````yaml
---
name: manual-qa-test
description: >
  Use when asked to manually test a web feature, verify a user flow,
  or check UI after deployment. Opens the browser, follows the test
  scenario, takes screenshots, and files a bug report if something is wrong.
---

# Manual QA Testing

## Before you start
1. Read the test scenario from `./examples/` if one exists for this flow
2. Ask: what URL to test, which environment (staging/production), and auth credentials if needed
3. Open the browser and navigate to the target URL

## Testing process
For each step in the test scenario:
1. **Act** — perform the user action (click, fill form, navigate)
2. **Observe** — check the result:
   - Page loaded correctly? No console errors?
   - UI matches expected state?
   - Data displays correctly?
3. **Screenshot** — capture the current state
4. **Note** — record pass/fail and any anomalies

## What to check at every step
- Visual: layout not broken, text readable, no overlapping elements
- Functional: buttons work, forms submit, navigation correct
- Data: correct values displayed, no "undefined" or empty states
- Console: no JS errors, no failed network requests (4xx/5xx)
- Responsive: if specified, check at mobile/tablet breakpoints

## If a bug is found
Create a bug report using the template from `./templates/bug-report.md`:
- Title: short description of what's broken
- Steps to reproduce: exact sequence of actions
- Expected vs actual result
- Screenshot of the problem
- Console errors if any
- Environment: URL, browser, viewport size

## Completion checklist
- [ ] All steps in the scenario executed
- [ ] Screenshots taken for each key step
- [ ] Console checked for errors at each step
- [ ] Bug reports filed for any issues found
- [ ] Summary: X steps passed, Y failed, Z bugs filed
````

**Пример файла `examples/checkout-flow.md`:**

```markdown
# Тестирование оформления заказа

## Предусловия
- Пользователь авторизован
- В корзине минимум 1 товар

## Шаги
1. Открыть корзину → проверить список товаров и сумму
2. Нажать «Оформить заказ» → форма доставки отобразилась
3. Заполнить адрес доставки → валидация работает (пустые поля, некорректный индекс)
4. Выбрать способ оплаты → сумма пересчиталась с учётом доставки
5. Нажать «Подтвердить» → редирект на страницу успеха
6. Проверить: заказ появился в «Мои заказы» с корректным статусом

## Граничные случаи
- Пустая корзина → кнопка «Оформить» недоступна
- Товар закончился пока оформляли → понятное сообщение
- Двойной клик на «Подтвердить» → заказ не дублируется
```

Такой skill превращает агента в QA-тестировщика: он открывает браузер, проходит сценарий, делает скриншоты, логирует результаты и оформляет баг-репорты. Человеку остаётся ревью результатов.

> Ключевая идея: skill описывает **процесс**, а не код. Агент с инструментами (браузер, файловая система) может автоматизировать любую повторяемую процедуру — не только написание кода.

### 4.10. Как написать хороший skill: чек-лист

**Обязательно:**
- **name** — глагол + существительное: `write-e2e-test`
- **description** — когда именно активировать (trigger-фраза)
- **Шаги** — пронумерованный список действий
- **Примеры из проекта** — реальные пути, реальный код
- **Чеклист** — что проверить перед завершением

**Рекомендуется:**
- **Шаблон кода** — template с плейсхолдерами
- **Anti-patterns** — чего избегать
- **Команды для проверки** — lint, test, build
- **Ссылки на ресурсы** — docs, примеры, API

---

## 5. Антипаттерны: топ ошибок в промптах и правилах

### 5.1. Ошибка #1: «Простыня» — слишком много правил

**Сломано:** Rules файл (800 строк) с 200+ правилами, покрывающими каждый мыслимый случай.

**Что происходит:** агент начинает игнорировать правила или противоречить сам себе. Чем длиннее контекст с правилами, тем ниже вероятность точного следования каждому из них.

**Починено:** .roo/rules/project.md (40 строк) с фокусом на критичном. Детали — в отдельных файлах по назначению:

```markdown
# .roo/rules/project.md (40 строк)
## Must
- TypeScript strict, no `any`
- Vitest for tests
- Result<T,E> for error handling

## Architecture
- Clean Architecture layers
- Never import infra in domain

## Forbidden
- No .env modifications
- No rm -rf or DROP TABLE

## Style
→ see rules-code/ for Code mode
→ see rules-architect/ for planning
```

### 5.2. Ошибка #2: Промпт-рецепт вместо цели

**Сломано:**

```
1. Открой файл src/api/users.ts
2. Найди функцию getUser на строке 42
3. Добавь после строки 45: if (!user.email) return null;
4. Сохрани файл
5. Запусти npm test
6. Если тест упал, добавь mock в __mocks__/userService.ts
```

**Что происходит:** если код поменялся, все номера строк неверны. Агент не понимает зачем — не может адаптироваться к изменениям.

**Починено:**

```
В src/api/users.ts функция getUser не проверяет наличие email у
пользователя, что приводит к 500 при обращении к /api/users/:id.

Добавь early return с 422, если email отсутствует или невалиден.

Тесты: npm test -- users
Не меняй поведение для валидных пользователей.
```

Агент сам найдёт нужное место, адаптируется к текущему коду, проверит результат.

### 5.3. Ошибка #3: Правила без примеров и причин

**Сломано:**

```markdown
# Rules
- Use proper error handling
- Follow best practices
- Write clean code
- Be consistent
- Don't use bad patterns
```

**Что происходит:** агент интерпретирует каждое слово по-своему. «Proper» и «clean» — не конкретика.

**Починено:**

```markdown
# Error Handling
Use Result<T, AppError> pattern.
Why: throw breaks type safety and makes error paths invisible.

## Good
const result = await getUser(id);
if (result.isErr()) {
  return res.status(404).json({
    error: result.error.message
  });
}

## Bad
try {
  const user = await getUser(id);
} catch (e) {
  // what type is e? unknown
}
```

**Шаблон:** Правило + Почему + Пример правильно + Пример неправильно.

### 5.4. Другие частые ошибки

**Противоречивые правила.** AGENTS.md: «используй классы». .roo/rules-code/: «используй функции». Результат: агент чередует стили случайным образом. Решение: один источник правды на стиль, остальные ссылаются.

**Skills без description.** Skill с пустым или размытым description вроде «General helper for code tasks» — агент либо никогда не активирует, либо активирует не к месту. Решение: description = trigger-фраза: «Use when writing E2E tests with Playwright».

**Правила в промпте вместо файла.** Каждый раз вручную пишете «не используй any, пиши тесты, используй vitest...». Решение: вынесите в rules-файл проекта — один раз написали, работает всегда.

**Никогда не обновляют правила.** Стек мигрировал с Jest на Vitest, а в правилах по-прежнему «use Jest». Решение: ревью правил при смене зависимостей. Добавьте в чеклист обновления стека.

---

## Ключевые выводы

1. **Агент ≠ чат.** Цикл Think-Act-Observe и tool use — фундаментальная разница. Промпты для агентов — это цели и границы, не рецепты.

2. **Хороший промпт:** контекст + локация + действие + проверка + границы. Одна задача — один промпт.

3. **Rules = конституция.** Действуют всегда. Держите их короткими, конкретными и с примерами.

4. **Skills = способности.** Активируются по требованию. Экономят контекст и стандартизируют процедуры.

5. **Коммитьте в Git.** Rules и Skills — часть проекта. Вся команда работает по одним стандартам.

---

## Практическое задание

### Задание 1: Rules

Создайте rules для своего инструмента: `.roo/rules/`, `AGENTS.md` или `.rules`
- Опишите стек, стиль, архитектуру
- Добавьте 5-7 запретов (Forbidden)
- Закоммитьте в репозиторий

### Задание 2: Skill

- Определите повторяющуюся задачу в вашем проекте
- Напишите skill с шагами, шаблоном и чеклистом
- Протестируйте: дайте агенту задачу, которая должна активировать skill
- Итерируйте description, пока активация не станет стабильной
