# Табло «Терминал» на визитке — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Блоки «Направления» и «Выступления» на `vladsazonov.com` переделаны по варианту G: карточки «выход на посадку» и тёмный щит вылета. Плашки только на коротких полях, полное название доклада — читаемым текстом.

**Architecture:** Всё по-прежнему в одном `index.html`. Данные лежат в `DESTINATIONS` / `TALKS`, чистые функции — между маркерами `// BEGIN pure` и `// END pure` (их тестирует node-скрипт), рендер DOM и анимация — на чистом JS. Новый шрифт плашек размещаем у себя в `assets/fonts/` и подключаем через новый `fonts-v2.css`. Работа идёт в worktree `$WT` на ветке `card-board-terminal`. Живая копия меняется только мержем после одобрения пользователя.

**Tech Stack:** HTML/CSS/JS без сборки и npm; node 24 и headless Chromium из `~/.cache/ms-playwright` только для проверок в `$TOOLS`; python3 `http.server` для локального показа.

**Spec:** `docs/superpowers/specs/2026-10-08-board-terminal-design.md` (макеты: `docs/superpowers/specs/assets/2026-10-08-board-terminal/G-terminal.dc.html`, `G-phone.dc.html`)

## Global Constraints

- Без build-системы и npm в репозитории. Проверочные скрипты лежат в `$TOOLS` и не коммитятся.
- Все тексты на русском, по правилам `AGENTS.md`. Английские подписи из макета (`Departures`, `Date`, `Destination · Talk`, `Remarks`, `GATE`) — намеренная двуязычность табло, их размечаем `lang="en"`.
- Статусы направлений: ПОСАДКА (мигает) / РЕГИСТРАЦИЯ / В ПОЛЁТЕ. Статусы докладов: ПО РАСПИСАНИЮ (мигает) / ПОСАДКА (мигает) / ВИДЕО / СЛАЙДЫ / ВЫЛЕТЕЛ.
- Коды рейсов AE 101 / MT 202 / TG 303 и выходы A1–A3 остаются. Живые часы «МСК» остаются.
- Токены щита одинаковы в обеих темах: `--board-bg #111111`, `--board-stripe #171717`, `--board-band #0a0a0a`, `--board-line #262626`, `--tile-top #282828`, `--tile-bottom #1e1e1e`, `--tile-fg #f5f1e8`, `--board-text #ece6da`, `--board-muted #a8a295`, `--board-label #9d978b`, `--board-accent #ffc93c`, `--board-edge var(--line)`.
- Статусы: `--ok #3ddc84`; `--soon` — `#ffc93c` в тёмной теме и `#ff5a36` в светлой; `--past #a7a196` в обеих темах.
- Тень щита и карточек: `0 0 0 1px var(--board-edge), 0 22px 44px -28px rgba(0,0,0,.55)`.
- Плашка: фон `linear-gradient(var(--tile-top) 0 50%, var(--tile-bottom) 50% 100%)`, щель 1 px `rgba(0,0,0,.65)`, шрифт `600 Fira Sans Condensed`, кегль равен ширине плашки, радиус 3 px (у выхода 4 px), зазор 2 px, между колонками 12 px.
- Конференция на щите — 20 ячеек. На плашках выводится `board || where`. Молча обрезать нельзя: тест проверяет, что все данные помещаются.
- `/assets` кэшируется как immutable: `fonts-v1.css` не трогаем, создаём `fonts-v2.css`. Курс остаётся на v1.
- `prefers-reduced-motion: reduce`: без перелистывания и мигания, текст сразу на месте, точка горит постоянно.
- Внешние ссылки: `target="_blank" rel="noopener noreferrer"`. Курс открывается в той же вкладке.
- Без горизонтальной прокрутки страницы на 360 px.
- Живая копия `/var/www/course.vladsazonov.com` не меняется до задачи 6, и там — только после явного «да» пользователя.

## Review Focus

1. **Новый доклад с длинной конференцией.** У доклада `where` длиннее 20 символов, а `board` не задан: тест данных должен упасть с понятным сообщением, а не дать молча обрезанное табло. Тест в задаче 2.
2. **Все доклады в прошлом.** Если среди докладов нет ни одного «ПО РАСПИСАНИЮ», ширина колонки статуса равна 7 («ВЫЛЕТЕЛ»/«СЛАЙДЫ»), а не 13. Тест `statusWidth` в задаче 2.
3. **Новый год в данных.** Первый доклад 2027 года создаёт новую полосу года сверху, порядок групп по убыванию. Тест `groupByYear` в задаче 2.
4. **Часы около полуночи и в чужом часовом поясе.** Посетитель из Новосибирска видит московское время, в 00:00 МСК — «00:00», а не «24:00». Тест `moscowHHMM` в задаче 2.
5. **Телефон 360 px со статусом «ПО РАСПИСАНИЮ».** Статус уходит на свою линию, страница не прокручивается по горизонтали. Проверка `overflowX` в задачах 4 и 6.

---

## Переменные окружения исполнителя

```bash
REPO=/var/www/course.vladsazonov.com
TOOLS=/tmp/claude-0/-var-www-course-vladsazonov-com/7ac50e94-b7de-431c-8a6d-708afcbdb603/scratchpad
WT=$TOOLS/wt-g          # worktree ветки card-board-terminal (уже создан)
CHROME=~/.cache/ms-playwright/chromium-1134/chrome-linux/chrome
```

Локальный сервер для показа: `cd $TOOLS && python3 -m http.server 8765 --bind 127.0.0.1` (в фоне). Ветка доступна по `http://127.0.0.1:8765/wt-g/`. Если сервер уже слушает 8765 (`ss -ltn | grep 8765`), второй не запускать.

## Файлы

| Файл | Что меняется |
|---|---|
| `assets/fonts/fira-sans-condensed-600-cyrillic.woff2` | новый, шрифт плашек (кириллица) |
| `assets/fonts/fira-sans-condensed-600-latin.woff2` | новый, шрифт плашек (латиница) |
| `assets/fonts/fonts-v2.css` | новый: всё из v1 + два `@font-face` Fira Sans Condensed |
| `index.html` | подключение v2, токены, CSS карточек и щита, данные, чистые функции, рендер, анимация, часы |
| `AGENTS.md` | раздел «Personal card: adding a talk»: статусы, лимит 20, поле `board`, `gate`/`code` |
| `$TOOLS/test-board-g.mjs` | node-тест чистых функций и данных (не коммитится) |
| `$TOOLS/check.html`, `$TOOLS/shots.sh` | проверка шрифта, горизонтального скролла и скриншоты (не коммитятся) |

---

### Task 1: Шрифт Fira Sans Condensed и инструменты проверки

**Files:**
- Create: `assets/fonts/fira-sans-condensed-600-cyrillic.woff2`, `assets/fonts/fira-sans-condensed-600-latin.woff2`, `assets/fonts/fonts-v2.css`
- Modify: `index.html` (строки `<link rel="preload" …>` и `<link href="./assets/fonts/fonts-v1.css" …>` в `<head>`)
- Create (не в репо): `$TOOLS/check.html`, `$TOOLS/shots.sh`

