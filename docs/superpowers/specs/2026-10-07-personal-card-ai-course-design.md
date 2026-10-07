# Визитка vladsazonov.com и переезд курса в /ai-course/ — Design Spec

> Подход: один репозиторий, визитка в корне, курс в подпапке `/ai-course/`, `course.vladsazonov.com` — постоянный редирект.
> Формат визитки: табло отправлений (Solari) в палитре курса.
> Роль страницы: навигация, а не продажа.

---

## Цель и критерии успеха

`vladsazonov.com` сейчас редиректит на курс. Нужно, чтобы по корню домена открывалась личная страница, а курс жил по адресу `vladsazonov.com/ai-course/`.

Визитка одновременно решает три задачи и не продаёт сама:
- **визитка** — кто я, одной-двумя строками;
- **менторство** — путь на GetMentor;
- **портфолио** — выступления с датами и ссылками на материалы.

Успех:
1. `https://vladsazonov.com/` — визитка-табло.
2. `https://vladsazonov.com/ai-course/` — курс, все конспекты и презентации работают.
3. Любая старая ссылка `https://course.vladsazonov.com/<путь>?<query>` отдаёт 301 на `https://vladsazonov.com/ai-course/<путь>?<query>`.
4. Добавить доклад = дописать одну строку в массив.

## Что не входит

- Резюме, цифры результатов, описание тем менторства — всё это есть по ссылкам или не предназначено для публичной страницы.
- Телефон и почта. Единственный контакт — Telegram.
- История карьерного роста.
- Изменения содержания курса (конспекты, слайды).
- Переименование каталога `/var/www/course.vladsazonov.com` и копия `course.vladsazonov.com-snailfish` — не трогаем.

---

## Структура репозитория

```text
.
├── index.html            # визитка-табло (новая)
├── assets/
│   └── avatar.jpg        # копия фото для визитки
├── ai-course/            # git mv из корня
│   ├── index.html
│   ├── reading/
│   ├── presentations/
│   ├── content/
│   └── assets/           # avatar.jpg, og-image.png курса
├── docs/                 # остаётся в корне, закрыт в nginx
├── AGENTS.md
└── README.md
```

`vercel.json` удаляется: сайт отдаёт nginx на собственном сервере, Vercel не используется.

Все ссылки внутри курса относительные (`./reading/`, `../content/`, `../../assets/`) — проверено, переезд в подпапку их не ломает. «На главную» в конспектах и слайдах продолжает вести на главную курса (`/ai-course/`).

---

## Содержание визитки

### Шапка

- Фото `assets/avatar.jpg`.
- **Владислав Сазонов**.
- Роль: «ИТ-лидер в Альфа-Банке · AI в инженерных командах».
- Фраза: «Работаю на стыке управления и ИИ: смотрю, как ИИ меняет устройство команд и процессов».
- Мелкая строка: «Программный комитет трека AI Native, Ontico».

### Табло «Направления»

| Направление (плашки) | Где | Статус | Ссылка | Окно |
|---|---|---|---|---|
| КУРС AGENTIC ENGINEERING | /ai-course/ | ОТКРЫТ | `/ai-course/` | та же вкладка |
| МЕНТОРСТВО | GetMentor | ЗАПИСЬ | `https://getmentor.dev/mentor/vladislav-sazonov-7056` | новая |
| TELEGRAM | «Вы не просили — я рассказал» | LIVE | `https://t.me/SazonovMaybeTalks` | новая |

### Табло «Выступления»

Сортировка — по дате, новые сверху.

| Дата | Выступление | Где | Ссылка | Материалы |
|---|---|---|---|---|
| 2026-10-16 | Ужесточаем harness, или Сага об агентской предсказуемости | ГигаКонф | `https://gigaconf.ru/program` | — |
| 2026-09-10 | Молиться на модель или строить harness | TeamLead Conf Сибирь | `https://teamleadconf.ru/siberia/2026/abstracts/18628` | слайды: `https://disk.yandex.ru/i/qI7HiEa-vA2wdA` |
| 2026-07-03 | Круглый стол «MCP vs CLI, harness, агентские платформы» | Agentic Dev Conf | `https://agenticdevconf.ru/talks/sazonov_vlad` | — |
| 2025-10-21 | Гильдии фронтендеров: что это и нужно ли вообще | FrontendConf | `https://frontendconf.ru/moscow/2025/abstracts/16364` | — |
| 2025-08-28 | Гильдия: место, где разработчик перестаёт быть одиноким кузнецом | MoscowJS 67 | `http://digital.alfabank.ru/events/moscow_JS_67` | — |

