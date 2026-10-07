# Визитка vladsazonov.com и переезд курса в /ai-course/ — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** По корню `vladsazonov.com` открывается визитка-табло, курс переезжает в `vladsazonov.com/ai-course/`, а `course.vladsazonov.com` отдаёт 301 на новый адрес.

**Architecture:** Один репозиторий `SazonovV/ai-education`. Курс через `git mv` переезжает в подпапку `ai-course/`, все его ссылки относительные. Визитка — один статический `index.html` в корне: данные лежат массивами в `<script>`, табло рендерит JS, анимация плашек на чистом JS. Работа идёт в git worktree вне `/var/www`. Живая копия переключается через `merge --ff-only` одновременно с подменой конфигов nginx.

**Tech Stack:** чистые HTML/CSS/JS без сборки и npm; nginx; node (только для одноразовых проверочных скриптов в scratchpad); Яндекс Метрика.

**Spec:** `docs/superpowers/specs/2026-10-07-personal-card-ai-course-design.md`

## Global Constraints

- Без build-системы и npm-зависимостей в репозитории. Проверочные скрипты живут в `$TOOLS` (scratchpad) и не коммитятся.
- Все тексты на русском и проверены по правилам `AGENTS.md` (без кальки с английского).
- Телефон и почта на визитке не публикуются. Единственный контакт — Telegram `https://t.me/SazonovMaybeTalks`.
- Счётчик Метрики: `107242055`, `trackLinks: true`.
- Палитра — токены курса: `--bg #080707`, `--bg-soft #12100f`, `--line #3d322a`, `--text #f8f0e4`, `--muted #c7b095`, `--teal #38d7c5`, `--amber #f6a54b`, `--rose #e8657a`, `--green #5ec269`; фон плашки `#1d1916`.
- Шрифты: Unbounded (имя), Manrope (текст), JetBrains Mono (табло) — через Google Fonts.
- Внешние ссылки: `target="_blank" rel="noopener noreferrer"`. Ссылка на курс открывается в той же вкладке.
- `prefers-reduced-motion: reduce` отключает анимацию полностью.
- Без горизонтального скролла на ширине 360 px.
- Живая копия `/var/www/course.vladsazonov.com` не меняется до задачи 5. Пуш в `origin` — только после подтверждения пользователя.

## Review Focus

1. **Старые ссылки с query.** `course.vladsazonov.com/reading/?part=part3_tools_mcp` должна попасть ровно на `vladsazonov.com/ai-course/reading/?part=part3_tools_mcp`, без двойного слэша и без потери query. Проверяется curl в задаче 5.
2. **День выступления и часовой пояс.** В день доклада статус «СЕГОДНЯ», а не «ПРОШЁЛ» или «СКОРО». Дата считается по местному календарю посетителя, без сдвига UTC около полуночи. Тест в задаче 2.
3. **Длинные названия на телефоне.** Название доклада на 70+ символов на ширине 360 px переносится и не создаёт горизонтальный скролл. Проверяется в задаче 4 по `scrollWidth`.
4. **Курс без завершающего слэша.** `vladsazonov.com/ai-course` должен отдавать 301 на `/ai-course/`, а не 404, и относительные пути не должны разрешаться от корня. Проверяется curl в задаче 5.
5. **Кэш старого `/assets/`.** `/assets/*` полгода отдавался с `immutable`. Корневой `/assets/avatar.jpg` должен остаться тем же файлом, тогда кэш у посетителей не испортится. Проверяется `cmp` в задаче 1.

---

## Переменные окружения исполнителя

Используются во всех задачах:

```bash
LIVE=/var/www/course.vladsazonov.com
SCRATCH=/tmp/claude-0/-var-www-course-vladsazonov-com/8f064a1c-82ce-463e-abad-50618a319934/scratchpad
WT=$SCRATCH/wt-personal-card
TOOLS=$SCRATCH/tools
```

## Карта файлов

| Файл | Действие | Ответственность |
|---|---|---|
| `ai-course/**` | `git mv` из корня | курс целиком, без изменения содержимого (кроме двух правок ниже) |
| `ai-course/index.html` | правка | `og:url`/`og:image`, имя автора ссылается на визитку |
| `assets/avatar.jpg` | копия | фото для визитки |
| `index.html` | новый | визитка-табло: данные, рендер, анимация, стили |
| `vercel.json` | удалить | — |
| `AGENTS.md`, `README.md` | правка | новая структура, URL, «как добавить выступление» |
| `$TOOLS/check-links.mjs` | новый, вне репо | проверка относительных ссылок курса |
| `$TOOLS/test-status.mjs` | новый, вне репо | тесты чистых функций табло |
| `/etc/nginx/conf.d/00-course-maps.conf` | правка | карта Cache-Control |
| `/etc/nginx/snippets/course-site.conf` | правка | правило `.md` под `/ai-course/` |
| `/etc/nginx/sites-available/vladsazonov.com` | замена | отдача сайта вместо редиректа |
| `/etc/nginx/sites-available/course.vladsazonov.com` | замена | 301 на `/ai-course/` |

---

### Задача 1: Worktree, переезд курса в `ai-course/`, проверка ссылок

**Files:**
- Move: `index.html`, `reading/`, `presentations/`, `content/`, `assets/` → `ai-course/`
- Create: `assets/avatar.jpg` (копия)
- Delete: `vercel.json`
- Modify: `ai-course/index.html:9-10` (og), `ai-course/index.html:346` (имя автора), CSS блока `.author-info h2`
- Test: `$TOOLS/check-links.mjs`

**Interfaces:**
- Produces: каталог `ai-course/` с рабочими относительными ссылками. Корень `index.html` свободен для задачи 2. `assets/avatar.jpg` в корне.

- [ ] **Шаг 1: Создать worktree на ветке `personal-card`**

```bash
git -C $LIVE worktree add -b personal-card $WT master
mkdir -p $TOOLS
git -C $WT log --oneline -1
```
Ожидание: последний коммит — коммит спецификации или плана.

- [ ] **Шаг 2: Написать проверку ссылок**

`$TOOLS/check-links.mjs`:

```js
// Usage: node check-links.mjs <repoRoot> <subdir>
// Проверяет, что относительные href/src и JS-константы в HTML внутри <subdir>
// и пути из content/lectures.json указывают на существующие файлы внутри <repoRoot>.
import fs from "node:fs";
import path from "node:path";

const [root, sub] = process.argv.slice(2);
const base = path.resolve(root, sub);
const errors = [];
let checked = 0;

function walk(dir) {
  return fs.readdirSync(dir, { withFileTypes: true }).flatMap(e => {
    const p = path.join(dir, e.name);
    return e.isDirectory() ? walk(p) : [p];
  });
}

function isLocal(u) {
  return u && !/^(https?:|data:|mailto:|tel:|javascript:|#|\/\/)/i.test(u) && !u.includes("${");
}

function resolveTarget(fromFile, u) {
  const clean = u.split("#")[0].split("?")[0];
  let target = clean === "" ? fromFile : path.resolve(path.dirname(fromFile), clean);
  if (clean === "" || clean.endsWith("/") || (fs.existsSync(target) && fs.statSync(target).isDirectory())) {
    target = path.join(target, "index.html");
  }
  return target;
}

function check(fromFile, u, kind) {
  if (!isLocal(u)) return;
  checked++;
  if (u.startsWith("/")) { errors.push(`${fromFile}: absolute path ${kind} "${u}"`); return; }
  const t = resolveTarget(fromFile, u);
  if (!t.startsWith(path.resolve(root) + path.sep)) errors.push(`${fromFile}: ${kind} "${u}" escapes repo`);
  else if (!fs.existsSync(t)) errors.push(`${fromFile}: ${kind} "${u}" -> missing ${path.relative(root, t)}`);
}

for (const file of walk(base).filter(f => f.endsWith(".html"))) {
  const html = fs.readFileSync(file, "utf8");
  for (const m of html.matchAll(/\b(href|src)\s*=\s*"([^"]*)"/g)) check(file, m[2], m[1]);
  for (const m of html.matchAll(/\.href\s*=\s*"([^"]+)"/g)) check(file, m[1], "js-href");
  for (const m of html.matchAll(/const\s+(LECTURES_URL|CONTENT_BASE)\s*=\s*"([^"]+)"/g)) check(file, m[2], m[1]);
}

const lecturesPath = path.join(base, "content", "lectures.json");
if (fs.existsSync(lecturesPath)) {
  const reader = path.join(base, "reading", "index.html");
  const raw = JSON.parse(fs.readFileSync(lecturesPath, "utf8"));
  const lectures = Array.isArray(raw) ? raw : raw.lectures;
  for (const lec of lectures) {
    check(reader, "../content/" + lec.file, "lecture.file");
    if (lec.slides) check(reader, lec.slides, "lecture.slides");
  }
}

console.log(`checked ${checked} links`);
if (errors.length) { console.error(errors.join("\n")); process.exit(1); }
console.log("OK");
```

- [ ] **Шаг 3: Прогнать проверку на текущем корне (до переезда), чтобы убедиться, что скрипт рабочий**

```bash
node $TOOLS/check-links.mjs $WT .
```
Ожидание: `checked N links` с N > 30 и `OK`. Если `lectures.json` не массив и не `{lectures: [...]}`, поправить чтение в скрипте под фактическую форму файла и прогнать ещё раз.

Затем убедиться, что скрипт ловит поломку:

```bash
node $TOOLS/check-links.mjs $WT presentations
```
Ожидание: `OK`, потому что `../../` из `presentations/partN/` указывает на корневой `index.html`, который существует. Потом временно переименовать файл:

```bash
mv $WT/content/lectures.json $WT/content/lectures.json.bak
node $TOOLS/check-links.mjs $WT . ; echo "exit=$?"
mv $WT/content/lectures.json.bak $WT/content/lectures.json
```
Ожидание: ошибка `LECTURES_URL "../content/lectures.json" -> missing content/lectures.json` и `exit=1`.

- [ ] **Шаг 4: Перенести курс, скопировать аватар, удалить vercel.json**

```bash
cd $WT
mkdir ai-course
git mv index.html reading presentations content assets ai-course/
mkdir assets && cp ai-course/assets/avatar.jpg assets/avatar.jpg
git rm -q vercel.json
cmp assets/avatar.jpg ai-course/assets/avatar.jpg && echo SAME
git status --short | head -20
```
Ожидание: `SAME`. Переименования `R  index.html -> ai-course/index.html` и т. п., `D vercel.json`.

- [ ] **Шаг 5: Проверить ссылки курса после переезда**

```bash
node $TOOLS/check-links.mjs $WT ai-course
```
Ожидание: `OK`. Ошибка «escapes repo» или «missing» означает абсолютную или сломанную ссылку — исправить в `ai-course/` и прогнать снова.

- [ ] **Шаг 6: Обновить og-теги и ссылку автора в `ai-course/index.html`**

Заменить строки 9–10:

```html
<meta property="og:image" content="https://vladsazonov.com/ai-course/assets/og-image.png">
<meta property="og:url" content="https://vladsazonov.com/ai-course/">
```

Заменить `<h2>Владислав Сазонов</h2>` в блоке `.author-info`:

```html
<h2><a class="author-name" href="../">Владислав Сазонов</a></h2>
```

Добавить в `<style>` сразу после правила `.author-info h2 { … }`:

```css
.author-name {
  color: inherit;
  text-decoration: none;
}

.author-name:hover,
.author-name:focus-visible {
  color: var(--teal);
}
```

- [ ] **Шаг 7: Повторить проверку ссылок**

```bash
node $TOOLS/check-links.mjs $WT ai-course
```
Ожидание: `OK`. Ссылка `../` ведёт на корневой `index.html`, которого пока нет, поэтому возможна ошибка `missing index.html`. Она уйдёт в задаче 2: временно создать заглушку `echo '<!doctype html>' > $WT/index.html`, прогнать проверку и не коммитить заглушку (`rm $WT/index.html`).

- [ ] **Шаг 8: Коммит**