**Interfaces:**
- Produces: CSS-семейство `"Fira Sans Condensed"` с весом 600; `$TOOLS/shots.sh <dir> <prefix>` — скриншоты и JSON-проверка; `$TOOLS/check.html?w=<px>&src=<path>` — печатает `{"fira":bool,"overflowX":bool,"tiles":n,"unsettled":n}`.

- [ ] **Step 1: Написать страницу проверки и скрипт скриншотов**

`$TOOLS/check.html`:

```html
<!doctype html><meta charset="utf-8"><body style="margin:0"><pre id="out">pending</pre>
<script>
// Usage: check.html?w=<ширина iframe>&src=</путь/к/странице>
const q = new URLSearchParams(location.search);
const f = document.createElement("iframe");
f.style.cssText = `width:${q.get("w") || 1280}px;height:1200px;border:0`;
f.src = q.get("src");
f.onload = () => setTimeout(() => {
  const d = f.contentDocument, r = d.documentElement;
  document.getElementById("out").textContent = JSON.stringify({
    fira: [...d.fonts].some(x => x.family.replace(/"/g, "") === "Fira Sans Condensed" && x.status === "loaded"),
    overflowX: r.scrollWidth > r.clientWidth,
    tiles: d.querySelectorAll(".tile").length,
    unsettled: [...d.querySelectorAll(".tile[data-ch]")].filter(t => t.textContent !== t.dataset.ch).length,
  });
}, 6000);
document.body.prepend(f);
</script>
```

`$TOOLS/shots.sh`:

```bash
#!/usr/bin/env bash
# Usage: shots.sh <каталог ветки внутри $TOOLS, напр. wt-g> <префикс файлов>
# Нужен сервер: cd $TOOLS && python3 -m http.server 8765 --bind 127.0.0.1
set -e
TOOLS="$(cd "$(dirname "$0")" && pwd)"; WT="$1"; P="$2"
C=~/.cache/ms-playwright/chromium-1134/chrome-linux/chrome
FLAGS="--headless=new --no-sandbox --disable-gpu --hide-scrollbars"
for t in light dark; do
  sed "s/<html lang=\"ru\">/<html lang=\"ru\" data-theme=\"$t\">/" "$TOOLS/$WT/index.html" > "$TOOLS/$WT/_$t.html"
  printf '<!doctype html><body style="margin:0;display:flex;gap:20px;background:#888"><iframe src="/%s/_%s.html" style="width:390px;height:2600px;border:0"></iframe><iframe src="/%s/_%s.html" style="width:360px;height:2600px;border:0"></iframe></body>' "$WT" "$t" "$WT" "$t" > "$TOOLS/frame-$t.html"
  $C $FLAGS --window-size=1280,2000 --virtual-time-budget=9000 --screenshot="$TOOLS/$P-desk-$t.png" "http://127.0.0.1:8765/$WT/_$t.html" 2>/dev/null
  $C $FLAGS --window-size=800,2600 --virtual-time-budget=9000 --screenshot="$TOOLS/$P-phone-$t.png" "http://127.0.0.1:8765/frame-$t.html" 2>/dev/null
done
$C $FLAGS --force-prefers-reduced-motion --window-size=1280,2000 --virtual-time-budget=1500 --screenshot="$TOOLS/$P-desk-reduced.png" "http://127.0.0.1:8765/$WT/_light.html" 2>/dev/null
for w in 1280 390 360; do
  echo "$w: $($C $FLAGS --virtual-time-budget=9000 --dump-dom "http://127.0.0.1:8765/check.html?w=$w&src=/$WT/index.html" 2>/dev/null | grep -o '{.*}')"
done
rm -f "$TOOLS/$WT/_light.html" "$TOOLS/$WT/_dark.html"
```

```bash
chmod +x $TOOLS/shots.sh
ss -ltn | grep -q 8765 || (cd $TOOLS && nohup python3 -m http.server 8765 --bind 127.0.0.1 >$TOOLS/http.log 2>&1 &)
```

- [ ] **Step 2: Убедиться, что проверка видит отсутствие шрифта**

Run: `$TOOLS/shots.sh wt-g t1-before | tail -3`
Expected: три строки JSON, во всех `"fira":false`.

- [ ] **Step 3: Скачать woff2 и собрать `fonts-v2.css`**

```bash
cd $WT
UA="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0 Safari/537.36"
curl -s -A "$UA" "https://fonts.googleapis.com/css2?family=Fira+Sans+Condensed:wght@600&display=swap" > $TOOLS/fira.css
python3 - "$TOOLS/fira.css" <<'EOF'
import re, sys, urllib.request
css = open(sys.argv[1]).read()
blocks = re.findall(r"/\* (\S+) \*/\s*(@font-face \{.*?\})", css, re.S)
out = []
for subset, block in blocks:
    if subset not in ("cyrillic", "latin"):
        continue
    url = re.search(r"url\((https://[^)]+)\)", block).group(1)
    name = f"fira-sans-condensed-600-{subset}.woff2"
    urllib.request.urlretrieve(url, f"assets/fonts/{name}")
    out.append(f"/* {subset} */\n" + block.replace(url, name))
assert len(out) == 2, f"expected cyrillic+latin, got {len(out)}"
v1 = open("assets/fonts/fonts-v1.css").read().rstrip("\n")
open("assets/fonts/fonts-v2.css", "w").write(v1 + "\n" + "\n".join(out) + "\n")
EOF
file assets/fonts/fira-sans-condensed-600-*.woff2
grep -c "@font-face" assets/fonts/fonts-v1.css assets/fonts/fonts-v2.css
```

Expected: оба файла `Web Open Font Format (Version 2)`; в v2 на 2 `@font-face` больше, чем в v1; в новых блоках `font-weight: 600` и `src: url(fira-sans-condensed-600-….woff2)`.

- [ ] **Step 4: Подключить v2 в `index.html`**

Заменить строку

```html
<link href="./assets/fonts/fonts-v1.css" rel="stylesheet">
```

на

```html
<link rel="preload" href="./assets/fonts/fira-sans-condensed-600-cyrillic.woff2" as="font" type="font/woff2" crossorigin>
<link href="./assets/fonts/fonts-v2.css" rel="stylesheet">
```

и добавить в `:root` (первый блок переменных) строку:

```css
  --tile-font: "Fira Sans Condensed", "PT Sans Narrow", sans-serif;
```

Шрифт подгружается браузером только при использовании. Чтобы проверка увидела его до задачи 3, временно добавить в конец `<body>` невидимый маркер (удаляется в шаге 6):

```html
<span id="fira-probe" style="font:600 20px var(--tile-font);position:absolute;left:-9999px">ДАТА</span>
```

- [ ] **Step 5: Проверить, что шрифт грузится**

Run: `$TOOLS/shots.sh wt-g t1 | tail -3`
Expected: во всех трёх строках `"fira":true` и `"overflowX":false`.

- [ ] **Step 6: Убрать маркер и закоммитить**

Удалить строку `<span id="fira-probe" …>ДАТА</span>` из `index.html`.

