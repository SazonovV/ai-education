# Лекция 6 · Context Engineering — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Создать конспект (reading page) и презентацию (Reveal.js slides) для лекции 6 «Context Engineering», обновить манифест, навигацию и главную страницу.

**Architecture:** Markdown конспект в `/content/part6_context_engineering.md`, Reveal.js презентация в `/presentations/part6/index.html`. Манифест `lectures.json`, главная `index.html` и навигация в закрывающих слайдах существующих лекций обновляются. В конце — фактчек утверждений и общее ревью.

**Tech Stack:** Markdown (Marked.js рендеринг), HTML/CSS/JS (Reveal.js 5.2.1), без build-системы.

**Spec:** `docs/superpowers/specs/2026-04-13-lecture6-context-engineering-design.md`

---

### Task 1: Конспект — секции 1-2 (Хук + Токенизация)

**Files:**
- Create: `content/part6_context_engineering.md`

**Контекст:** Конспект пишется на русском. Стиль и структура — как в `content/part5_workflow_observability.md`. Лингвистические правила из `AGENTS.md`: без калькирования с английского, проверка после написания. Заголовочный блок — из спеки.

- [ ] **Step 1: Создать файл конспекта с заголовком, содержанием и секциями 1-2**

Файл `content/part6_context_engineering.md`. Содержание:

1. **Заголовочный блок** — как в спеке (заголовок, контекст, аудитория, фокус)
2. **Содержание** — ToC с якорями (8 секций из спеки)
3. **Секция 1 · Два разработчика, одна задача** — хук ($3.00 vs $0.30), определение Context Engineering, карта лекции с метафорой бюджета
4. **Секция 2 · Токен — атом контекста** — что такое токен, BPE на пальцах, почему важно для инженера, токен как единица валюты. Мини-таблица «что сколько стоит в токенах»

Ориентир по объёму: ~80-100 строк. Без кода в этих секциях.

- [ ] **Step 2: Проверить на калькирование с английского**

Перечитать написанный текст. Проверить по правилам из `AGENTS.md`:
- Нет ли untranslated English в русских предложениях
- Нет ли буквального перевода идиом
- Нет ли гибридных слов (англ. корень + рус. суффикс)
- Slide-заголовки и термины — корректная русская адаптация

Допустимо: `tools`, `token`, `BPE`, `prompt`, `MCP`, `JSON`, `CLAUDE.md`, `AGENTS.md`, имена инструментов.

- [ ] **Step 3: Коммит**

```bash
git add content/part6_context_engineering.md
git commit -m "lecture6: add conspect — sections 1-2 (hook + tokenization)"
```

---

### Task 2: Конспект — секции 3-4 (Контекстное окно + Что съедает)

**Files:**
- Modify: `content/part6_context_engineering.md`

- [ ] **Step 1: Дописать секции 3 и 4**

**Секция 3 · Контекстное окно — анатомия:**
- Определение контекстного окна
- Таблица зон контекста (system prompt, MCP-схемы, история, tool calls, запрос, ответ) с колонками «Зона / Что в ней / Кто контролирует»
- Фиксированные vs переменные расходы (метафора: аренда vs операционные)
- Primacy/recency bias — позиция в окне влияет на внимание модели
- Размеры окон 2026: Claude (200K), GPT-4.1 (1M), Gemini (1-2M), GLM 5.1 (фактчек размера окна — в Task 8)

**Секция 4 · Что съедает контекст:**
- Иллюзия контроля — промпт 50 токенов, а в окне уже 63K
- Топ пожирателей (tool results, история, фиксированные расходы, мусорная вежливость) с примерами порядков величин
- Возврат к двум разработчикам из хука — конкретика
- Кривая деградации — скачкообразное падение качества при ~60-70% заполнения

Ориентир по объёму: ~100-120 строк.

- [ ] **Step 2: Проверить на калькирование с английского**

Аналогично Task 1, Step 2.

- [ ] **Step 3: Коммит**

```bash
git add content/part6_context_engineering.md
git commit -m "lecture6: add conspect — sections 3-4 (context window anatomy + what eats context)"
```

---

### Task 3: Конспект — секция 5 (Стратегии управления)

**Files:**
- Modify: `content/part6_context_engineering.md`

- [ ] **Step 1: Дописать секцию 5**

**Секция 5 · Стратегии управления контекстом:**