```bash
cd $WT
git add -A ai-course assets
git commit -m "move course into /ai-course/, drop vercel.json

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Задача 2: Визитка — данные, чистые функции, статичный рендер табло

**Files:**
- Create: `index.html`
- Test: `$TOOLS/test-status.mjs`

**Interfaces:**
- Consumes: `assets/avatar.jpg` из задачи 1; `./ai-course/` как цель ссылки курса.
- Produces (внутри `index.html`, между маркерами `// BEGIN pure` и `// END pure`):
  - `todayISO(date: Date): string` — `"YYYY-MM-DD"` по местному времени;
  - `talkStatus(talk: {date, slides, video}, today: string): {label: string, tone: "soon"|"done"|"past"}`;
  - `talkHref(talk: {url, slides, video}): string` — `video || slides || url`;
  - `fmtDate(iso: string): string` — `"16.10.26"`.
- Produces (DOM для задачи 3): каждая строка — `a.row` внутри `ul.rows`, поля плашками — `span.flaps[aria-hidden=true][data-text]` с дочерними `span.flap`, рядом `span.vh` с настоящим текстом.

- [ ] **Шаг 1: Написать падающий тест чистых функций**

`$TOOLS/test-status.mjs`:

```js
// Usage: node test-status.mjs <path/to/index.html>
import fs from "node:fs";
import assert from "node:assert/strict";

const html = fs.readFileSync(process.argv[2], "utf8");
const m = html.match(/\/\/ BEGIN pure([\s\S]*?)\/\/ END pure/);
assert.ok(m, "markers // BEGIN pure ... // END pure not found");
const { todayISO, talkStatus, talkHref, fmtDate } =
  new Function(m[1] + "\nreturn { todayISO, talkStatus, talkHref, fmtDate };")();

const t = (date, extra = {}) => ({ date, url: "https://conf", slides: null, video: null, ...extra });
const today = "2026-10-16";

assert.deepEqual(talkStatus(t("2026-10-17"), today), { label: "СКОРО", tone: "soon" });
assert.deepEqual(talkStatus(t("2026-10-16"), today), { label: "СЕГОДНЯ", tone: "soon" });
assert.deepEqual(talkStatus(t("2026-10-15"), today), { label: "ПРОШЁЛ", tone: "past" });
assert.deepEqual(talkStatus(t("2026-10-15", { slides: "s" }), today), { label: "СЛАЙДЫ", tone: "done" });
assert.deepEqual(talkStatus(t("2026-10-15", { video: "v" }), today), { label: "ВИДЕО", tone: "done" });
assert.deepEqual(talkStatus(t("2026-10-15", { video: "v", slides: "s" }), today), { label: "ВИДЕО", tone: "done" });
// будущий доклад с заранее выложенными слайдами всё равно СКОРО
assert.deepEqual(talkStatus(t("2026-12-01", { slides: "s" }), today), { label: "СКОРО", tone: "soon" });

assert.equal(talkHref(t("2026-01-01")), "https://conf");
assert.equal(talkHref(t("2026-01-01", { slides: "s" })), "s");
assert.equal(talkHref(t("2026-01-01", { slides: "s", video: "v" })), "v");

// местная дата, а не UTC: 00:30 16 октября по местному времени — это 16-е
assert.equal(todayISO(new Date(2026, 9, 16, 0, 30)), "2026-10-16");
assert.equal(todayISO(new Date(2026, 0, 5, 23, 59)), "2026-01-05");

assert.equal(fmtDate("2026-10-16"), "16.10.26");
assert.equal(fmtDate("2025-08-28"), "28.08.25");

console.log("all status tests passed");
```

- [ ] **Шаг 2: Убедиться, что тест падает**

```bash
node $TOOLS/test-status.mjs $WT/index.html
```
Ожидание: `ENOENT` (файла ещё нет). Это и есть падение.

- [ ] **Шаг 3: Создать `index.html` с данными, чистыми функциями и рендером**

`$WT/index.html`:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Владислав Сазонов</title>
<meta name="description" content="ИТ-лидер в Альфа-Банке · AI в инженерных командах. Курс Agentic Engineering, выступления, менторство.">
<meta property="og:title" content="Владислав Сазонов">
<meta property="og:description" content="ИТ-лидер в Альфа-Банке · AI в инженерных командах. Курс, выступления, менторство.">
<meta property="og:url" content="https://vladsazonov.com/">
<meta property="og:type" content="profile">
<meta property="og:locale" content="ru_RU">
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>✈️</text></svg>">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;700&family=Unbounded:wght@700&family=JetBrains+Mono:wght@500&display=swap" rel="stylesheet">
<style>
:root {
  --bg: #080707;
  --bg-soft: #12100f;
  --line: #3d322a;
  --text: #f8f0e4;
  --muted: #c7b095;
  --teal: #38d7c5;
  --amber: #f6a54b;
  --rose: #e8657a;
  --green: #5ec269;
  --flap-bg: #1d1916;
  --mono: "JetBrains Mono", ui-monospace, Menlo, monospace;
}

* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: "Manrope", "Segoe UI", sans-serif;
  color: var(--text);
  background:
    radial-gradient(circle at 10% 12%, rgba(56,215,197,0.14), transparent 35%),
    radial-gradient(circle at 90% 10%, rgba(246,165,75,0.14), transparent 28%),
    linear-gradient(145deg, #090807 0%, #0f0d0c 55%, #090807 100%);
  background-color: var(--bg);
  min-height: 100vh;
  padding: 48px 16px;
}

body::after {
  content: "";
  position: fixed;
  inset: 0;
  pointer-events: none;
  opacity: 0.028;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  background-size: 180px;
}

.shell { position: relative; z-index: 1; max-width: 960px; margin: 0 auto; }

.vh {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; border: 0;
}

/* ---- Шапка ---- */
.who { display: flex; align-items: center; gap: 22px; margin-bottom: 40px; }
.who .photo { width: 104px; height: 104px; border-radius: 50%; object-fit: cover; flex-shrink: 0; }
.who h1 { font-family: "Unbounded", "Manrope", sans-serif; font-size: 28px; margin: 0 0 6px; }
.who .role { color: var(--teal); font-weight: 700; margin: 0 0 6px; }
.who .bio { color: var(--muted); margin: 0 0 4px; line-height: 1.55; }
.who .extra { color: var(--muted); opacity: .75; font-size: 13px; margin: 0; }

