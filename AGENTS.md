# AGENTS.md

## Architecture

Static site (no build step, no npm) served by nginx on our own server (not Vercel).

- `/` — personal card (`/index.html`): a split-flap departures board linking to the course, mentoring, Telegram and talks.
- `/ai-course/` — the AI Engineering course (everything below lives under `ai-course/`).
- `course.vladsazonov.com/*` permanently redirects (301) to `vladsazonov.com/ai-course/*`.

All links inside `ai-course/` must stay relative — the course must not assume it is served from the site root.

### Reading pages (conspects)

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

### Presentations

Reveal.js slides live in `/ai-course/presentations/partN/index.html`. Each is a standalone HTML file.

### Navigation in reading pages

The top bar shows only two buttons:
- "На главную" — link to the course home (/ai-course/)
- "Следующая лекция →" — link to the next lecture (hidden on the last one)

### Adding a new lecture

1. Create `/ai-course/content/partN_<slug>.md`
2. Add entry to `/ai-course/content/lectures.json`
3. Create `/ai-course/presentations/partN/index.html`
4. Add card to `/ai-course/index.html`

### Personal card: adding a talk

Talks live in the `TALKS` array in `/index.html`. Add one object:

`{ date: "YYYY-MM-DD", title: "…", where: "Conference", url: "talk page", slides: null, video: null }`

Order does not matter (sorted by date). Status (СКОРО / СЕГОДНЯ / ВИДЕО / СЛАЙДЫ / ПРОШЁЛ) is computed from the date and materials; when slides or video appear, fill `slides` / `video`. Destinations live in `DESTINATIONS` — when changing them, update the `<noscript>` list too. No phone or e-mail on the card; Telegram is the only contact.

## Linguistic rules for Russian content

All conspects and slides are in Russian. After writing or editing text, verify there are no direct calques from English. Common violations:

- **Untranslated English nouns/phrases** in Russian sentences: `evidence`, `ongoing tool calls`, `bite-sized`, `per step`. Translate or adapt.
- **Literal translation of idioms**: `срезы углов` (cut corners) → `упрощения`; `минимальный пол` (minimum floor) → `базовый минимум`; `первый класс` (first-class) → `полноценная поддержка`.
- **Hybrid words with English roots + Russian suffixes** that sound unnatural: `хэндофф-артефакты` → `артефакты передачи контекста`; `правило sizing'а` → `правило оценки масштаба`.
- **Slide titles entirely in English** without Russian adaptation: `Verification before completion` → `Проверка перед завершением`.

Acceptable English in Russian text: established tech terms (`hooks`, `pipeline`, `CI/CD`, `TDD`, `spec`, `plan`, `scope`, `workaround`, tool names, code identifiers).

**Rule of thumb:** if a native Russian speaker would pause and mentally translate the phrase, rewrite it.

## Tech stack

- Pure HTML/CSS/JS, no build system
- Reveal.js for presentations
- Marked.js + Highlight.js for markdown rendering
- Served by nginx from this repo's working copy (configs in /etc/nginx, outside the repo)