Три подсекции:
- **5.1 Сокращение входа** — tool calls с прицелом (Read с offset/limit, Grep с фильтром), MCP-гигиена (scoping в CC, per-agent в Kilo, opencode.json), system prompt как бюджет (CLAUDE.md/AGENTS.md — фиксированные расходы; правило: не для каждой сессии → вынести в skill или файл)
- **5.2 Управление историей** — новая сессия как осознанный инструмент; compact/summarization через призму «сигнал vs шум» (качество сжатия зависит от качества входа); что теряется при сжатии (имена файлов, номера строк, промежуточные решения)
- **5.3 Проектирование структуры работы** — декомпозиция как стратегия контекста (чистое окно для субагента); порядок действий (primacy/recency: сначала контекст, потом задача); правило одной задачи (одна сессия = одна задача)

Таблица «стратегия → на что влияет → сколько экономит» (порядок величин).

Ориентир по объёму: ~80-100 строк.

- [ ] **Step 2: Проверить на калькирование с английского**

Аналогично Task 1, Step 2.

- [ ] **Step 3: Коммит**

```bash
git add content/part6_context_engineering.md
git commit -m "lecture6: add conspect — section 5 (context management strategies)"
```

---

### Task 4: Конспект — секции 6-8 (Memory + Подводные камни + Итоги)

**Files:**
- Modify: `content/part6_context_engineering.md`

- [ ] **Step 1: Дописать секции 6, 7 и 8**

**Секция 6 · Memory — контекст за пределами сессии:**
- 6.1 Проблема межсессионного контекста (новая сессия = чистый лист)
- 6.2 Три уровня памяти — таблица (файлы проекта / проектная память / персональная память) с колонками «Уровень / Что хранит / Время жизни / Пример»
- 6.3 Memory как расход контекста — что хранить, что НЕ хранить, принцип «кэш, не архив»
- 6.4 Поддержка в инструментах — краткая сводка (Claude Code, Kilo Code, OpenCode)

**Секция 7 · Подводные камни:**
Формат: проблема → симптом → решение (4 ловушки):
- 7.1 «Большое окно — можно не думать»
- 7.2 Переоптимизация контекста
- 7.3 Memory как свалка
- 7.4 Compact как волшебная кнопка

**Секция 8 · Итоги и практика:**
- 8.1 Шесть ключевых выводов (нумерованный список, каждый с жирным тезисом)
- 8.2 Практическое задание «Аудит контекста» (4 шага)
- 8.3 Следующая лекция — Лекция 7 · RAG

Ориентир по объёму: ~100-120 строк.

- [ ] **Step 2: Проверить на калькирование с английского**

Аналогично Task 1, Step 2.

- [ ] **Step 3: Коммит**

```bash
git add content/part6_context_engineering.md
git commit -m "lecture6: add conspect — sections 6-8 (memory, pitfalls, summary)"
```

---

### Task 5: Манифест и главная страница

**Files:**
- Modify: `content/lectures.json`
- Modify: `index.html`

- [ ] **Step 1: Добавить запись в `content/lectures.json`**

Добавить в конец массива (перед закрывающей `]`):

```json
{
  "id": "part6_context_engineering",
  "title": "Лекция 6 · Context Engineering",
  "description": "Токенизация, контекстное окно, что съедает контекст, стратегии управления, memory-системы",
  "file": "part6_context_engineering.md",
  "slides": "../presentations/part6/"
}
```

- [ ] **Step 2: Обновить `index.html`**

Два изменения:

**a)** Перенести карточку лекции 6 из секции «Планируются» в секцию «В работе». Удалить старую planned-карточку лекции 6. Добавить после карточки лекции 5 в секции «В работе»:

```html
<article class="card clickable" data-href="./presentations/part6/">
  <span class="complexity mid">Средняя сложность</span>
  <h3>Лекция 6 · Context Engineering</h3>
  <p>Токенизация, контекстное окно, что съедает контекст, стратегии управления, memory-системы.</p>
  <div class="row">
    <a class="btn" href="./presentations/part6/">Презентация</a>
    <a class="btn secondary" href="./reading/?part=part6_context_engineering">Материалы лекции</a>
  </div>
</article>
```

**b)** Убедиться, что в секции «Планируются» нет дублирующей карточки лекции 6.

- [ ] **Step 3: Проверить reading page**

Открыть через HTTP: `http://localhost:PORT/reading/?part=part6_context_engineering` — конспект должен рендериться. (Если нет локального сервера — `npx serve .` или `python3 -m http.server`.)

- [ ] **Step 4: Коммит**

```bash
git add content/lectures.json index.html
git commit -m "lecture6: add to manifest and move card to 'in progress' on homepage"
```

---

### Task 6: Презентация — структура, стили и первые слайды (секции 1-2)

**Files:**
- Create: `presentations/part6/index.html`