Строка ведёт на материалы (видео, иначе слайды), а если их нет — на страницу доклада. Все ссылки выступлений открываются в новой вкладке.

### Статус выступления

Чистая функция `talkStatus(talk, today)`; сравнение по календарной дате в локальном времени посетителя:

| Условие | Статус | Цвет |
|---|---|---|
| `date > today` | СКОРО | акцентный (розовый `--rose`) |
| `date == today` | СЕГОДНЯ | акцентный (розовый `--rose`) |
| `date < today`, есть `video` | ВИДЕО | бирюзовый `--teal` |
| `date < today`, есть `slides` | СЛАЙДЫ | бирюзовый `--teal` |
| `date < today`, материалов нет | ПРОШЁЛ | приглушённый `--muted` |

Цвет всегда дублируется текстом статуса.

### Данные

Направления и выступления — два массива объектов в `<script>` внутри `index.html`, без отдельного JSON и fetch:

```js
const TALKS = [
  { date: "2026-10-16", title: "…", where: "ГигаКонф", url: "…", slides: null, video: null },
];
```

Порядок добавления доклада описывается в `AGENTS.md`.

Табло рендерится скриптом из массивов. Блок `<noscript>` содержит обычный список трёх направлений (они меняются редко), чтобы без JS оставалась навигация. Выступления в `<noscript>` не дублируются — иначе добавление доклада перестаёт быть одной строкой.

---

## Визуальный стиль

Гибрид: механика табло Solari в палитре курса.

- Токены цвета — те же, что у курса (`--bg`, `--line`, `--text`, `--muted`, `--teal`, `--amber`, `--rose`, `--green`), тёмная тема.
- Шрифты: Unbounded (имя), Manrope (текст), моноширинный (JetBrains Mono) для плашек.
- Плашка: отдельный символ на фоне `#1d1916`, янтарный текст, тонкая горизонтальная «щель» посередине.
- Подписи табло: «НАПРАВЛЕНИЯ» и «ВЫСТУПЛЕНИЯ», капслок, разрядка, бирюзовый.
- Фон — радиальные градиенты и зерно, как на главной курса.

## Поведение

### Анимация

- При загрузке каждая плашка прокручивает 4–8 случайных символов и встаёт на нужный. Строки стартуют каскадом сверху вниз; общая длительность ~1,5 с.
- Наведение (или фокус) на строку повторно перелистывает её плашки.
- `prefers-reduced-motion: reduce` — анимации нет, текст сразу на месте.
- Настоящий текст строки находится в DOM всегда (доступен поиску, скринридерам и копированию); анимированные плашки помечены `aria-hidden="true"`.

### Адаптивность

- Десктоп (≥ 720 px): колонки «дата / название / где / статус» у выступлений, «направление / где / статус» у направлений. Длинное название обрезается многоточием, полное — в `title`.
- Телефон (< 720 px): каждая строка в две линии — сверху плашками короткое поле (дата или направление) и статус, снизу обычным текстом название и место. Без горизонтального скролла на ширине 360 px.

### Доступность

- Каждая строка — одна ссылка `<a>` на всю строку, видимый `:focus-visible`.
- Внешние ссылки: `target="_blank" rel="noopener noreferrer"`.
- `lang="ru"`, осмысленный `alt` у фото.

### Мета и аналитика

- `<title>`: «Владислав Сазонов»; `og:title`, `og:description`, `og:url = https://vladsazonov.com/`, `og:type = profile`. `og:image` не задаётся (отдельная картинка — вне объёма).
- Яндекс Метрика: тот же счётчик `107242055`, с `trackLinks: true` — переходы по внешним ссылкам учитываются автоматически.
- Фавикон — та же эмодзи-схема, что у курса, символ ✈️.

---

## Изменения курса

- `ai-course/index.html`: `og:url` → `https://vladsazonov.com/ai-course/`, `og:image` → `https://vladsazonov.com/ai-course/assets/og-image.png`.
- Блок «Автор» на главной курса: имя становится ссылкой на `/` (визитку).
- `README.md` и `AGENTS.md`: новая структура, ключевые URL с префиксом `/ai-course/`, раздел «Добавление новой лекции» с путями `ai-course/…`, новый раздел «Визитка: как добавить выступление».

---

## Сервер (nginx)

Конфиги лежат вне репозитория, в `/etc/nginx`.

### `conf.d/00-course-maps.conf` — карта кэша

```nginx
~*^/ai-course/content/.+\.(md|json)$  "public, max-age=0, must-revalidate";
~*^/(ai-course/)?assets/              "public, max-age=31536000, immutable";
~*\.html$                             "public, max-age=0, must-revalidate";
~*/$                                  "public, max-age=0, must-revalidate";
default                               "public, max-age=300";
```