/* ---- Табло ---- */
.board { margin-bottom: 36px; }
.board-title {
  font: 700 12px/1 "Manrope", sans-serif;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: rgba(56, 215, 197, 0.75);
  margin: 0 0 10px;
}
.board-head, .row {
  display: grid;
  gap: 8px 16px;
  align-items: center;
  padding: 10px 12px;
}
.board-head {
  font: 500 11px/1 var(--mono);
  letter-spacing: 1px;
  text-transform: uppercase;
  color: var(--muted);
  opacity: .7;
  border-bottom: 1px solid var(--line);
}
.board--dest .board-head, .board--dest .row { grid-template-columns: minmax(0, 1.7fr) minmax(0, 1fr) auto; }
.board--talks .board-head, .board--talks .row { grid-template-columns: auto minmax(0, 2.2fr) minmax(0, 1fr) auto; }

.rows { list-style: none; margin: 0; padding: 0; }
.row {
  color: var(--text);
  text-decoration: none;
  border-bottom: 1px solid var(--line);
  transition: background-color .15s;
}
.row:hover { background: rgba(246, 165, 75, 0.06); }
.row:focus-visible { outline: 2px solid var(--teal); outline-offset: -2px; }

.c-text { font-family: var(--mono); font-size: 13px; color: var(--muted); min-width: 0; }
.c-title { color: var(--amber); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

/* ---- Плашки ---- */
.flaps { display: flex; flex-wrap: wrap; gap: 2px; min-width: 0; }
.flap {
  position: relative;
  display: inline-grid;
  place-items: center;
  width: 1.05em;
  height: 1.5em;
  background: var(--flap-bg);
  color: var(--amber);
  border-radius: 3px;
  font: 500 15px/1 var(--mono);
}
.flap::after {
  content: "";
  position: absolute;
  left: 0; right: 0; top: 50%;
  height: 1px;
  background: rgba(0, 0, 0, 0.6);
}
.flap--space { background: transparent; width: .55em; }
.flap--space::after { display: none; }

.tone-open .flap, .tone-done .flap { color: var(--teal); }
.tone-soon .flap { color: var(--rose); }
.tone-past .flap { color: var(--muted); }

.plain { line-height: 2; }
.plain a { color: var(--teal); }

/* ---- Телефон ---- */
@media (max-width: 719px) {
  body { padding: 32px 16px; }
  .who { flex-direction: column; align-items: flex-start; gap: 14px; }
  .who .photo { width: 80px; height: 80px; }
  .who h1 { font-size: 22px; }
  .board-head { display: none; }
  .board--dest .row, .board--talks .row { grid-template-columns: minmax(0, 1fr) auto; }
  .c-key { order: 0; }
  .c-status { order: 1; }
  .c-title { order: 2; grid-column: 1 / -1; white-space: normal; }
  .c-where { order: 3; grid-column: 1 / -1; }
  .flap { font-size: 13px; }
}
</style>
</head>
<body>
<main class="shell">
  <header class="who">
    <img class="photo" src="./assets/avatar.jpg" alt="Владислав Сазонов" width="104" height="104">
    <div>
      <h1>Владислав Сазонов</h1>
      <p class="role">ИТ-лидер в Альфа-Банке · AI в инженерных командах</p>
      <p class="bio">Работаю на стыке управления и ИИ: смотрю, как ИИ меняет устройство команд и процессов.</p>
      <p class="extra">Программный комитет трека AI Native, Ontico</p>
    </div>
  </header>

  <section class="board board--dest" aria-labelledby="dest-title">
    <h2 class="board-title" id="dest-title">Направления</h2>
    <div class="board-head" aria-hidden="true"><span>Направление</span><span>Где</span><span>Статус</span></div>
    <ul class="rows" id="destinations"></ul>
    <noscript>
      <ul class="plain">
        <li><a href="./ai-course/">Курс Agentic Engineering</a></li>
        <li><a href="https://getmentor.dev/mentor/vladislav-sazonov-7056" target="_blank" rel="noopener noreferrer">Менторство на GetMentor</a></li>
        <li><a href="https://t.me/SazonovMaybeTalks" target="_blank" rel="noopener noreferrer">Telegram-канал «Вы не просили — я рассказал»</a></li>
      </ul>
    </noscript>
  </section>

  <section class="board board--talks" aria-labelledby="talks-title">
    <h2 class="board-title" id="talks-title">Выступления</h2>
    <div class="board-head" aria-hidden="true"><span>Дата</span><span>Выступление</span><span>Где</span><span>Статус</span></div>
    <ul class="rows" id="talks"></ul>
  </section>
</main>

<script>
// Направления: меняются редко. При правке обновить и <noscript> выше.
const DESTINATIONS = [
  { name: "КУРС AGENTIC ENGINEERING", where: "/ai-course/", status: "ОТКРЫТ", url: "./ai-course/", external: false },
  { name: "МЕНТОРСТВО", where: "GetMentor", status: "ЗАПИСЬ", url: "https://getmentor.dev/mentor/vladislav-sazonov-7056", external: true },
  { name: "TELEGRAM", where: "«Вы не просили — я рассказал»", status: "LIVE", url: "https://t.me/SazonovMaybeTalks", external: true },
];

// Выступления: добавить доклад = дописать строку. Порядок не важен, сортировка по дате.
// slides / video — ссылки на материалы или null; статус считается автоматически.
const TALKS = [
  { date: "2026-10-16", title: "Ужесточаем harness, или Сага об агентской предсказуемости", where: "ГигаКонф", url: "https://gigaconf.ru/program", slides: null, video: null },
  { date: "2026-09-10", title: "Молиться на модель или строить harness", where: "TeamLead Conf Сибирь", url: "https://teamleadconf.ru/siberia/2026/abstracts/18628", slides: "https://disk.yandex.ru/i/qI7HiEa-vA2wdA", video: null },
  { date: "2026-07-03", title: "Круглый стол «MCP vs CLI, harness, агентские платформы»", where: "Agentic Dev Conf", url: "https://agenticdevconf.ru/talks/sazonov_vlad", slides: null, video: null },
  { date: "2025-10-21", title: "Гильдии фронтендеров: что это и нужно ли вообще", where: "FrontendConf", url: "https://frontendconf.ru/moscow/2025/abstracts/16364", slides: null, video: null },
  { date: "2025-08-28", title: "Гильдия: место, где разработчик перестаёт быть одиноким кузнецом", where: "MoscowJS 67", url: "http://digital.alfabank.ru/events/moscow_JS_67", slides: null, video: null },
];

// BEGIN pure
function todayISO(d) {
  const p = n => String(n).padStart(2, "0");
  return `${d.getFullYear()}-${p(d.getMonth() + 1)}-${p(d.getDate())}`;
}

function talkStatus(talk, today) {
  if (talk.date > today) return { label: "СКОРО", tone: "soon" };
  if (talk.date === today) return { label: "СЕГОДНЯ", tone: "soon" };
  if (talk.video) return { label: "ВИДЕО", tone: "done" };
  if (talk.slides) return { label: "СЛАЙДЫ", tone: "done" };
  return { label: "ПРОШЁЛ", tone: "past" };
}

function talkHref(talk) {
  return talk.video || talk.slides || talk.url;
}

function fmtDate(iso) {
  const [y, m, d] = iso.split("-");
  return `${d}.${m}.${y.slice(2)}`;
}
// END pure

function el(tag, className, text) {
  const node = document.createElement(tag);
  if (className) node.className = className;
  if (text != null) node.textContent = text;
  return node;
}

// Поле плашками: визуальные плашки скрыты от скринридеров, настоящий текст — в .vh.
function flapCell(text, className) {
  const cell = el("span", className);
  const flaps = el("span", "flaps");
  flaps.setAttribute("aria-hidden", "true");
  flaps.dataset.text = text;
  for (const ch of text) {
    flaps.append(el("span", ch === " " ? "flap flap--space" : "flap", ch === " " ? "" : ch));
  }
  cell.append(flaps, el("span", "vh", text));
  return cell;
}

function rowLink(url, external, title) {
  const a = el("a", "row");
  a.href = url;
  if (external) { a.target = "_blank"; a.rel = "noopener noreferrer"; }
  if (title) a.title = title;
  return a;
}

function renderDestinations(list) {
  const ul = document.getElementById("destinations");
  for (const d of list) {
    const a = rowLink(d.url, d.external);
    a.append(
      flapCell(d.name, "c-key"),
      el("span", "c-text c-where", d.where),
      flapCell(d.status, "c-status tone-open"),
    );
    const li = el("li");
    li.append(a);
    ul.append(li);
  }
}

function renderTalks(list, today) {
  const ul = document.getElementById("talks");
  const sorted = [...list].sort((a, b) => b.date.localeCompare(a.date));
  for (const t of sorted) {
    const status = talkStatus(t, today);
    const a = rowLink(talkHref(t), true, `${t.title} — ${t.where}`);
    a.append(
      flapCell(fmtDate(t.date), "c-key"),
      el("span", "c-text c-title", t.title),
      el("span", "c-text c-where", t.where),
      flapCell(status.label, `c-status tone-${status.tone}`),
    );
    const li = el("li");
    li.append(a);
    ul.append(li);
  }
}

renderDestinations(DESTINATIONS);
renderTalks(TALKS, todayISO(new Date()));
</script>

<script>
(function(m,e,t,r,i,k,a){m[i]=m[i]||function(){(m[i].a=m[i].a||[]).push(arguments)};
m[i].l=1*new Date();
for(var j=0;j<document.scripts.length;j++){if(document.scripts[j].src===r){return}}
k=e.createElement(t),a=e.getElementsByTagName(t)[0],k.async=1,k.src=r,a.parentNode.insertBefore(k,a)})
(window,document,"script","https://mc.yandex.ru/metrika/tag.js","ym");
ym(107242055,"init",{clickmap:true,trackLinks:true,accurateTrackBounce:true,webvisor:true});
</script>
<noscript><div><img src="https://mc.yandex.ru/watch/107242055" style="position:absolute;left:-9999px" alt=""></div></noscript>
</body>
</html>
```

- [ ] **Шаг 4: Тест проходит**

```bash
node $TOOLS/test-status.mjs $WT/index.html
```
Ожидание: `all status tests passed`.

- [ ] **Шаг 5: Ссылки курса и визитки сходятся**

```bash
node $TOOLS/check-links.mjs $WT ai-course
grep -c 'href="./ai-course/"' $WT/index.html
```
Ожидание: `OK` (`../` из курса теперь ведёт на настоящий `index.html`), счётчик `1` (ссылка в noscript).

- [ ] **Шаг 6: Коммит**

```bash
cd $WT
git add index.html
git commit -m "add vladsazonov.com card: departures board with destinations and talks

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Задача 3: Анимация перелистывания плашек

**Files:**
- Modify: `index.html` — новый `<script>` сразу после скрипта рендера (до Метрики)

**Interfaces:**
- Consumes: DOM из задачи 2 — `a.row`, `span.flaps[data-text] > span.flap` (у пробела `flap--space`).
- Produces: функции `flipRow(row: HTMLElement, delay: number): void` и `boardIntro(): void`.

- [ ] **Шаг 1: Добавить скрипт анимации**

Вставить в `index.html` после закрывающего `</script>` блока рендера:

```html
<script>
// Перелистывание плашек: каждая прокручивает несколько случайных символов и встаёт на свой.
const FLAP_CHARSET = "АБВГДЕЖЗИКЛМНОПРСТУФХЦЧШЭЮЯ0123456789";
const FLAP_TICK_MS = 55;
const ROW_STAGGER_MS = 110;
const CHAR_STAGGER_MS = 14;
const reduceMotion = window.matchMedia("(prefers-reduced-motion: reduce)");

function flipFlap(flap, target, delay) {
  const turns = 4 + Math.floor(Math.random() * 5);
  let n = 0;
  setTimeout(function tick() {
    if (n < turns) {
      flap.textContent = FLAP_CHARSET[Math.floor(Math.random() * FLAP_CHARSET.length)];
      n++;
      setTimeout(tick, FLAP_TICK_MS);
    } else {
      flap.textContent = target;
    }
  }, delay);
}

function flipRow(row, delay) {
  if (reduceMotion.matches || row.dataset.flipping) return;
  row.dataset.flipping = "1";
  let longest = 0;
  for (const group of row.querySelectorAll(".flaps")) {
    const chars = [...group.dataset.text];
    group.querySelectorAll(".flap").forEach((flap, i) => {
      if (chars[i] === " ") return;
      const d = delay + i * CHAR_STAGGER_MS;
      flipFlap(flap, chars[i], d);
      longest = Math.max(longest, d + 9 * FLAP_TICK_MS);
    });
  }
  setTimeout(() => { delete row.dataset.flipping; }, longest);
}

function boardIntro() {
  const rows = document.querySelectorAll(".row");
  rows.forEach((row, i) => {
    flipRow(row, i * ROW_STAGGER_MS);
    row.addEventListener("mouseenter", () => flipRow(row, 0));
    row.addEventListener("focus", () => flipRow(row, 0));
  });
}

boardIntro();
</script>
```

- [ ] **Шаг 2: Тесты чистых функций не сломались**

```bash
node $TOOLS/test-status.mjs $WT/index.html
```
Ожидание: `all status tests passed`.

- [ ] **Шаг 3: Синтаксис всех inline-скриптов**

```bash
node -e '
const html = require("fs").readFileSync(process.argv[1], "utf8");
const scripts = [...html.matchAll(/<script>([\s\S]*?)<\/script>/g)].map(m => m[1]);
scripts.forEach((s, i) => { new Function(s); console.log("script", i, "ok"); });
' $WT/index.html
```
Ожидание: `script 0 ok`, `script 1 ok`, `script 2 ok`.

- [ ] **Шаг 4: Коммит**

```bash
cd $WT
git add index.html
git commit -m "card: split-flap animation with reduced-motion support

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Задача 4: Документация и визуальная приёмка

**Files:**
- Modify: `AGENTS.md`, `README.md`

**Interfaces:**
- Consumes: структура из задач 1–3, массивы `DESTINATIONS` и `TALKS`.

- [ ] **Шаг 1: Обновить `AGENTS.md`**

Заменить первые абзацы раздела `## Architecture` до `### Reading pages (conspects)` на:

```markdown
## Architecture

Static site (no build step, no npm) served by nginx on our own server (not Vercel).

- `/` — personal card (`/index.html`): a split-flap departures board linking to the course, mentoring, Telegram and talks.
- `/ai-course/` — the AI Engineering course (everything below lives under `ai-course/`).
- `course.vladsazonov.com/*` permanently redirects (301) to `vladsazonov.com/ai-course/*`.

All links inside `ai-course/` must stay relative — the course must not assume it is served from the site root.
```

В разделе `### Reading pages (conspects)` заменить пути и URL:

```markdown
All reading pages are served by a **single dynamic SPA**: `/ai-course/reading/index.html`.

- Lectures are defined in `/ai-course/content/lectures.json` (id, title, description, file, slides)
- Markdown content lives in `/ai-course/content/*.md`
- The reader loads markdown dynamically via `?part=<id>` query parameter
- **Do NOT create separate `/ai-course/reading/partN/` directories** — they were removed as redundant

Key URLs:
- `/ai-course/reading/` — index with lecture cards
- `/ai-course/reading/?part=part1_vibecoding` — lecture 1
- `/ai-course/reading/?part=part2_prompt_engineering` — lecture 2
- `/ai-course/reading/?part=part3_tools_mcp` — lecture 3
```

`### Presentations`: `Reveal.js slides live in /ai-course/presentations/partN/index.html. Each is a standalone HTML file.`

`### Navigation in reading pages`: строку про «На главную» заменить на `- "На главную" — link to the course home (/ai-course/)`.

`### Adding a new lecture`:

```markdown
1. Create `/ai-course/content/partN_<slug>.md`
2. Add entry to `/ai-course/content/lectures.json`
3. Create `/ai-course/presentations/partN/index.html`
4. Add card to `/ai-course/index.html`
```

Добавить после него новый раздел:

```markdown
### Personal card: adding a talk

Talks live in the `TALKS` array in `/index.html`. Add one object:

`{ date: "YYYY-MM-DD", title: "…", where: "Conference", url: "talk page", slides: null, video: null }`

Order does not matter (sorted by date). Status (СКОРО / СЕГОДНЯ / ВИДЕО / СЛАЙДЫ / ПРОШЁЛ) is computed from the date and materials; when slides or video appear, fill `slides` / `video`. Destinations live in `DESTINATIONS` — when changing them, update the `<noscript>` list too. No phone or e-mail on the card; Telegram is the only contact.
```

В `## Tech stack` заменить `- Deployed on Vercel (vercel.json)` на `- Served by nginx from this repo's working copy (configs in /etc/nginx, outside the repo)`.

- [ ] **Шаг 2: Обновить `README.md`**

````markdown
# vladsazonov.com

Личная страница и курс по AI-инженерии — от вайбкодинга до мультиагентных систем.

- **Визитка:** [vladsazonov.com](https://vladsazonov.com)
- **Курс:** [vladsazonov.com/ai-course/](https://vladsazonov.com/ai-course/)

## Структура

```text
.
├── index.html           # визитка-табло
├── assets/              # фото для визитки
└── ai-course/
    ├── assets/          # изображения курса
    ├── content/         # исходные .md конспекты и lectures.json
    ├── presentations/   # Reveal.js слайды (part1–part6)
    ├── reading/         # SPA чтения конспектов
    └── index.html       # главная курса
```
````

Остальные разделы README, если они есть ниже «Структуры», сохранить и поправить пути на `ai-course/…`.

- [ ] **Шаг 3: Проверить язык**

```bash
grep -nE "\b(evidence|bite-sized|per step|first-class)\b" $WT/index.html $WT/README.md || echo "no calques"
```
Ожидание: `no calques`. Затем перечитать тексты визитки (шапка, подписи табло, названия статусов) по правилу из `AGENTS.md`: если носитель запнётся и мысленно переведёт фразу — переписать.

- [ ] **Шаг 4: Коммит**

```bash
cd $WT
git add AGENTS.md README.md
git commit -m "docs: describe card + /ai-course/ layout and how to add a talk

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

- [ ] **Шаг 5: Поднять локальный просмотр worktree**

```bash
cd $WT && python3 -m http.server 50830 --bind 127.0.0.1
```
(Запускать в фоне.) Проверка: `curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:50830/` и `.../ai-course/reading/?part=part1_vibecoding` → `200`. Попросить пользователя добавить в SSH-туннель `-L 50830:localhost:50830` и открыть `http://localhost:50830/`.

- [ ] **Шаг 6: Визуальная приёмка вместе с пользователем**

Показать в визуальном помощнике экран с двумя `<iframe src="http://localhost:50830/">` шириной 1100 px и 360 px. Проверить с пользователем:
- плашки перелистываются при загрузке, строки идут каскадом, наведение перелистывает строку ещё раз;
- на 360 px нет горизонтального скролла: в консоли браузера `document.documentElement.scrollWidth <= innerWidth` → `true`; длинное название «Гильдия: место, где разработчик перестаёт быть одиноким кузнецом» переносится;
- клик по «Курс» открывает `/ai-course/`, имя автора на главной курса возвращает на визитку;
- с включённым в системе «уменьшением движения» текст появляется сразу.

Правки по замечаниям — отдельными коммитами в ветку, затем повторить шаги 2–3 задачи 3. Дальше — только после явного «да» пользователя.

- [ ] **Шаг 7: Остановить локальный сервер** (`kill` фонового процесса из шага 5).

---

### Задача 5: Переключение живого сайта и nginx

**Files:**
- Modify: `/etc/nginx/conf.d/00-course-maps.conf`, `/etc/nginx/snippets/course-site.conf`
- Replace: `/etc/nginx/sites-available/vladsazonov.com`, `/etc/nginx/sites-available/course.vladsazonov.com`

**Interfaces:**
- Consumes: ветка `personal-card` (задачи 1–4), одобренная пользователем.

- [ ] **Шаг 1: Резервные копии**

```bash
B=/root/nginx-backup-2026-10-07
mkdir -p $B
cp -a /etc/nginx/conf.d/00-course-maps.conf /etc/nginx/snippets/course-site.conf \
      /etc/nginx/sites-available/vladsazonov.com /etc/nginx/sites-available/course.vladsazonov.com $B/
ls $B
```
Ожидание: четыре файла.

- [ ] **Шаг 2: Подготовить новые конфиги в `$SCRATCH/nginx-new/`**

`$SCRATCH/nginx-new/00-course-maps.conf`:

```nginx
# Cache-Control policy for vladsazonov.com (card at /, course at /ai-course/).
# Regexes are evaluated in the order listed - most specific first.
map $uri $course_cache_control {
    ~*^/ai-course/content/.+\.(md|json)$  "public, max-age=0, must-revalidate";
    ~*^/(ai-course/)?assets/              "public, max-age=31536000, immutable";
    ~*\.html$                             "public, max-age=0, must-revalidate";
    ~*/$                                  "public, max-age=0, must-revalidate";
    default                               "public, max-age=300";
}
```

`$SCRATCH/nginx-new/course-site.conf` — копия текущего сниппета с тремя изменениями:

```bash
mkdir -p $SCRATCH/nginx-new
sed -e 's|^# Shared serving rules for course.vladsazonov.com (included from :80 and :443).|# Shared serving rules for vladsazonov.com: card at /, course at /ai-course/.|' \
    -e 's|^# vercel.json: /content/\*.md is served as plain text.|# /ai-course/content/*.md is served as plain text.|' \
    -e 's|location ~\* \^/content/.+\\.md\$ {|location ~* ^/ai-course/content/.+\\.md$ {|' \
    /etc/nginx/snippets/course-site.conf > $SCRATCH/nginx-new/course-site.conf