```bash
cd $WT
git add assets/fonts/fira-sans-condensed-600-cyrillic.woff2 assets/fonts/fira-sans-condensed-600-latin.woff2 assets/fonts/fonts-v2.css index.html
git commit -m "card: self-host Fira Sans Condensed 600, fonts-v2.css

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 2: Данные и чистые функции табло

**Files:**
- Modify: `index.html` — массив `DESTINATIONS`, комментарий над `TALKS`, блок `// BEGIN pure … // END pure`
- Create (не в репо): `$TOOLS/test-board-g.mjs`

**Interfaces:**
- Consumes: ничего из задачи 1.
- Produces (внутри маркеров `// BEGIN pure` / `// END pure`):
  - `todayISO(d: Date): string` — без изменений;
  - `talkStatus(talk, today: string): {label: string, tone: "soon"|"ok"|"past", blink: boolean}`;
  - `talkHref(talk): string` — без изменений;
  - `fmtDate(iso): string` — `"16.10.26"`, без изменений (нужен для скринридера);
  - `fmtBoardDate(iso: string): string` — `"16 ОКТ"`;
  - `boardWhere(talk): string` — `talk.board || talk.where`;
  - `cells(text: string, n: number, align?: "left"|"right"): string[]` — ровно `n` элементов, верхний регистр, пробел и пустое место — `""`;
  - `statusWidth(talks, today): number` — длина самого длинного статуса;
  - `groupByYear(sortedTalks): {year: string, talks: Talk[]}[]`;
  - `moscowHHMM(date: Date): string` — `"HH:MM"` по Москве.
- Produces (данные): у каждого `DESTINATIONS[i]` есть `gate`, `code`, `name`, `where`, `status`, `blink`, `url`, `external`.

> Между задачами 2 и 4 старый рендер выступлений получает класс `tone-ok` вместо `tone-done` (зелёный цвет у статуса «СЛАЙДЫ» временно пропадёт). Это нормально: старый рендер удаляется в задаче 4.

- [ ] **Step 1: Написать падающий тест**

`$TOOLS/test-board-g.mjs`:

```js
// Usage: node test-board-g.mjs <path/to/index.html>
import fs from "node:fs";
import assert from "node:assert/strict";

const html = fs.readFileSync(process.argv[2], "utf8");
const m = html.match(/\/\/ BEGIN pure([\s\S]*?)\/\/ END pure/);
assert.ok(m, "markers // BEGIN pure ... // END pure not found");
const { talkStatus, talkHref, fmtDate, fmtBoardDate, boardWhere, cells, statusWidth, groupByYear, moscowHHMM } =
  new Function(m[1] + "\nreturn { talkStatus, talkHref, fmtDate, fmtBoardDate, boardWhere, cells, statusWidth, groupByYear, moscowHHMM };")();

const t = (date, extra = {}) => ({ date, where: "ГигаКонф", url: "https://conf", slides: null, video: null, ...extra });
const today = "2026-10-16";

// --- статусы ---
assert.deepEqual(talkStatus(t("2026-10-17"), today), { label: "ПО РАСПИСАНИЮ", tone: "soon", blink: true });
assert.deepEqual(talkStatus(t("2026-10-16"), today), { label: "ПОСАДКА", tone: "soon", blink: true });
assert.deepEqual(talkStatus(t("2026-10-16", { video: "v" }), today), { label: "ПОСАДКА", tone: "soon", blink: true });
assert.deepEqual(talkStatus(t("2026-10-15"), today), { label: "ВЫЛЕТЕЛ", tone: "past", blink: false });
assert.deepEqual(talkStatus(t("2026-10-15", { slides: "s" }), today), { label: "СЛАЙДЫ", tone: "ok", blink: false });
assert.deepEqual(talkStatus(t("2026-10-15", { video: "v" }), today), { label: "ВИДЕО", tone: "ok", blink: false });
assert.deepEqual(talkStatus(t("2026-10-15", { video: "v", slides: "s" }), today), { label: "ВИДЕО", tone: "ok", blink: false });

// --- даты ---
assert.equal(fmtBoardDate("2026-10-16"), "16 ОКТ");
assert.equal(fmtBoardDate("2025-01-05"), "05 ЯНВ");
assert.equal(fmtBoardDate("2025-05-31"), "31 МАЙ");
assert.equal(fmtBoardDate("2025-12-01"), "01 ДЕК");
assert.equal(fmtDate("2026-10-16"), "16.10.26");

// --- конференция на табло ---
assert.equal(boardWhere(t("2026-01-01", { where: "TeamLead Conf Сибирь" })), "TeamLead Conf Сибирь");
assert.equal(boardWhere(t("2026-01-01", { where: "Очень длинное название конференции", board: "ДЛИННАЯ КОНФ" })), "ДЛИННАЯ КОНФ");

// --- ячейки ---
assert.deepEqual(cells("ab", 4), ["A", "B", "", ""]);
assert.deepEqual(cells("ok", 4, "right"), ["", "", "O", "K"]);
assert.deepEqual(cells("в полёте", 8), ["В", "", "П", "О", "Л", "Ё", "Т", "Е"]);
assert.deepEqual(cells("12:35", 5), ["1", "2", ":", "3", "5"]);
assert.deepEqual(cells("", 2), ["", ""]);
assert.equal(cells("«Вы не просили»", 20).length, 20);

// --- ширина статуса ---
assert.equal(statusWidth([t("2026-10-17"), t("2026-10-01")], today), 13);
assert.equal(statusWidth([t("2026-10-01"), t("2026-10-02", { slides: "s" })], today), 7);
assert.equal(statusWidth([t("2026-10-02", { video: "v" })], today), 5);

// --- группы по годам ---
const sorted = [t("2027-02-01"), t("2026-10-16"), t("2026-07-03"), t("2025-10-21")];
assert.deepEqual(groupByYear(sorted).map(g => [g.year, g.talks.length]), [["2027", 1], ["2026", 2], ["2025", 1]]);
assert.deepEqual(groupByYear([t("2026-01-01")]).map(g => g.year), ["2026"]);
assert.deepEqual(groupByYear([]), []);

// --- московское время ---
assert.equal(moscowHHMM(new Date("2026-10-08T09:35:00Z")), "12:35");
assert.equal(moscowHHMM(new Date("2026-10-08T21:00:00Z")), "00:00");
assert.equal(moscowHHMM(new Date("2026-01-15T05:07:00Z")), "08:07");

assert.equal(talkHref({ url: "u", slides: "s", video: null }), "s");

// --- данные ---
const arr = name => new Function("return " + html.match(new RegExp(`const ${name} = (\\[[\\s\\S]*?\\n\\]);`))[1])();
const TALKS = arr("TALKS");
for (const x of TALKS) {
  const w = boardWhere(x);
  assert.ok([...w].length <= 20, `«${w}» длиннее 20 символов — добавьте докладу поле board`);
}
const DESTINATIONS = arr("DESTINATIONS");
assert.equal(DESTINATIONS.length, 3);
for (const d of DESTINATIONS) {
  assert.match(d.gate, /^[A-Z]\d$/);
  assert.match(d.code, /^[A-Z]{2} \d{3}$/);
  assert.ok(d.name && d.where && d.status && d.url, `incomplete destination ${d.gate}`);
  assert.equal(typeof d.blink, "boolean");
  assert.ok(html.includes(`href="${d.url}"`) || d.url === "./ai-course/", `noscript lacks ${d.url}`);
}
assert.deepEqual(DESTINATIONS.map(d => d.status), ["ПОСАДКА", "РЕГИСТРАЦИЯ", "В ПОЛЁТЕ"]);

console.log("ok");
```