### `snippets/course-site.conf` — правила отдачи

- `root /var/www/course.vladsazonov.com;` — без изменений.
- Правило `.md` как `text/plain`: `^/ai-course/content/.+\.md$`.
- Запрет дотфайлов и `/docs` — без изменений.
- Комментарий в шапке сниппета обновить: сниппет обслуживает `vladsazonov.com`.

### `sites-enabled/vladsazonov.com`

- :80 `vladsazonov.com`, `www.vladsazonov.com` → ACME-челлендж + `301 https://vladsazonov.com$request_uri`.
- :443 `www.vladsazonov.com` → `301 https://vladsazonov.com$request_uri`.
- :443 `vladsazonov.com` → `include snippets/course-site.conf;`, HSTS `max-age=31536000` (без `includeSubDomains`), существующий сертификат `vladsazonov.com`.

Сертификат `vladsazonov.com` покрывает и `www` (проверено: SAN `vladsazonov.com`, `www.vladsazonov.com`).

### `sites-enabled/course.vladsazonov.com`

- :80 → ACME-челлендж + `301 https://vladsazonov.com/ai-course$request_uri`.
- :443 → `301 https://vladsazonov.com/ai-course$request_uri`; сертификат `course.` сохраняется для продления.

Перед правкой — резервные копии изменяемых файлов nginx рядом в `/root/nginx-backup-2026-10-07/`.

### Порядок выката

Сервер отдаёт рабочую копию репозитория напрямую, поэтому `git mv` в ней мгновенно ломает текущий `course.`. Вся работа идёт в отдельном git worktree, живая копия переключается одним шагом:

1. Создать ветку `personal-card` в worktree вне `/var/www` (в scratchpad). Там — `git mv`, визитка, правки курса и документов; коммиты в ветку.
2. Проверить статусы и ссылки курса скриптами, показать визитку через визуальный помощник (десктоп и 360 px), получить одобрение.
3. Подготовить новые конфиги nginx во временных файлах, сделать резервные копии текущих.
4. В живой копии `git merge --ff-only personal-card`, сразу подменить конфиги, `nginx -t`, `systemctl reload nginx`. Разрыв — секунды.
5. Прогнать проверку сервера (см. ниже).
6. Пуш `master` в `origin` — после подтверждения пользователя.

Откат: вернуть конфиги из резервной копии, в живой копии `git reset --hard 86ceec8`, `systemctl reload nginx`.

---

## Проверка

1. **Логика статусов** — одноразовый node-скрипт в scratchpad, извлекающий `talkStatus` из `index.html`; кейсы: дата завтра, сегодня, вчера; прошедшая со слайдами, с видео, с обоими, без материалов. В репозиторий не попадает.
2. **Ссылки курса** — скрипт проходит по относительным `href`/`src`/`fetch`-константам в `ai-course/**/*.html` и `ai-course/content/lectures.json` и проверяет, что каждый путь указывает на существующий файл.
3. **Сервер** — `nginx -t`, затем `curl` с ожидаемыми кодами:

| Запрос | Ожидание |
|---|---|
| `https://vladsazonov.com/` | 200, визитка |
| `https://vladsazonov.com/ai-course/` | 200, курс |
| `https://vladsazonov.com/ai-course` | 301 → `/ai-course/` |
| `https://vladsazonov.com/ai-course/reading/?part=part1_vibecoding` | 200 |
| `https://vladsazonov.com/ai-course/presentations/part1/` | 200 |
| `https://vladsazonov.com/ai-course/content/lectures.json` | 200, `Cache-Control: … max-age=0` |
| `https://vladsazonov.com/ai-course/content/part1_foundations.md` | 200, `text/plain; charset=utf-8` |
| `https://vladsazonov.com/docs/` | 403 |
| `https://course.vladsazonov.com/reading/?part=part2_prompt_engineering` | 301 → `https://vladsazonov.com/ai-course/reading/?part=part2_prompt_engineering` |
| `http://course.vladsazonov.com/` | 301 → `https://vladsazonov.com/ai-course/` |
| `http://vladsazonov.com/` | 301 → `https://vladsazonov.com/` |
| `https://www.vladsazonov.com/` | 301 → `https://vladsazonov.com/` |

4. **Визуально** — визитка из worktree через визуальный помощник на ширине десктопа и 360 px, до переключения живой копии; одобрение пользователя.
5. **Язык** — тексты визитки проверены по правилам `AGENTS.md` (без кальки с английского).