diff /etc/nginx/snippets/course-site.conf $SCRATCH/nginx-new/course-site.conf
```
Ожидание: diff ровно из трёх изменённых строк: две строки комментариев и `location ~* ^/ai-course/content/.+\.md$ {`. Если `sed` не сработал на какой-то строке, поправить её вручную в файле `$SCRATCH/nginx-new/course-site.conf`.

`$SCRATCH/nginx-new/vladsazonov.com`:

```nginx
# vladsazonov.com: personal card at /, course at /ai-course/.
# http and www redirect to the canonical https apex, preserving path and query.
server {
    listen 80;
    listen [::]:80;
    server_name vladsazonov.com www.vladsazonov.com;

    access_log /var/log/nginx/vladsazonov.access.log;
    error_log  /var/log/nginx/vladsazonov.error.log;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
        default_type "text/plain";
        try_files $uri =404;
    }

    location / {
        return 301 https://vladsazonov.com$request_uri;
    }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name www.vladsazonov.com;

    access_log /var/log/nginx/vladsazonov.access.log;
    error_log  /var/log/nginx/vladsazonov.error.log;

    ssl_certificate     /etc/letsencrypt/live/vladsazonov.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/vladsazonov.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/vladsazonov.com/chain.pem;
    include snippets/tls-params.conf;

    location / {
        return 301 https://vladsazonov.com$request_uri;
    }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name vladsazonov.com;

    access_log /var/log/nginx/vladsazonov.access.log;
    error_log  /var/log/nginx/vladsazonov.error.log;

    ssl_certificate     /etc/letsencrypt/live/vladsazonov.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/vladsazonov.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/vladsazonov.com/chain.pem;
    include snippets/tls-params.conf;

    # Sent only over HTTPS. No includeSubDomains and no preload, so the
    # commitment stays reversible.
    add_header Strict-Transport-Security "max-age=31536000" always;

    include snippets/course-site.conf;
}
```

HSTS и `add_header` из сниппета оказываются на одном (server) уровне, поэтому применяются все вместе — так же, как в текущем конфиге `course.`.

`$SCRATCH/nginx-new/course.vladsazonov.com`:

```nginx
# course.vladsazonov.com moved to vladsazonov.com/ai-course/.
# Permanent redirect preserving path and query; cert kept for renewals.
server {
    listen 80;
    listen [::]:80;
    server_name course.vladsazonov.com;

    access_log /var/log/nginx/course.access.log;
    error_log  /var/log/nginx/course.error.log;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
        default_type "text/plain";
        try_files $uri =404;
    }

    location / {
        return 301 https://vladsazonov.com/ai-course$request_uri;
    }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name course.vladsazonov.com;

    access_log /var/log/nginx/course.access.log;
    error_log  /var/log/nginx/course.error.log;

    ssl_certificate     /etc/letsencrypt/live/course.vladsazonov.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/course.vladsazonov.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/course.vladsazonov.com/chain.pem;
    include snippets/tls-params.conf;

    add_header Strict-Transport-Security "max-age=31536000" always;

    location / {
        return 301 https://vladsazonov.com/ai-course$request_uri;
    }
}
```

- [ ] **Шаг 3: Подтверждение пользователя на выкат**

Показать пользователю `diff` старых и новых конфигов (`diff -u $B/<file> $SCRATCH/nginx-new/<file>` для каждого) и `git -C $WT log --oneline master..personal-card`. Дальше — только после явного «да».

- [ ] **Шаг 4: Переключение (одним блоком, без пауз)**

```bash
set -e
git -C $LIVE status --short   # должен быть пуст
git -C $LIVE merge --ff-only personal-card
cp $SCRATCH/nginx-new/00-course-maps.conf /etc/nginx/conf.d/00-course-maps.conf
cp $SCRATCH/nginx-new/course-site.conf /etc/nginx/snippets/course-site.conf
cp $SCRATCH/nginx-new/vladsazonov.com /etc/nginx/sites-available/vladsazonov.com
cp $SCRATCH/nginx-new/course.vladsazonov.com /etc/nginx/sites-available/course.vladsazonov.com
nginx -t && systemctl reload nginx
```
Ожидание: `syntax is ok`, `test is successful`, reload без ошибок.

Если `nginx -t` упал — немедленный откат:

```bash
cp -a $B/00-course-maps.conf /etc/nginx/conf.d/
cp -a $B/course-site.conf /etc/nginx/snippets/
cp -a $B/vladsazonov.com $B/course.vladsazonov.com /etc/nginx/sites-available/
git -C $LIVE reset --hard master@{1}
nginx -t && systemctl reload nginx
```

- [ ] **Шаг 5: Проверка сервера**

`$TOOLS/check-server.sh`:

```bash
#!/usr/bin/env bash
# Проверяет коды ответа, Location и заголовки после переключения.
fail=0
expect() { # url code [location] [header-regex]
  local out code loc
  out=$(curl -sS -o /dev/null -D - "$1")
  code=$(printf '%s' "$out" | awk 'NR==1{print $2}')
  loc=$(printf '%s' "$out" | awk 'tolower($1)=="location:"{print $2}' | tr -d '\r')
  if [[ "$code" != "$2" ]]; then echo "FAIL $1: code $code != $2"; fail=1; return; fi
  if [[ -n "$3" && "$loc" != "$3" ]]; then echo "FAIL $1: location '$loc' != '$3'"; fail=1; return; fi
  if [[ -n "$4" ]] && ! printf '%s' "$out" | grep -qiE "$4"; then echo "FAIL $1: header /$4/ missing"; fail=1; return; fi
  echo "ok   $1 -> $code ${loc}"
}
S=https://vladsazonov.com
expect "$S/"                                          200 "" "strict-transport-security"
expect "$S/ai-course/"                                200
expect "$S/ai-course"                                 301 "$S/ai-course/"
expect "$S/ai-course/reading/?part=part1_vibecoding"  200
expect "$S/ai-course/presentations/part1/"            200
expect "$S/ai-course/content/lectures.json"           200 "" "cache-control: public, max-age=0"
expect "$S/ai-course/content/part1_foundations.md"    200 "" "content-type: text/plain; charset=utf-8"
expect "$S/assets/avatar.jpg"                         200 "" "immutable"
expect "$S/ai-course/assets/avatar.jpg"               200 "" "immutable"
expect "$S/docs/"                                     403
expect "$S/.git/HEAD"                                 403
expect "https://course.vladsazonov.com/reading/?part=part2_prompt_engineering" 301 "$S/ai-course/reading/?part=part2_prompt_engineering"
expect "https://course.vladsazonov.com/reading/?part=part3_tools_mcp"          301 "$S/ai-course/reading/?part=part3_tools_mcp"
expect "http://course.vladsazonov.com/"                301 "$S/ai-course/"
expect "https://course.vladsazonov.com/"               301 "$S/ai-course/"
expect "http://vladsazonov.com/ai-course/"             301 "$S/ai-course/"
expect "https://www.vladsazonov.com/"                  301 "$S/"
exit $fail
```

```bash
bash $TOOLS/check-server.sh; echo "exit=$?"
```
Ожидание: все строки `ok`, `exit=0`. Если `/ai-course` без слэша отдаёт `Location: http://…` (схема http) — добавить в `course-site.conf` директиву `absolute_redirect off;`, повторить `nginx -t && systemctl reload nginx`, заменить ожидание на `/ai-course/` и перезапустить проверку.

- [ ] **Шаг 6: Пройти глазами живой сайт**

Открыть `https://vladsazonov.com/` и `https://course.vladsazonov.com/reading/?part=part1_vibecoding` (должен перекинуть на конспект). Сообщить пользователю результат.

---

### Задача 6: Пуш и уборка

- [ ] **Шаг 1: Подтверждение пользователя на пуш**

Спросить: «Пушу `master` в `origin` (github.com/SazonovV/ai-education)?» Дальше — только после «да».

- [ ] **Шаг 2: Пуш**

```bash
git -C $LIVE push origin master
```

- [ ] **Шаг 3: Убрать worktree и ветку**

```bash
git -C $LIVE worktree remove $WT
git -C $LIVE branch -d personal-card
git -C $LIVE worktree list
```
Ожидание: в списке только `$LIVE`.

- [ ] **Шаг 4: Остановить визуальный помощник**

```bash
bash /root/.claude/plugins/cache/claude-plugins-official/superpowers/6.4.1/skills/brainstorming/scripts/stop-server.sh $SCRATCH/.superpowers/brainstorm/1194156-1791400108
```
