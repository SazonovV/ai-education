# Лекция 3 · Tools и MCP

> Контекст: третья лекция курса AI Engineering.
> Аудитория: разработчики, уже знакомые с агентами и промпт-инжинирингом (лекции 1-2).

---

## Содержание
1. [Два слоя расширения агента](#1-два-слоя-расширения-агента)
2. [Tools](#2-tools)
3. [MCP — Model Context Protocol](#3-mcp--model-context-protocol)
4. [Tool vs MCP: когда что выбрать](#4-tool-vs-mcp-когда-что-выбрать)
5. [CLI vs MCP: когда лучше CLI утилита](#5-cli-vs-mcp-когда-лучше-cli-утилита)
6. [Итоги и практика](#6-итоги-и-практика)

---

## 1. Два слоя расширения агента

### 1.1 Зачем агенту расширения

Без tools агент — это чат-бот. Он генерирует текст, но не может:
- Прочитать файл
- Запустить команду
- Обратиться к API
- Изменить реальный проект

**Tools превращают генератор текста в агента**, способного действовать в реальном мире.

### 1.2 Два способа дать агенту инструменты

```
                        ┌───────────────────────────┐
                        │       Агент (LLM)          │
                        │  Видит единый список tools  │
                        └─────────┬─────────────────┘
                                  │ tool_use
                        ┌─────────▼─────────────────┐
                        │         Хост               │
                        │  (IDE, CLI, ваш сервер)    │
                        └────┬──────────────┬───────┘
              Встроенные /   │              │   \ MCP (JSON-RPC)
             кастомные tools │              │
                    ┌────────▼──┐    ┌──────▼────────┐
                    │ Локально  │    │ MCP Server(ы) │
                    │ Read,Edit │    │ GitHub,Sentry │
                    │ Bash,Grep │    │ Playwright,БД │
                    └───────────┘    └───────┬───────┘
                                             │
                                    ┌────────▼───────┐
                                    │  Внешний мир   │
                                    │  API, сервисы  │
                                    └────────────────┘
```

Ключевой момент: **модель не различает** встроенный tool и MCP-tool. Она видит единый список с name, description, input_schema. Разница — только в том, как хост выполняет вызов:

**Встроенные / кастомные tools** — хост выполняет их сам, локально в своём процессе. Определены хостом (IDE, CLI) или вами через API.

**MCP tools** — хост пересылает вызов внешнему MCP-серверу через JSON-RPC (stdio или HTTP). Сервер выполняет действие и возвращает результат. С точки зрения модели — ничего не изменилось.

### 1.3 Как агент вызывает tool

1. Пользователь даёт задачу
2. LLM анализирует задачу и решает, какой tool нужен
3. LLM генерирует JSON с именем tool и параметрами (structured output)
4. Хост выполняет tool и возвращает результат
5. LLM получает результат и решает: продолжить или завершить

```json
{
  "type": "tool_use",
  "name": "Read",
  "input": {
    "file_path": "/src/app.ts",
    "limit": 50
  }
}
```

> Важно: агент **не выполняет** tool — он отправляет **запрос**. Хост (IDE, CLI) выполняет и возвращает результат.

---

## 2. Tools

### 2.1 Встроенные tools

Набор зависит от хоста (Claude Code, VS Code, Cursor), но типичные:

**Файловая система:**
- `Read` — прочитать файл (с поддержкой offset/limit)
- `Edit` — точечная правка (замена строк)
- `Write` — создать или перезаписать файл
- `Glob` — найти файлы по паттерну
- `Grep` — поиск по содержимому (регулярные выражения)

**Исполнение:**
- `Bash` — выполнение терминальных команд
- `Agent` — запуск субагентов
- `NotebookEdit` — работа с Jupyter-ячейками

**Навигация и поиск:**
- `WebSearch` — поиск в интернете
- `WebFetch` — загрузка веб-страниц
- `LSP` — навигация по коду (go to definition, find references)

### 2.2 Встроенные vs кастомные tools

Встроенные tools (Read, Edit, Bash) уже определены хостом — IDE или CLI. Вам не нужно их описывать.

Но когда вы строите **своё приложение через Claude API**, вы определяете tools сами: передаёте их в параметре `tools` каждого API-запроса. Модель видит ваши кастомные tools точно так же, как встроенные — через name, description и input_schema.

Дальше в этом разделе — как создать и подключить кастомный tool.

### 2.3 Как LLM выбирает tool

Модель видит три элемента каждого tool:

| Поле | Назначение |
|------|-----------|
| `name` | Идентификатор. Для кастомных tools — snake_case |
| `description` | **Главный фактор выбора**. Когда и зачем использовать |
| `input_schema` | JSON Schema входных параметров |

Tools передаются в каждом API-запросе как часть payload. LLM читает описания **всех** доступных tools, сопоставляет с текущей задачей и генерирует structured output: имя tool + параметры.

> Description — самая важная часть. Если описание плохое — tool не будет вызван, даже если он идеально подходит для задачи.

> Важно: каждый tool definition занимает токены контекста. 20 tools с подробными описаниями — это ~2000-4000 токенов. При 50+ tools нужна стратегия: dynamic tool selection, фильтрация по задаче, группировка.

### 2.4 Анатомия tool definition

```json
{
  "name": "get_weather",
  "description": "Get current weather for a city. Use when user asks about weather or needs temperature data.",
  "input_schema": {
    "type": "object",
    "properties": {
      "city": {
        "type": "string",
        "description": "City name, e.g. Moscow"
      },
      "units": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature units"
      }
    },
    "required": ["city"]
  }
}
```

Ключевые моменты:
- **name** — короткое, понятное. Для кастомных tools рекомендуется snake_case (встроенные tools хоста могут использовать PascalCase: Read, Edit, Bash)
- **description** — когда вызывать, что делает, что НЕ делает
- **input_schema** — JSON Schema. `description` у каждого поля помогает модели заполнить правильно
- **required** — обязательные параметры

### 2.5 Как написать хороший description

**Плохо:**
```json
{
  "name": "db_query",
  "description": "Queries the database"
}
```
Проблемы: не ясно когда использовать, какую БД, какие запросы допустимы.

**Хорошо:**
```json
{
  "name": "db_query",
  "description": "Execute a read-only SQL query against the PostgreSQL analytics database. Use when user asks about metrics, user counts, or revenue data. Returns max 100 rows. Do NOT use for mutations."
}
```
Почему работает: ясно read-only, PostgreSQL, analytics. Ясно когда использовать: metrics, user counts, revenue. Ясны ограничения: max 100, no mutations.

### 2.6 Tool calling в Claude API

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=[{
        "name": "get_weather",
        "description": "Get current weather for a city",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {"type": "string"}
            },
            "required": ["city"]
        }
    }],
    messages=[{
        "role": "user",
        "content": "Какая погода в Москве?"
    }]
)
```

Ответ модели при tool use:
```json
{
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01A...",
      "name": "get_weather",
      "input": {"city": "Moscow"}
    }
  ],
  "stop_reason": "tool_use"
}
```

### 2.7 Замыкаем цикл: tool_result

После получения `tool_use` от модели, ваш код выполняет действие и отправляет результат обратно:

```python
import json

# 1. Получили tool_use от модели (из response выше)
tool_use_block = response.content[0]  # type: "tool_use"

# 2. Выполняем действие
weather = get_weather_from_api(tool_use_block.input["city"])

# 3. Отправляем tool_result обратно модели
final_response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    messages=[
        {"role": "user", "content": "Какая погода в Москве?"},
        {"role": "assistant", "content": response.content},
        {
            "role": "user",
            "content": [{
                "type": "tool_result",
                "tool_use_id": tool_use_block.id,
                "content": json.dumps(weather)
            }]
        }
    ]
)
# final_response → «В Москве −5°C, идёт снег»
```

**Если tool вернул ошибку**, используйте флаг `is_error`:
```python
{
    "type": "tool_result",
    "tool_use_id": tool_use_block.id,
    "content": "City not found: Moskva",
    "is_error": True
}
```
Модель увидит ошибку и скорректирует подход — например, уточнит название города.

### 2.8 Полный цикл tool use

```
User: «Погода в Москве?»
    ↓