- [ ] **Step 2: Запустить и убедиться, что падает**

Run: `node $TOOLS/test-board-g.mjs $WT/index.html`
Expected: FAIL — `ReferenceError: fmtBoardDate is not defined`.

- [ ] **Step 3: Обновить данные**

Заменить массив `DESTINATIONS` целиком:

```js
// Направления: меняются редко. При правке обновить и <noscript> выше.
// gate — выход (2 символа), code — код рейса; blink — мигающая точка у статуса.
const DESTINATIONS = [
  { gate: "A1", code: "AE 101", name: "Курс Agentic Engineering", where: "vladsazonov.com/ai-course", status: "ПОСАДКА", blink: true, url: "./ai-course/", external: false },
  { gate: "A2", code: "MT 202", name: "Менторство", where: "GetMentor", status: "РЕГИСТРАЦИЯ", blink: false, url: "https://getmentor.dev/mentor/vladislav-sazonov-7056", external: true },
  { gate: "A3", code: "TG 303", name: "Telegram-канал", where: "«Вы не просили — я рассказал»", status: "В ПОЛЁТЕ", blink: false, url: "https://t.me/SazonovMaybeTalks", external: true },
];
```

Комментарий над `TALKS` заменить на:

```js
// Выступления: добавить доклад = дописать строку. Порядок не важен, сортировка по дате.
// where выводится на табло плашками (20 ячеек); если длиннее — добавить board с сокращением.
// slides / video — ссылки на материалы или null; статус считается автоматически.
```

Сам массив `TALKS` не меняется: все `where` короче 20 символов.

- [ ] **Step 4: Заменить блок чистых функций**

Весь текст между `// BEGIN pure` и `// END pure` (включая маркеры) заменить на:

```js
// BEGIN pure
function todayISO(d) {
  const p = n => String(n).padStart(2, "0");
  return `${d.getFullYear()}-${p(d.getMonth() + 1)}-${p(d.getDate())}`;
}

function talkStatus(talk, today) {
  if (talk.date > today) return { label: "ПО РАСПИСАНИЮ", tone: "soon", blink: true };
  if (talk.date === today) return { label: "ПОСАДКА", tone: "soon", blink: true };
  if (talk.video) return { label: "ВИДЕО", tone: "ok", blink: false };
  if (talk.slides) return { label: "СЛАЙДЫ", tone: "ok", blink: false };
  return { label: "ВЫЛЕТЕЛ", tone: "past", blink: false };
}

function talkHref(talk) {
  return talk.video || talk.slides || talk.url;
}

function fmtDate(iso) {
  const [y, m, d] = iso.split("-");
  return `${d}.${m}.${y.slice(2)}`;
}

const MONTHS = ["ЯНВ", "ФЕВ", "МАР", "АПР", "МАЙ", "ИЮН", "ИЮЛ", "АВГ", "СЕН", "ОКТ", "НОЯ", "ДЕК"];
function fmtBoardDate(iso) {
  const [, m, d] = iso.split("-");
  return `${d} ${MONTHS[Number(m) - 1]}`;
}

function boardWhere(talk) {
  return talk.board || talk.where;
}

// Текст по плашкам: ровно n ячеек, пробел и пустое место — "" (пустая плашка).
function cells(text, n, align = "left") {
  const chars = [...String(text).toUpperCase()].slice(0, n).map(ch => (ch === " " ? "" : ch));
  const pad = Array(n - chars.length).fill("");
  return align === "right" ? [...pad, ...chars] : [...chars, ...pad];
}

function statusWidth(talks, today) {
  return Math.max(0, ...talks.map(t => [...talkStatus(t, today).label].length));
}

// Доклады уже отсортированы по убыванию даты.
function groupByYear(sorted) {
  const groups = [];
  for (const t of sorted) {
    const year = t.date.slice(0, 4);
    const last = groups[groups.length - 1];
    if (last && last.year === year) last.talks.push(t);
    else groups.push({ year, talks: [t] });
  }
  return groups;
}

const MSK_TIME = new Intl.DateTimeFormat("ru-RU", { timeZone: "Europe/Moscow", hour: "2-digit", minute: "2-digit" });
function moscowHHMM(date) {
  return MSK_TIME.format(date);
}
// END pure
```

- [ ] **Step 5: Запустить тест**

Run: `node $TOOLS/test-board-g.mjs $WT/index.html`
Expected: `ok`.

Старый рендер направлений читает `d.name` и `d.status` — страница не должна сломаться. Проверка: `$TOOLS/shots.sh wt-g t2 | tail -3` — JSON печатается (страница без ошибок рендерится), `"overflowX":false`.

- [ ] **Step 6: Commit**

```bash
cd $WT && git add index.html
git commit -m "card: airport statuses, gates and flight codes; pure helpers for the terminal board

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 3: Токены, плашки и карточки «Направления»

**Files:**
- Modify: `index.html` — `:root` и оба блока светлой темы, CSS (новые правила перед `/* ---- Переключатель темы ---- */`, новые правила в `@media (max-width: 719px)`), разметка секции `board--dest`, функция `renderDestinations`

**Interfaces:**
- Consumes: `cells()`, `DESTINATIONS` (задача 2), `--tile-font` (задача 1).
- Produces:
  - DOM-хелперы `tileEls(chars: string[]): HTMLElement[]` и `tiles(chars: string[], className?: string): HTMLElement` — `span.tiles[aria-hidden=true]` с детьми `span.tile`; у непустой плашки `dataset.ch`;
  - `link(url, external, className): HTMLAnchorElement`;
  - класс `flip-unit` на каждом элементе, который перелистывается целиком (карточка, строка щита, часы) — используется в задаче 5;
  - CSS-классы `tiles`, `tile`, `tone`, `tone-ok|soon|past|accent`, `blink-dot`; размер плашки задают переменные `--tw` / `--th` на контейнере.

- [ ] **Step 1: Добавить токены**

В `:root` (первый блок) после `--grain: 0.028;` добавить:

```css
  --board-bg: #111111;
  --board-stripe: #171717;
  --board-band: #0a0a0a;
  --board-line: #262626;
  --tile-top: #282828;
  --tile-bottom: #1e1e1e;
  --tile-fg: #f5f1e8;
  --board-text: #ece6da;
  --board-muted: #a8a295;
  --board-label: #9d978b;
  --board-accent: #ffc93c;
  --board-edge: var(--line);
  --board-shadow: 0 0 0 1px var(--board-edge), 0 22px 44px -28px rgba(0, 0, 0, .55);
```

В том же блоке заменить `--past: #7d828c;` на `--past: #a7a196;` (в светлых блоках уже `#a7a196`).

- [ ] **Step 2: Добавить CSS плашек и карточек**

Перед строкой `/* ---- Переключатель темы ---- */` вставить:

```css
/* ---- Плашки табло ---- */
.tiles { display: flex; gap: 2px; flex: none; }
.tile {
  position: relative;
  display: inline-grid;
  place-items: center;
  flex: none;
  width: var(--tw);
  height: var(--th);
  border-radius: 3px;
  background: linear-gradient(var(--tile-top) 0 50%, var(--tile-bottom) 50% 100%);
  color: var(--tile-fg);
  font: 600 var(--tw)/1 var(--tile-font);
}
.tile::after {
  content: "";
  position: absolute;
  left: 0; right: 0; top: 50%;
  height: 1px;
  background: rgba(0, 0, 0, .65);
}
.tone .tile { color: inherit; }
.tone-ok { color: var(--ok); }
.tone-soon { color: var(--soon); }
.tone-past { color: var(--past); }
.tone-accent { color: var(--board-accent); }
.blink-dot {
  width: 8px;
  height: 8px;
  flex: none;
  border-radius: 50%;
  background: currentColor;
  animation: board-blink 1.4s infinite;
}
@keyframes board-blink { 0%, 49% { opacity: 1; } 50%, 100% { opacity: .15; } }
@media (prefers-reduced-motion: reduce) { .blink-dot { animation: none; } }
/* ---- Направления: карточки выходов ---- */
.dest { margin-bottom: 48px; }
.section-head { display: flex; align-items: baseline; flex-wrap: wrap; gap: 7px 14px; margin-bottom: 14px; }
.section-head .board-title { margin: 0; }
.section-sub { font: 500 11px/1 var(--mono); letter-spacing: 1.5px; color: var(--muted); }
.dest-cards {
  list-style: none;
  margin: 0;
  padding: 0;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(260px, 100%), 1fr));
  gap: 16px;
}
.dest-card {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  grid-template-rows: auto 1fr auto;
  grid-template-areas: "gate code" "body body" "foot foot";
  gap: 16px;
  height: 100%;
  min-height: 236px;
  padding: 18px 20px;
  border-radius: 14px;
  background: var(--board-bg);
  color: var(--tile-fg);
  text-decoration: none;
  box-shadow: var(--board-shadow);
}
.dest-card:focus-visible, .talk-row:focus-visible { outline: 2px solid var(--board-accent); outline-offset: 2px; }
.dest-gate { grid-area: gate; display: flex; flex-direction: column; gap: 8px; --tw: 30px; --th: 44px; }
.dest-gate .tile { border-radius: 4px; }
.dest-label { font: 500 10px/1 var(--mono); letter-spacing: 2px; color: var(--board-label); }
.dest-code { grid-area: code; font: 500 13px/1 var(--mono); letter-spacing: 1px; color: var(--board-label); }
.dest-body { grid-area: body; display: flex; flex-direction: column; gap: 6px; }
.dest-name { font: 800 22px/1.2 "Manrope", sans-serif; color: var(--tile-fg); text-wrap: balance; }
.dest-where { font: 500 14px/1.4 "Manrope", sans-serif; color: var(--board-muted); }
.dest-foot {
  grid-area: foot;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding-top: 14px;
  border-top: 1px solid var(--board-line);
  color: var(--ok);
}
.dest-status { display: flex; align-items: center; gap: 8px; --tw: 18px; --th: 28px; }
.dest-arrow { flex: none; }
```

В `@media (max-width: 719px)` перед закрывающей `}` добавить:

```css
  .dest { margin-bottom: 32px; }
  .section-head { flex-direction: column; align-items: flex-start; margin-bottom: 12px; }
  .section-sub { font-size: 10px; }
  .dest-cards { gap: 12px; }
  .dest-card {
    grid-template-columns: auto minmax(0, 1fr);
    grid-template-rows: auto auto auto;
    grid-template-areas: "gate code" "gate body" "foot foot";
    gap: 4px 14px;
    min-height: 0;
    padding: 14px 16px;
    border-radius: 12px;
  }
  .dest-gate { gap: 6px; --tw: 26px; --th: 38px; }
  .dest-label { font-size: 9px; letter-spacing: 1.5px; }
  .dest-label-en { display: none; }
  .dest-code { font-size: 11px; }
  .dest-name { font-size: 18px; line-height: 1.25; }
  .dest-where { font-size: 13px; line-height: 1.35; }
  .dest-foot { margin-top: 8px; padding-top: 12px; gap: 10px; }
  .dest-status { gap: 7px; --tw: 16px; --th: 25px; }
  .blink-dot { width: 7px; height: 7px; }
```

- [ ] **Step 3: Заменить разметку секции направлений**

Строки от `<section class="board board--dest" aria-labelledby="dest-title">` до первого `</section>` после неё заменить на:

```html
  <section class="dest" aria-labelledby="dest-title">
    <div class="section-head">
      <h2 class="board-title" id="dest-title">Направления</h2>
      <span class="section-sub" aria-hidden="true">ВЫХОД НА ПОСАДКУ · <span lang="en">GATES</span></span>
    </div>
    <ul class="dest-cards" id="destinations"></ul>
    <noscript>
      <ul class="plain">
        <li><a href="./ai-course/">Курс Agentic Engineering</a></li>
        <li><a href="https://getmentor.dev/mentor/vladislav-sazonov-7056" target="_blank" rel="noopener noreferrer">Менторство на GetMentor</a></li>
        <li><a href="https://t.me/SazonovMaybeTalks" target="_blank" rel="noopener noreferrer">Telegram-канал «Вы не просили — я рассказал»</a></li>
      </ul>
    </noscript>
  </section>
```

- [ ] **Step 4: Добавить DOM-хелперы и рендер карточек**

Сразу после функции `el(...)` вставить:

```js
function tileEls(chars) {
  return chars.map(ch => {
    const tile = el("span", "tile", ch);
    if (ch) tile.dataset.ch = ch;
    return tile;
  });
}

// Плашки скрыты от скринридеров: настоящий текст лежит рядом в .vh или виден обычным текстом.
function tiles(chars, className) {
  const wrap = el("span", className ? `tiles ${className}` : "tiles");
  wrap.setAttribute("aria-hidden", "true");
  wrap.append(...tileEls(chars));
  return wrap;
}

function link(url, external, className) {
  const a = el("a", className);
  a.href = url;
  if (external) { a.target = "_blank"; a.rel = "noopener noreferrer"; }
  return a;
}

const ARROW_SVG = '<svg class="dest-arrow" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M5 12h14"/><path d="m13 6 6 6-6 6"/></svg>';
```

Функцию `renderDestinations` заменить целиком:

```js
function renderDestinations(list) {
  const ul = document.getElementById("destinations");
  for (const d of list) {
    const a = link(d.url, d.external, "dest-card flip-unit");
    const gate = el("span", "dest-gate");
    const label = el("span", "dest-label", "ВЫХОД");
    const labelEn = el("span", "dest-label-en", " · GATE");
    labelEn.lang = "en";
    label.append(labelEn);
    label.setAttribute("aria-hidden", "true");
    gate.append(label, tiles(cells(d.gate, 2), "tone tone-accent"));
    const code = el("span", "dest-code", d.code);
    code.setAttribute("aria-hidden", "true");
    const body = el("span", "dest-body");
    body.append(el("span", "vh", `Выход ${d.gate}, рейс ${d.code}.`), el("span", "dest-name", d.name), el("span", "dest-where", d.where));
    const foot = el("span", "dest-foot");
    const status = el("span", "dest-status");
    if (d.blink) status.append(el("span", "blink-dot"));
    status.append(tiles(cells(d.status, [...d.status].length), "tone tone-ok"), el("span", "vh", `Статус: ${d.status}.`));
    foot.append(status);
    foot.insertAdjacentHTML("beforeend", ARROW_SVG);
    a.append(gate, code, body, foot);
    const li = el("li");
    li.append(a);
    ul.append(li);
  }
}
```

`rowLink` пока не удалять — им ещё пользуется старый `renderTalks` (удаляется в задаче 4).

- [ ] **Step 5: Проверить**

Run: `node $TOOLS/test-board-g.mjs $WT/index.html && $TOOLS/shots.sh wt-g t3`
Expected:
- тест `ok`;
- в JSON на 1280/390/360 `"overflowX":false`, `"fira":true`;
- на `t3-desk-light.png` и `t3-desk-dark.png` три тёмные карточки в ряд: жёлтые плашки A1/A2/A3, код рейса справа, название крупно, «где» серым, внизу зелёные плашки статуса и стрелка; у курса точка перед ПОСАДКА;
- на `t3-phone-*.png` карточки одна под другой, выход слева, код и название справа, нет подписи `· GATE`;
- сверить с `docs/superpowers/specs/assets/2026-10-08-board-terminal/G-terminal.dc.html` и `G-phone.dc.html` (верхняя секция) по размерам и цветам.

Посмотреть скриншоты инструментом Read.

- [ ] **Step 6: Commit**

```bash
cd $WT && git add index.html
git commit -m "card: destinations as boarding gate cards; board tokens and tiles

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 4: Щит «Выступления»

**Files:**
- Modify: `index.html` — разметка секции `board--talks`, CSS (удалить старые правила табло, добавить правила щита), функция `renderTalks`, удалить `flapCell` и `rowLink`

**Interfaces:**
- Consumes: `tiles`, `tileEls`, `link`, `cells`, `fmtBoardDate`, `boardWhere`, `statusWidth`, `groupByYear`, `talkStatus`, `talkHref`, `fmtDate`, `moscowHHMM` (задачи 2–3).
- Produces: `span#clock.tiles.tone.tone-accent.flip-unit` с 5 плашками; строки `a.talk-row.flip-unit` — используются в задаче 5.

- [ ] **Step 1: Заменить разметку секции выступлений**

Секцию `<section class="board board--talks" …> … </section>` заменить на:

```html
  <section class="departures" aria-labelledby="talks-title">
    <div class="dep-head">
      <div class="dep-title">
        <svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M2 22h20"/><path d="M6.36 17.4 4 17l-2-4 1.1-.55a2 2 0 0 1 1.8 0l.17.1a2 2 0 0 0 1.8 0L8 12 5 6l.9-.45a2 2 0 0 1 2.09.2l4.02 3a2 2 0 0 0 2.1.2l4.19-2.06a2.41 2.41 0 0 1 1.73-.17L21 7a1.4 1.4 0 0 1 .87 1.99l-.38.76c-.23.46-.6.84-1.07 1.08L7.58 17.2a2 2 0 0 1-1.22.18Z"/></svg>
        <span class="dep-title-text">
          <h2 id="talks-title">Выступления</h2>
          <span class="dep-sub" lang="en" aria-hidden="true">Departures</span>
        </span>
      </div>
      <div class="dep-clock" aria-hidden="true">
        <span class="dep-clock-label">МСК</span>
        <span class="tiles tone tone-accent flip-unit" id="clock"></span>
      </div>
    </div>
    <div class="dep-cols" aria-hidden="true">
      <span class="dep-col dep-col--date"><b>ДАТА</b><i lang="en">Date</i></span>
      <span class="dep-col"><b>НАПРАВЛЕНИЕ · ДОКЛАД</b><i lang="en">Destination · Talk</i></span>
      <span class="dep-col dep-col--status"><b>СТАТУС</b><i lang="en">Remarks</i></span>
    </div>
    <ul class="dep-rows" id="talks"></ul>
  </section>
```

- [ ] **Step 2: Удалить старый CSS табло и добавить CSS щита**

Удалить блок от строки `/* ---- Табло ---- */` до строки `/* ---- Плашки табло ---- */` (не включая её) и вставить на его место:

```css
/* ---- Подписи секций ---- */
.board-title {
  font: 700 12px/1 "Manrope", sans-serif;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--muted);
  margin: 0 0 10px;
}
.plain { line-height: 2; }
.plain a { color: var(--accent); }
```

Перед строкой `/* ---- Переключатель темы ---- */` добавить:

```css
/* ---- Выступления: щит вылета ---- */
.departures {
  margin-bottom: 36px;
  background: var(--board-bg);
  color: var(--tile-fg);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: var(--board-shadow);
  --tw: 20px;
  --th: 32px;
}
.dep-head { display: flex; justify-content: space-between; align-items: center; gap: 16px; padding: 16px 20px; }
.dep-title { display: flex; align-items: center; gap: 12px; color: var(--board-accent); }
.dep-title-text { display: flex; align-items: center; gap: 12px; }
.dep-title h2 { margin: 0; font: 800 22px/1 "Manrope", sans-serif; color: var(--tile-fg); }
.dep-sub { font: 500 15px/1 "Manrope", sans-serif; color: var(--board-label); }
.dep-clock { display: flex; align-items: center; gap: 10px; }
.dep-clock-label { font: 500 11px/1 var(--mono); letter-spacing: 1.5px; color: var(--board-label); }
.dep-cols { display: flex; align-items: flex-end; gap: 12px; padding: 10px 20px; background: var(--board-band); }
.dep-col { display: flex; flex-direction: column; gap: 3px; }
.dep-col--date { width: calc(6 * var(--tw) + 5 * 2px); }
.dep-col--status { margin-left: auto; align-items: flex-end; }
.dep-col b { font: 800 11px/1 "Manrope", sans-serif; letter-spacing: .1em; color: var(--board-accent); }
.dep-col i { font: 500 11px/1 "Manrope", sans-serif; font-style: normal; color: #8f897e; }
.dep-rows { list-style: none; margin: 0; padding: 0; }
.dep-year {
  padding: 8px 20px;
  background: var(--board-band);
  border-top: 1px solid #1f1f1f;
  font: 800 12px/1 "Manrope", sans-serif;
  letter-spacing: .14em;
  color: var(--board-accent);
}
.talk-row {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 12px 20px 14px;
  background: var(--board-bg);
  color: var(--tile-fg);
  text-decoration: none;
}
.talk-row--alt { background: var(--board-stripe); }
.talk-line { display: flex; flex-wrap: wrap; align-items: center; gap: 8px 12px; }
.talk-status { margin-left: auto; display: flex; align-items: center; gap: 8px; }
.talk-title {
  padding-left: calc(6 * var(--tw) + 5 * 2px + 12px);
  font: 600 16px/1.4 "Manrope", sans-serif;
  color: var(--board-text);
  text-wrap: pretty;
}
```