**Контекст:** Скопировать структуру из `presentations/part5/index.html`:
- Yandex.Metrika counter (тот же сниппет)
- Весь блок `<style>` целиком (CSS-переменные, все классы: `.slide-shell`, `.chapter-shell`, `.card`, `.table-wrap`, `table.mini`, `.process`, `.promo-layout`, `.back-home`, `.mode-toggle` и т.д.)
- Reveal.js подключение и конфиг в конце (тот же)
- Кнопка `.back-home` и `.open-notes`

- [ ] **Step 1: Создать `presentations/part6/index.html`**

Содержание:
1. Полный `<head>` (метрика, title «Context Engineering · Лекция 6», CSS из part5)
2. Кнопки `.back-home` (href `../../`) и `.open-notes` (href `../../reading/?part=part6_context_engineering`)
3. `<div class="reveal"><div class="slides">`

**Титульный слайд:**
```html
<section>
  <div class="chapter-shell">
    <span class="kicker">AI Engineering Course</span>
    <h1 class="hero-title">Context Engineering</h1>
    <p class="hero-sub">Лекция 6</p>
    <div class="meta-row">
      <span class="pill">Фокус: контекстное окно, токены, стратегии, memory</span>
    </div>
  </div>
</section>
```

**Слайд содержания:**
```html
<section>
  <div class="slide-shell">
    <h2 class="section-title">Содержание</h2>
    <div class="cards c3">
      <div class="card teal fragment"><h3>1. Два разработчика</h3><p>$3 vs $0.30 — в чём разница</p></div>
      <div class="card amber fragment"><h3>2. Токен — атом контекста</h3><p>BPE, валюта, расходы</p></div>
      <div class="card rose fragment"><h3>3. Контекстное окно</h3><p>Зоны, позиции, bias</p></div>
      <div class="card green fragment"><h3>4. Что съедает контекст</h3><p>Пожиратели и кривая деградации</p></div>
      <div class="card amber fragment"><h3>5. Стратегии управления</h3><p>Вход, история, структура работы</p></div>
      <div class="card teal fragment"><h3>6. Memory</h3><p>Контекст за пределами сессии</p></div>
      <div class="card rose fragment"><h3>7. Подводные камни</h3><p>4 типичные ловушки</p></div>
      <div class="card green fragment"><h3>8. Итоги и практика</h3><p>Выводы и задание</p></div>
    </div>
  </div>
</section>
```

**Слайды секции 1 (Два разработчика):** ~3-4 слайда:
- Chapter slide (kicker «Часть 1», заголовок)
- Хук — два разработчика, одна задача (карточки c2: красный $3 / зелёный $0.30)
- Определение Context Engineering
- Карта лекции с метафорой бюджета

**Слайды секции 2 (Токен):** ~3-4 слайда:
- Chapter slide (kicker «Часть 2», заголовок)
- Что такое токен + BPE на пальцах
- Таблица «что сколько стоит в токенах» (table.mini)
- Токен как единица валюты — ввод метафоры бюджета

Без закрывающих слайдов и Reveal.js скрипта — они будут добавлены в Task 8.

- [ ] **Step 2: Проверить открытие в браузере**

Открыть `http://localhost:PORT/presentations/part6/` — должен отобразиться титульный слайд. Навигация стрелками работает.

- [ ] **Step 3: Коммит**

```bash
git add presentations/part6/index.html
git commit -m "lecture6 slides: scaffold + sections 1-2 (hook, tokenization)"
```

---

### Task 7: Презентация — слайды секций 3-5

**Files:**
- Modify: `presentations/part6/index.html`

- [ ] **Step 1: Добавить слайды секций 3, 4 и 5**

**Секция 3 (Контекстное окно):** ~3-4 слайда:
- Chapter slide
- Таблица зон контекста (table.mini, 6 строк)
- Фиксированные vs переменные расходы (карточки c2 или визуальная диаграмма)
- Primacy/recency bias + размеры окон 2026

**Секция 4 (Что съедает контекст):** ~3-4 слайда:
- Chapter slide
- Иллюзия контроля (одна карточка с цифрами)
- Топ пожирателей (4 карточки с fragment)
- Кривая деградации (описание или визуальная схема) + возврат к двум разработчикам

**Секция 5 (Стратегии):** ~4-5 слайдов:
- Chapter slide
- 5.1 Сокращение входа (tool calls, MCP-гигиена, system prompt как бюджет)
- 5.2 Управление историей (новая сессия, compact через «сигнал/шум», что теряется)
- 5.3 Проектирование структуры работы (декомпозиция, порядок, правило одной задачи)
- Сводная таблица стратегий (table.mini)

- [ ] **Step 2: Проверить навигацию по слайдам**