LLM: tool_use → get_weather({city: "Moscow"})
    ↓
Ваш код: вызывает Weather API → {temp: -5, condition: "snow"}
    ↓
tool_result → отправляется обратно модели
    ↓
LLM: «В Москве −5°C, идёт снег»
```

> Модель **никогда** не вызывает API напрямую. Она **просит** ваш код сделать это и получает результат.

### 2.9 tool_choice: управление вызовом

Параметр `tool_choice` контролирует, должна ли модель вызвать tool:

| Значение | Поведение |
|----------|----------|
| `{"type": "auto"}` | Модель сама решает (по умолчанию) |
| `{"type": "any"}` | Модель обязана вызвать какой-либо tool |
| `{"type": "tool", "name": "get_weather"}` | Модель обязана вызвать конкретный tool |

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    tools=tools,
    tool_choice={"type": "any"},  # заставить вызвать tool
    messages=[...]
)
```

Полезно для workflow, где tool call обязателен (например, structured extraction).

### 2.10 Паттерны работы с tools

**Параллельный вызов.** Модель может запросить несколько tools одновременно. Хост выполняет параллельно, результаты возвращаются пакетом. Существенно ускоряет I/O-bound операции (чтение файлов, сетевые запросы).

**Цепочка вызовов.** Результат одного tool → вход другого. Например: Read файл → Edit файл → Bash (запуск тестов). Это и есть agentic loop.