В `@media (max-width: 719px)` удалить строки от `.board-head { display: none; }` до `.flap { font-size: 13px; }` включительно (старые правила табло: `.board-head`, `:root { --fw…}`, `.board--dest .row…`, `.c-key`, `.c-status`, `.c-title`, `.c-where`, `.flap`) и перед закрывающей `}` добавить:

```css
  .departures { border-radius: 14px; --tw: min(14px, calc((100vw - 98px) / 20)); --th: calc(var(--tw) * 1.64); }
  .dep-head { padding: 14px; gap: 10px; }
  .dep-title { gap: 10px; }
  .dep-title svg { width: 22px; height: 22px; }
  .dep-title-text { flex-direction: column; align-items: flex-start; gap: 3px; }
  .dep-title h2 { font-size: 18px; }
  .dep-sub { font-size: 12px; }
  .dep-clock { --tw: 15px; --th: 24px; }
  .dep-clock-label { display: none; }
  .dep-cols { display: none; }
  .dep-year { padding: 8px 14px; font-size: 11px; }
  .talk-row { padding: 12px 14px 14px; }
  .talk-where { order: 3; flex-basis: 100%; }
  .talk-title { padding-left: 0; font-size: 15px; }
```

(`100vw - 98px`: 32 px отступы страницы + 28 px отступы строки + 19 зазоров по 2 px.)

Убедиться, что из `:root` можно удалить `--fw`, `--fh`, `--fgap`: `grep -n "var(--f[wh]\|var(--fgap)" index.html` должен ничего не найти; тогда удалить эти три строки.

- [ ] **Step 3: Заменить рендер выступлений**

Удалить функции `flapCell` (с комментарием над ней) и `rowLink`. Функцию `renderTalks` и последние две строки вызова заменить на:

```js
function renderTalks(list, today) {
  const ul = document.getElementById("talks");
  const sorted = [...list].sort((a, b) => b.date.localeCompare(a.date));
  const statusN = statusWidth(sorted, today);
  let n = 0;
  for (const group of groupByYear(sorted)) {
    ul.append(el("li", "dep-year", group.year));
    for (const t of group.talks) {
      const s = talkStatus(t, today);
      const a = link(talkHref(t), true, `talk-row flip-unit${n++ % 2 ? " talk-row--alt" : ""}`);
      a.title = `${t.title} — ${t.where}`;
      const line = el("span", "talk-line");
      const status = el("span", `talk-status tone tone-${s.tone}`);
      if (s.blink) status.append(el("span", "blink-dot"));
      status.append(tiles(cells(s.label, statusN, "right")));
      line.append(tiles(cells(fmtBoardDate(t.date), 6), "talk-date"), tiles(cells(boardWhere(t), 20), "talk-where"), status);
      a.append(
        line,
        el("span", "vh", `${fmtDate(t.date)}, ${t.where}.`),
        el("span", "talk-title", t.title),
        el("span", "vh", `Статус: ${s.label}.`),
      );
      const li = el("li");
      li.append(a);
      ul.append(li);
    }
  }
}

function renderClock(now) {
  document.getElementById("clock").replaceChildren(...tileEls(cells(moscowHHMM(now), 5)));
}

renderDestinations(DESTINATIONS);
renderTalks(TALKS, todayISO(new Date()));
renderClock(new Date());
```

Старый `boardIntro` ищет `.row` и `.flaps` — после этой задачи он ничего не находит и ничего не ломает. Анимация возвращается в задаче 5.

- [ ] **Step 4: Проверить**

Run: `node $TOOLS/test-board-g.mjs $WT/index.html && $TOOLS/shots.sh wt-g t4`
Expected:
- тест `ok`;
- JSON: `"overflowX":false` на 1280, 390 и 360;
- `t4-desk-*`: тёмный щит, шапка с жёлтым самолётом, «Выступления», серое «Departures», справа «МСК» и часы плашками; полоса подписей колонок; полосы годов 2026 и 2025; строки чередуют фон; в строке `16 ОКТ` · `ГИГАКОНФ` в 20 ячейках · жёлтые/красные `ПО РАСПИСАНИЮ` с точкой справа; ниже полное название под колонкой конференции;
- `t4-phone-*`: дата слева, статус справа; на 360 px «ПО РАСПИСАНИЮ» может уйти на отдельную линию, прижатую вправо; конференция — 20 ячеек на всю ширину; название ниже;
- сверить с макетами G (нижняя секция).

Также проверить доступный текст:

```bash
$CHROME --headless=new --no-sandbox --disable-gpu --virtual-time-budget=6000 --dump-dom http://127.0.0.1:8765/wt-g/index.html 2>/dev/null | grep -o 'class="vh">[^<]*' | head -8
```

Expected: `Выход A1, рейс AE 101.`, `Статус: ПОСАДКА.`, …, `16.10.26, ГигаКонф.`, `Статус: ПО РАСПИСАНИЮ.`.

- [ ] **Step 5: Commit**

```bash
cd $WT && git add index.html
git commit -m "card: talks as a departures board — year groups, tiles for date/venue/status, readable titles

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 5: Перелистывание, мигание и часы

**Files:**
- Modify: `index.html` — второй `<script>` (анимация)

**Interfaces:**
- Consumes: `.flip-unit`, `.tile[data-ch]`, `#clock`, `moscowHHMM`, `cells` (задачи 2–4).
- Produces: `flipUnit(unit, delay)`, `boardIntro()`, `startClock()`.

- [ ] **Step 1: Убедиться, что сейчас нет каскада**

Run: `$CHROME --headless=new --no-sandbox --disable-gpu --virtual-time-budget=300 --dump-dom http://127.0.0.1:8765/wt-g/index.html 2>/dev/null | grep -c 'class="tile" data-ch="[^"]*"></span>'`
Expected: `0` — через 300 мс все плашки уже показывают итоговые буквы (перелистывания нет). После шага 2 то же число должно стать больше 0: на старте каскада плашки ещё пустые.

- [ ] **Step 2: Заменить скрипт анимации**

Второй `<script>` (от `// Перелистывание плашек…` до `boardIntro();`) заменить на:

```html
<script>
// Перелистывание плашек: каждая прокручивает несколько случайных символов и встаёт на свой.
const FLAP_CHARSET = "АБВГДЕЖЗИКЛМНОПРСТУФХЦЧШЭЮЯABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
const FLAP_TICK_MS = 126;
const ROW_STAGGER_MS = 266;
const CHAR_STAGGER_MS = 31;
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
      flap.textContent = flap.dataset.ch || target;
    }
  }, delay);
}

// Карточка, строка щита или часы перелистываются целиком.
function flipUnit(unit, delay) {
  if (reduceMotion.matches || unit.dataset.flipping) return;
  unit.dataset.flipping = "1";
  let longest = 0;
  unit.querySelectorAll(".tile[data-ch]").forEach((tile, i) => {
    const d = delay + i * CHAR_STAGGER_MS;
    flipFlap(tile, tile.dataset.ch, d);
    longest = Math.max(longest, d + 9 * FLAP_TICK_MS);
  });
  setTimeout(() => { delete unit.dataset.flipping; }, longest);
}

// Каскад сверху вниз: карточки направлений, часы, строки щита.
function boardIntro() {
  const units = document.querySelectorAll(".flip-unit");
  if (!reduceMotion.matches) {
    // Прячем итоговый текст до начала перелистывания, чтобы не было вспышки.
    units.forEach(u => u.querySelectorAll(".tile[data-ch]").forEach(t => { t.textContent = ""; }));
  }
  units.forEach((u, i) => {
    flipUnit(u, i * ROW_STAGGER_MS);
    if (u.matches("a")) {
      u.addEventListener("mouseenter", () => flipUnit(u, 0));
      u.addEventListener("focus", () => flipUnit(u, 0));
    }
  });
}

// Часы МСК: раз в минуту, перелистываются только изменившиеся цифры.
function startClock() {
  const clock = document.getElementById("clock");
  const tick = () => {
    cells(moscowHHMM(new Date()), 5).forEach((ch, i) => {
      const tile = clock.children[i];
      if (tile.dataset.ch === ch) return;
      tile.dataset.ch = ch;
      if (reduceMotion.matches) tile.textContent = ch;
      else flipFlap(tile, ch, i * CHAR_STAGGER_MS);
    });
    setTimeout(tick, 60000 - (Date.now() % 60000) + 50);
  };
  setTimeout(tick, 60000 - (Date.now() % 60000) + 50);
}

boardIntro();
startClock();
</script>
```

(`flipFlap` в конце ставит `flap.dataset.ch || target`: если за время перелистывания часы успели смениться, плашка встанет на новую цифру, а не на старую.)

- [ ] **Step 3: Проверить каскад и итоговое состояние**

Run: `$CHROME --headless=new --no-sandbox --disable-gpu --virtual-time-budget=300 --dump-dom http://127.0.0.1:8765/wt-g/index.html 2>/dev/null | grep -c 'class="tile" data-ch="[^"]*"></span>'`
Expected: число больше 0 (каскад ещё идёт).

Run: `$TOOLS/shots.sh wt-g t5`
Expected:
- во всех JSON `"unsettled":0` (через 6 с все плашки встали на свои буквы), `"overflowX":false`, `"fira":true`;
- `t5-desk-reduced.png` (снят через 1,5 с с `--force-prefers-reduced-motion`): все буквы уже на месте.

Проверить reduced motion отдельно: `$CHROME --headless=new --no-sandbox --disable-gpu --force-prefers-reduced-motion --virtual-time-budget=300 --dump-dom http://127.0.0.1:8765/wt-g/index.html 2>/dev/null | grep -c 'class="tile" data-ch="[^"]*"></span>'` → `0`.

Часы: в DOM-дампе `id="clock"` содержит 5 плашек, их текст совпадает с `TZ=Europe/Moscow date +%H:%M` (±1 минута).

- [ ] **Step 4: Commit**

```bash
cd $WT && git add index.html
git commit -m "card: flip cascade for cards, board rows and Moscow clock; reduced-motion safe

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 6: Документация, итоговая проверка, показ пользователю и раскатка

**Files:**
- Modify: `AGENTS.md` — раздел «Personal card: adding a talk»

**Interfaces:**
- Consumes: всё из задач 1–5.

- [ ] **Step 1: Обновить AGENTS.md**

В разделе «Personal card: adding a talk» заменить текст от строки ``Talks live in the `TALKS` array…`` до конца предложения ``…update the `<noscript>` list too.`` (фраза `No phone or e-mail on the card; Telegram is the only contact.` остаётся отдельной строкой после замены) на:

```markdown
Talks live in the `TALKS` array in `/index.html`. Add one object:

`{ date: "YYYY-MM-DD", title: "…", where: "Conference", url: "talk page", slides: null, video: null }`

Order does not matter (sorted by date, grouped by year). `where` is shown on split-flap tiles in 20 cells: if it is longer, add `board: "SHORT NAME"` (≤ 20 characters) — the full `where` stays for screen readers. Status is computed from the date and materials: ПО РАСПИСАНИЮ (future) / ПОСАДКА (today) / ВИДЕО / СЛАЙДЫ / ВЫЛЕТЕЛ (past, no materials); when slides or video appear, fill `slides` / `video`. Destinations live in `DESTINATIONS` (`gate`, `code`, `name`, `where`, `status`, `blink`) — when changing them, update the `<noscript>` list too.
```

Затем `grep -n "СКОРО\|ПРОШЁЛ" AGENTS.md` — ничего не должно найтись. В разделе «Tech stack» строку `- Web fonts are self-hosted in /assets/fonts/fonts-v1.css …` заменить на:

```markdown
- Web fonts are self-hosted in `/assets/fonts/` (cyrillic + latin subsets). The card uses `fonts-v2.css` (adds Fira Sans Condensed for the split-flap tiles), the course stays on `fonts-v1.css`; /assets is cached immutable — on change, create a new `fonts-vN.css`.
```

- [ ] **Step 2: Итоговая проверка**

```bash
node $TOOLS/test-board-g.mjs $WT/index.html
$TOOLS/shots.sh wt-g final
cd $WT && git status --short && git diff master --stat
grep -n "СКОРО\|ПРОШЁЛ\|flapCell\|rowLink\|board--\|\.flaps" index.html
```

Expected:
- тест `ok`;
- JSON на 1280/390/360: `"fira":true`, `"overflowX":false`, `"unsettled":0`;
- grep ничего не находит;
- изменены только `index.html`, `AGENTS.md`, `assets/fonts/fonts-v2.css`, два woff2, плюс спека и план в `docs/`.

Просмотреть `final-desk-light.png`, `final-desk-dark.png`, `final-phone-light.png`, `final-phone-dark.png`, `final-desk-reduced.png` и сверить с макетами G. Проверить язык новых русских строк по правилам `AGENTS.md`: «Выход A1, рейс AE 101.», «Статус: …», «ВЫХОД НА ПОСАДКУ», «НАПРАВЛЕНИЕ · ДОКЛАД».

- [ ] **Step 3: Commit**

```bash
cd $WT && git add AGENTS.md
git commit -m "docs: AGENTS.md — terminal board statuses, board/where limit, fonts-v2

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

- [ ] **Step 4: Показать пользователю и дождаться «да»**

Сообщить пользователю: ветка `card-board-terminal`, превью через `ssh -L 8765:127.0.0.1:8765 <сервер>` и `http://localhost:8765/wt-g/`, что проверено и что не проверялось (живое наведение, реальная смена минуты, реальные устройства). **Остановиться и ждать явного одобрения.** Без него не мержить.

- [ ] **Step 5: Раскатка (только после «да»)**

```bash
cd $REPO && git status --short          # должно быть пусто
git merge --ff-only card-board-terminal
curl -s -o /dev/null -w "%{http_code}\n" https://vladsazonov.com/assets/fonts/fonts-v2.css
curl -s https://vladsazonov.com/ | grep -c "fonts-v2.css"
```

Expected: fast-forward без конфликтов; `200`; `1`. Пуш в `origin` — отдельным вопросом пользователю (см. память про deploy key `github-ai-education`).
