# AGENTS.md

## Architecture

This is a static site (no build step, no npm) for an AI Engineering course.

### Reading pages (conspects)

All reading pages are served by a **single dynamic SPA**: `/reading/index.html`.

- Lectures are defined in `/content/lectures.json` (id, title, description, file, slides)
- Markdown content lives in `/content/*.md`
- The reader loads markdown dynamically via `?part=<id>` query parameter
- **Do NOT create separate `/reading/partN/` directories** — they were removed as redundant

Key URLs:
- `/reading/` — index with lecture cards
- `/reading/?part=part1_vibecoding` — lecture 1
- `/reading/?part=part2_prompt_engineering` — lecture 2
- `/reading/?part=part3_tools_mcp` — lecture 3

### Presentations

Reveal.js slides live in `/presentations/partN/index.html`. Each is a standalone HTML file.

### Navigation in reading pages

The top bar shows only two buttons:
- "На главную" — link to site root
- "Следующая лекция →" — link to the next lecture (hidden on the last one)

### Adding a new lecture

1. Create `/content/partN_<slug>.md`
2. Add entry to `/content/lectures.json`
3. Create `/presentations/partN/index.html`
4. Add card to `/index.html`

## Tech stack

- Pure HTML/CSS/JS, no build system
- Reveal.js for presentations
- Marked.js + Highlight.js for markdown rendering
- Deployed on Vercel (`vercel.json`)