**Fallback.** Если tool вернул ошибку — модель пробует другой подход. Grep не нашёл → Glob → Read. Bash failed → анализирует stderr → исправляет команду.

### 2.11 Подводные камни (gotchas)

**Размер tool_result съедает контекст.** Если tool вернул 100KB текста (например, `Read` большого файла или `Bash` с длинным логом) — всё это попадает в контекст и уменьшает доступное окно. Используйте `limit`/`offset`, фильтруйте вывод, возвращайте только нужное.

**Каждый виток agentic loop = API-вызов = деньги.** Агент, который делает Read → Edit → Bash → Read → Edit → Bash, совершает 6+ API-вызовов. При сложных задачах — десятки. Следите за стоимостью. Используйте `cache_control` на tool definitions для кэширования (prompt caching):

```python
tools=[{
    "name": "get_weather",
    "description": "...",
    "input_schema": {...},
    "cache_control": {"type": "ephemeral"}  # кэшировать определение
}]
```

**Модель может «галлюцинировать» tool calls.** Если tools плохо описаны — модель может выдумать несуществующее имя tool, неправильно заполнить параметры или вызвать tool, когда это не нужно. Защита: хорошие description, валидация на стороне хоста, `tool_choice: "auto"` по умолчанию.

**Слишком много tools — модель теряется.** При 50+ tools модель хуже выбирает нужный. Стратегии:
- *Dynamic tool selection* — показывать модели только релевантные tools для текущей задачи
- *Группировка* — объединять мелкие tools в один с параметром `action`
- *Двухступенчатый выбор* — сначала модель выбирает категорию, потом конкретный tool

---

## 3. MCP — Model Context Protocol

### 3.1 Что такое MCP

**Model Context Protocol** — открытый стандарт от Anthropic для подключения AI-агентов к внешним сервисам.

Ключевые характеристики:
- Протокол на основе **JSON-RPC 2.0**
- Аналогия: **USB для AI** — один протокол, любое устройство
- Поддержка: Claude Code, VS Code (Copilot), Cursor, Roo Code, Zed, JetBrains...
- Экосистема: тысячи готовых серверов

### 3.2 Зачем MCP

**До MCP:** каждая интеграция — уникальный код. M агентов × N сервисов = M×N интеграций.

**С MCP:** один стандартный протокол. Написал MCP-сервер для GitHub — он работает в любом MCP-совместимом агенте.

### 3.3 Архитектура MCP

```
Host (Claude Code, VS Code)
  └── MCP Client
        ↔ MCP Server 1 (GitHub)
        ↔ MCP Server 2 (Playwright)
        ↔ MCP Server 3 (PostgreSQL)
```

**Транспорт:**
- **stdio** — сервер как дочерний процесс (stdin/stdout). Самый распространённый для локальных серверов.
- **Streamable HTTP** — основной HTTP-транспорт. Сервер отвечает обычным JSON или SSE-потоком в зависимости от запроса. Используется для удалённых серверов.
- **SSE** — Server-Sent Events по HTTP. *Deprecated* — заменён на Streamable HTTP.