Открыть в браузере, пройти все слайды стрелками. Проверить, что fragments работают, таблицы не выходят за пределы.

- [ ] **Step 3: Коммит**

```bash
git add presentations/part6/index.html
git commit -m "lecture6 slides: sections 3-5 (context window, consumers, strategies)"
```

---

### Task 8: Презентация — слайды секций 6-8 + закрывающие слайды

**Files:**
- Modify: `presentations/part6/index.html`

- [ ] **Step 1: Добавить слайды секций 6, 7, 8 и закрывающие**

**Секция 6 (Memory):** ~3-4 слайда:
- Chapter slide
- Проблема межсессионного контекста
- Три уровня памяти (table.mini)
- Memory как расход + что хранить / что не хранить
- Краткая сводка инструментов (карточки c3)

**Секция 7 (Подводные камни):** ~2 слайда:
- Chapter slide
- 4 ловушки (карточки с fragment, формат проблема → решение)

**Секция 8 (Итоги):** ~2-3 слайда:
- Chapter slide
- 6 ключевых выводов (карточки c2 или c3 с fragment)
- Практическое задание «Аудит контекста»

**Закрывающие слайды:**

Слайд «Дальше → Лекция 7»:
```html
<section>
  <div class="chapter-shell">
    <span class="kicker">Дальше</span>
    <h1>Лекция 7</h1>
    <p class="cta">RAG — embedding, vector store, retrieval. Когда контекстного окна не хватает — доставать нужное точечно.</p>
    <div class="meta-row" style="justify-content:center">
      <a class="pill" href="../part1/">Слайды · Лекция 1</a>
      <a class="pill" href="../part2/">Слайды · Лекция 2</a>
      <a class="pill" href="../part3/">Слайды · Лекция 3</a>
      <a class="pill" href="../part4/">Слайды · Лекция 4</a>
      <a class="pill" href="../part5/">Слайды · Лекция 5</a>
      <a class="pill" href="../../">Главная</a>
    </div>
  </div>
</section>
```

Слайд «Остаёмся на связи» — Telegram promo (скопировать из part5: promo-layout с фото + QR-код).

Закрыть `</div></div>`, добавить Reveal.js скрипты и конфигурацию (скопировать из part5).

- [ ] **Step 2: Полный прогон презентации**

Открыть в браузере, пройти все слайды от начала до конца. Проверить:
- Все fragments работают
- Таблицы не обрезаются
- Текст не выходит за slide-shell
- Закрывающие слайды рендерятся
- Кнопка «A: анимации вкл/выкл» работает

- [ ] **Step 3: Коммит**

```bash
git add presentations/part6/index.html
git commit -m "lecture6 slides: sections 6-8 (memory, pitfalls, summary) + closing slides"
```

---

### Task 9: Обновить навигацию в существующих презентациях

**Files:**
- Modify: `presentations/part1/index.html`
- Modify: `presentations/part2/index.html`
- Modify: `presentations/part3/index.html`
- Modify: `presentations/part4/index.html`
- Modify: `presentations/part5/index.html`

**Контекст:** В каждой презентации есть закрывающий слайд «Дальше → Лекция N» с `meta-row` содержащим pill-ссылки на другие лекции. Нужно добавить ссылку на part6 в каждую.

- [ ] **Step 1: Добавить pill-ссылку на лекцию 6 в part1-part5**

В каждом файле найти закрывающий слайд с `meta-row` и pill-ссылками. Добавить:

```html
<a class="pill" href="../part6/">Слайды · Лекция 6</a>
```

Расположить в числовом порядке (после part5, перед ссылкой на главную).

Файлы и примерные строки:
- `part1/index.html` — найти `meta-row` в закрывающем слайде (~строка 1131)
- `part2/index.html` — найти `meta-row` (~строка 1978)
- `part3/index.html` — найти `meta-row` (~строка 1849)
- `part4/index.html` — найти `meta-row` (~строка 2023)
- `part5/index.html` — найти `meta-row` (~строка 1350)

- [ ] **Step 2: Обновить анонс следующей лекции в part5**

В `presentations/part5/index.html` найти закрывающий слайд «Дальше → Лекция 6» (~строка 1344-1357). Обновить текст:

Было: `Context Management — что съедает токены, summarization, compact-режимы. Как видеть, сколько контекста стоит каждое действие.`

Стало: `Context Engineering — токенизация, контекстное окно, стратегии управления контекстом и memory-системы.`

- [ ] **Step 3: Коммит**

```bash
git add presentations/part1/index.html presentations/part2/index.html presentations/part3/index.html presentations/part4/index.html presentations/part5/index.html
git commit -m "lecture6: update navigation pills in all existing presentations"
```

---

### Task 10: Фактчек

**Files:**
- Possibly modify: `content/part6_context_engineering.md`
- Possibly modify: `presentations/part6/index.html`
- Create: `docs/superpowers/reports/2026-04-13-lecture6-factcheck.md`

**Контекст:** Проверить фактические утверждения в конспекте и слайдах. Использовать веб-поиск для верификации. Результат оформить как отчёт.

- [ ] **Step 1: Составить список утверждений для проверки**

Извлечь из конспекта и слайдов все фактические утверждения:

1. Размеры контекстных окон: Claude (200K), GPT-4.1 (1M), Gemini (1-2M), GLM 5.1 (?)
2. Один токен ~ 3-4 символа в английском, ~ 1-2 символа в русском
3. BPE — корректность упрощённого описания алгоритма
4. Кривая деградации ~60-70% — есть ли подтверждающие исследования/данные
5. `function` — один токен (проверить для конкретных токенизаторов)
6. Другие числовые утверждения (15K system prompt, порядки величин tool results и т.д.)
7. Claude Code memory path (`~/.claude/projects/<project>/memory/`)
8. Kilo Code memory — как именно работает, per-workspace
9. OpenCode session state API — корректность описания

- [ ] **Step 2: Провести веб-поиск и верификацию**

Для каждого утверждения — найти первоисточник или авторитетный источник. Зафиксировать:
- Утверждение
- Статус: подтверждено / опровергнуто / не найдено / требует уточнения
- Источник
- Исправление (если нужно)

- [ ] **Step 3: Написать отчёт фактчека**

Создать `docs/superpowers/reports/2026-04-13-lecture6-factcheck.md` с результатами.

- [ ] **Step 4: Внести исправления в конспект и слайды**

Если фактчек выявил ошибки — исправить в `content/part6_context_engineering.md` и `presentations/part6/index.html`.

- [ ] **Step 5: Коммит**

```bash
git add docs/superpowers/reports/2026-04-13-lecture6-factcheck.md content/part6_context_engineering.md presentations/part6/index.html
git commit -m "lecture6: factcheck report + fix verified issues"
```

---

### Task 11: Общее ревью лекции

**Files:**
- Possibly modify: `content/part6_context_engineering.md`
- Possibly modify: `presentations/part6/index.html`

**Контекст:** Финальная проверка всего: конспект, слайды, навигация, рендеринг. Это не фактчек (уже сделан), а проверка целостности, качества и готовности к публикации.

- [ ] **Step 1: Ревью конспекта**

Прочитать `content/part6_context_engineering.md` целиком. Проверить:

- **Структура:** все 8 секций на месте, ToC-якоря корректны
- **Логический поток:** каждая секция логично следует из предыдущей, bottom-up от токена к memory
- **Метафора бюджета:** вводится в секции 2, последовательно используется дальше, не ломается
- **Сигнал/шум:** используется точечно в секции 5.2, не раздут
- **Два разработчика:** хук из секции 1 возвращается в секции 4 и/или 5
- **Лингвистика:** финальная проверка на калькирование с английского (AGENTS.md)
- **Самодостаточность:** лекция понятна без обязательного чтения лекций 1-5 (минимальные отсылки)
- **Таблицы:** все таблицы корректно отформатированы в markdown
- **Длина:** адекватна содержанию, нет водянистых секций

- [ ] **Step 2: Ревью слайдов**

Открыть `presentations/part6/index.html` в браузере. Пройти все слайды:

- **Визуал:** текст не обрезается, таблицы помещаются, карточки не наезжают
- **Fragments:** все fragment-анимации работают корректно
- **Содержание:** соответствует конспекту, не противоречит
- **Закрывающие слайды:** навигация работает, QR-код рендерится
- **Кнопки:** «На главную» ведёт на `/`, «Материалы лекции» — на reading page с правильным `?part=`

- [ ] **Step 3: Проверить cross-навигацию**

- Открыть `index.html` — карточка лекции 6 в секции «В работе», кнопки работают
- Открыть `/reading/?part=part6_context_engineering` — конспект рендерится, навигация работает
- Открыть `/presentations/part5/` — закрывающий слайд содержит правильный анонс лекции 6 и pill-ссылку
- Открыть несколько других лекций — pill-ссылка на part6 присутствует

- [ ] **Step 4: Исправить найденные проблемы**

Если обнаружены проблемы — исправить.

- [ ] **Step 5: Финальный коммит**

```bash
git add -A
git commit -m "lecture6: final review fixes"
```

(Только если были исправления. Если всё чисто — коммит не нужен.)