**Что предоставляет сервер (capabilities):**
- **Tools** — функции, которые модель может вызвать (действия с side effects)
- **Resources** — данные для чтения (файлы, записи БД). Read-only, без side effects. Поддерживают подписки — сервер уведомляет клиента об изменениях.
- **Prompts** — шаблоны промптов от сервера
- **Sampling** — сервер может запрашивать LLM-completions через клиента (обратный вызов)

### 3.4 MCP Server на Python

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("demo-server")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers together."""
    return a + b

@mcp.tool()
def get_time() -> str:
    """Get current server time."""
    from datetime import datetime
    return datetime.now().isoformat()

@mcp.resource("config://app")
def get_config() -> str:
    """Application configuration."""
    return "debug=true, version=1.0"

if __name__ == "__main__":
    mcp.run()
```

- `FastMCP` — SDK для Python (`pip install mcp`)
- `@mcp.tool()` — регистрация tool из обычной функции
- Docstring автоматически становится description
- Type hints → JSON Schema автоматически
- `@mcp.resource()` — данные для чтения

### 3.5 Подключение MCP Server к агенту

**Claude Code** — файл `.mcp.json` в корне проекта:

```json
{
  "mcpServers": {
    "demo": {
      "command": "python",
      "args": ["server.py"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

**VS Code / Cursor** — файл `.vscode/mcp.json`:

```json
{
  "servers": {
    "demo": {
      "type": "stdio",
      "command": "python",
      "args": ["server.py"]
    }
  }
}
```

После подключения tools сервера появляются в списке доступных для модели. Пользователь может не знать, что tool — внешний MCP.

### 3.6 Популярные MCP-серверы

| Сервер | Что делает | Пример |
|--------|-----------|--------|
| **GitHub** | Issues, PRs, repos, code search | «Создай PR с этими изменениями» |
| **Playwright** | Управление браузером | «Открой страницу и проверь кнопку» |
| **Sentry** | Ошибки, performance | «Покажи топ ошибок за сегодня» |
| **PostgreSQL** | SQL-запросы к БД | «Сколько пользователей вчера?» |
| **Slack** | Сообщения, каналы | «Отправь результат в #releases» |
| **Filesystem** | Безопасный доступ к файлам | Sandbox для файловых операций |
| **Context7** | Документация библиотек | «Как использовать React Query v5?» |

Каталоги: [mcp.so](https://mcp.so), [smithery.ai](https://smithery.ai), [glama.ai](https://glama.ai) — тысячи серверов.

### 3.7 Безопасность MCP

**Риски:**
- **Tool poisoning** — вредоносный сервер внедряет инструкции в description
- **Prompt injection через данные** — данные от сервера содержат инструкции для модели
- **Чрезмерные разрешения** — сервер с доступом к БД может удалить данные
- **Утечка секретов** — токены в env переменных сервера

**Защита:**
- Явное разрешение на каждый tool call (permissions)
- Логирование всех вызовов (аудит)
- Sandbox — сервер работает в изоляции
- Проверка серверов — используйте только из надёжных источников
- Принцип минимальных привилегий — read-only токены по умолчанию

> MCP-сервер — это расширение агента. Доверяйте ему как npm-пакету: проверяйте перед установкой.

---

## 4. Tool vs MCP: когда что выбрать

### 4.1 Сравнение

| Критерий | Кастомный Tool (API) | MCP Server |
|----------|---------------------|------------|
| Где живёт | Внутри вашего кода | Отдельный процесс |
| Переиспользование | Только ваше приложение | Любой MCP-агент |
| Протокол | Ваш собственный | Стандартный JSON-RPC |
| Язык | Любой (обработчик отделён от API) | Любой |
| Изоляция | В процессе приложения | Отдельный процесс, sandbox |
| Latency | Минимальная (in-process) | Overhead JSON-RPC через stdio/HTTP |
| Сложность | Низкая (одна функция) | Средняя (сервер + конфиг) |
| Экосистема | Нет | Тысячи серверов |

### 4.2 Когда хватит Tool

- Логика специфична для вашего приложения
- Нужен доступ к внутреннему состоянию (in-memory данные)
- Одноразовый инструмент, не планируете переиспользовать
- Простой API: одна функция, несколько параметров
- Не хотите усложнять инфраструктуру

*Пример: валидация бизнес-логики, расчёт скидки, внутренний поиск*

### 4.3 Когда нужен MCP

- Интеграция с внешним сервисом (GitHub, Jira, Slack)
- Хотите переиспользовать в разных агентах/IDE
- Нужна изоляция (sandbox, отдельные permissions)
- Сложная интеграция с множеством tools
- Есть готовый MCP-сервер в экосистеме

*Пример: работа с GitHub, управление браузером, мониторинг*

### 4.4 Путь развития

```
Функция → Tool → MCP Server → Публикация (npm/pip)
```

> **Правило:** начните с tool. Если его хочется переиспользовать или изолировать — оберните в MCP. Не прыгайте сразу к MCP (YAGNI).

---

## 5. CLI vs MCP: когда лучше CLI утилита

### 5.1 Проблема: не всё нужно оборачивать в MCP

У многих инструментов уже есть отличные CLI-утилиты: `git`, `gh`, `docker`, `kubectl`, `npm`, `psql`, `curl`. Агент может вызывать их через `Bash` tool напрямую. Зачем тогда MCP-сервер?

**Ответ:** часто незачем. MCP-сервер — это дополнительный слой абстракции, который не всегда добавляет ценность.

### 5.2 Когда CLI лучше MCP

**1. CLI уже покрывает 100% нужных операций**

```bash
# Агент и так может делать всё это через Bash:
git log --oneline -10
gh pr create --title "Fix bug" --body "..."
docker ps
kubectl get pods -n production
```

Если агент вызывает `Bash("gh pr list")` — результат тот же, что от MCP GitHub-сервера. Зачем прослойка?

**2. CLI даёт больше гибкости**

MCP-сервер предоставляет фиксированный набор tools. CLI-утилита — это полный набор команд с pipe, фильтрами, комбинированием.

```bash
# Такое MCP-серверу GitHub не по силам:
gh pr list --json number,title,author | jq '.[] | select(.author.login=="sazonov")'

# Или комбинирование нескольких CLI:
git diff HEAD~3 --stat | grep ".ts" | wc -l
```

**3. Нет overhead на запуск и поддержку сервера**

CLI-утилита — это одна команда. MCP-сервер — это:
- Процесс, который нужно запустить и поддерживать
- Конфигурация в `.mcp.json`
- Зависимости (npm/pip пакет)
- Обновления и совместимость

**4. Лучше документация и отладка**

`git --help`, `man curl`, Stack Overflow — миллионы примеров. MCP-серверы часто с минимальной документацией.

**5. Агент уже умеет работать с CLI**

LLM отлично знает `git`, `curl`, `docker` из обучающих данных. С MCP-tool ему нужно разбираться по description.

### 5.3 Когда MCP всё-таки лучше CLI

| Ситуация | Почему MCP лучше |
|----------|-----------------|
| **Нужна авторизация с OAuth/токенами** | MCP-сервер управляет сессией, CLI требует настройки на каждой машине |
| **Сложный API без CLI** | У сервиса нет CLI (Sentry, Linear, Notion) — MCP заменяет curl к REST API |
| **Безопасность и ограничение операций** | MCP предоставляет фиксированный набор tools с JSON Schema. Bash — полный доступ к shell, любая команда |
| **Structured output** | MCP возвращает структурированный JSON, который модели проще парсить. CLI даёт текст, в котором модель может ошибиться |
| **Работа в песочнице без shell** | Если агент работает без доступа к Bash (веб-агенты, ограниченные среды) |
| **Нужна типизация и валидация** | MCP описывает параметры через JSON Schema, Bash — свободная строка |
| **Несколько IDE/агентов** | Конфиг один раз → работает везде; CLI-команды нужно адаптировать |

### 5.4 Практическое правило

```
Есть хорошая CLI-утилита?
  ├── Да → Агент хорошо с ней справляется?
  │         ├── Да → Используй CLI через Bash ✅
  │         └── Нет (сложный вывод, auth) → Рассмотри MCP
  └── Нет → Есть REST API?
              ├── Да → MCP-сервер оправдан ✅
              └── Нет → Напиши свой tool / MCP
```

> **Правило:** Если `Bash("gh pr list")` даёт тот же результат, что MCP GitHub-сервер — выбирай CLI. Не добавляй абстракцию ради абстракции.

### 5.5 Пример: GitHub — CLI vs MCP

**CLI (через Bash):**
```bash
# Список PR — одна команда
gh pr list --limit 5

# Создание PR — одна команда
gh pr create --title "Fix auth" --body "Fixed token refresh"

# Сложные запросы с jq
gh api repos/myorg/myrepo/actions/runs --jq '.workflow_runs[:3] | .[].conclusion'
```

**MCP (GitHub-сервер):**
- Требует: установку, конфигурацию, токен в `.mcp.json`
- Даёт: tools вроде `list_pull_requests`, `create_issue`
- Плюс: не нужен `gh` CLI на машине, structured JSON-ответы
- Минус: ограниченный набор операций, нет pipe/комбинирования

**Вывод:** если `gh` уже установлен и настроен — CLI гибче. MCP-сервер полезен, когда нет CLI, нужна работа из среды без shell, или важна безопасность (MCP ограничивает набор операций, а Bash — полный доступ к shell).

---

## 6. Итоги и практика

### 6.1 Ключевые выводы

1. **Tools = действия.** Без tools агент — чат-бот. Tools дают возможность действовать в реальном мире.
2. **Description решает всё.** Модель выбирает tool по описанию. Плохое описание = tool не вызывается.
3. **MCP = USB для AI.** Один протокол подключает любой внешний сервис к любому агенту.
4. **Tool → MCP — эволюция.** Начинайте с простого tool. Если нужно переиспользование — оборачивайте в MCP.
5. **CLI — не забывай.** Если у инструмента есть хорошая CLI-утилита — MCP-сервер может быть лишним. Bash + `gh`/`git`/`docker` часто проще и мощнее.
6. **Безопасность — приоритет.** MCP-сервер — расширение агента. Проверяйте серверы, ограничивайте разрешения.

### 6.2 Практическое задание

**Задание 1: Свой Tool (check_endpoint)**

Создайте tool, который проверяет статус HTTP-эндпоинта.

```python
# 1. Tool definition
tool = {
    "name": "check_endpoint",
    "description": "Check HTTP endpoint status. Use when user asks to verify if a service is running or to test an API endpoint. Returns status code, response time, and key headers.",
    "input_schema": {
        "type": "object",
        "properties": {
            "url": {"type": "string", "description": "Full URL to check, e.g. https://api.example.com/health"},
            "method": {"type": "string", "enum": ["GET", "POST", "HEAD"], "description": "HTTP method (default: GET)"}
        },
        "required": ["url"]
    }
}

# 2. Обработчик
import httpx, time

def handle_check_endpoint(url: str, method: str = "GET") -> dict:
    start = time.monotonic()
    r = httpx.request(method, url, timeout=10)
    return {
        "status": r.status_code,
        "latency_ms": round((time.monotonic() - start) * 1000),
        "content_type": r.headers.get("content-type", ""),
        "ok": r.is_success
    }

# 3. Подключите к Claude API (см. раздел 2.6-2.7)
# 4. Тест: «Проверь, работает ли https://httpbin.org/get»
```

Ожидаемый результат: модель вызывает `check_endpoint`, получает `{status: 200, latency_ms: 150, ok: true}`, формирует человекочитаемый ответ.

**Задание 2: MCP Server**

Подключите Filesystem MCP-сервер и дайте агенту задачу.

```bash
# 1. Подключение — добавьте в .mcp.json:
{
  "mcpServers": {
    "fs": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp/sandbox"]
    }
  }
}

# 2. Перезапустите Claude Code
# 3. Задача агенту: «Создай в /tmp/sandbox файл report.md с текущей датой»
# 4. Проверьте: какие tool calls агент сделал? (видно в логах)
```

Ожидаемый результат: агент использует `write_file` от MCP-сервера вместо встроенного `Write`, потому что путь ограничен `/tmp/sandbox`.

### 6.3 Следующая лекция

**Лекция 4 · Субагенты, оркестрация и Hooks** — у агента теперь есть инструменты. Как научить его делегировать часть работы другому агенту?
