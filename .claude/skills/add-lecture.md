---
name: add-lecture
description: Add a new lecture to the project — creates markdown content, updates lectures.json manifest, scaffolds presentation slides with correct structure
---

# Add a new lecture

Use this skill when adding a new lecture/part to the presentation project.

## Steps

### 1. Create the markdown content file

Create `content/<id>.md` where `<id>` is the lecture identifier (e.g., `part2_advanced`).

Structure:

```markdown
# Title

> Цикл: AI-инструменты для разработки · Часть N
> Аудитория: ...

---

## Содержание

1. Section name
2. Section name
...

---

## 1. Section name

### 1.1. Subsection

Content...

---

## N. Итоги и что дальше

...

---

*Конец конспекта к Части N · Title*
```

### 2. Update the manifest

Add an entry to `content/lectures.json`:

```json
{
  "id": "<id>",
  "title": "Часть N · Short title",
  "description": "Brief description of what the lecture covers",
  "file": "<id>.md",
  "slides": "../presentations/<folder>/"
}
```

After this, the lecture automatically appears in the reading page at `/reading/?part=<id>`.

### 3. Create the presentation

Create `presentations/<folder>/index.html`. Copy the structure from an existing presentation (part0 or part1) including:

- Yandex.Metrika counter (same snippet)
- All CSS variables and styles (copy the full `<style>` block)
- Deck switcher nav — add the new lecture to ALL presentation deck-switchers
- Reveal.js config at the bottom

#### Required slide structure

Every presentation must start with:

```html
<section>
  <div class="chapter-shell">
    <span class="kicker">Agentic Engineering</span>
    <h1 class="hero-title">Lecture Title</h1>
    <p class="hero-sub">Часть N</p>
    <div class="meta-row">
      <span class="pill">Audience description</span>
      <span class="pill">Goal description</span>
    </div>
  </div>
</section>
```

And end with the Telegram promo slide (copy from existing presentations).

For slide patterns, see the `add-slides` skill.

### 4. Update navigation

Update the deck-switcher in ALL existing presentations to include the new lecture:

```html
<div class="deck-switcher-menu">
  <a href="../part0/">Слайды · Часть 0</a>
  <a href="../part1/">Слайды · Часть 1</a>
  <a href="../NEW/">Слайды · Часть N</a>
  <a href="../../reading/?part=part0_vibecoding">Конспект · Часть 0</a>
  <a href="../../reading/?part=part1_foundations">Конспект · Часть 1</a>
  <a href="../../reading/?part=NEW_ID">Конспект · Часть N</a>
  <a href="../../">Главная</a>
</div>
```

Also update `index.html` (home page) to add a card for the new lecture.

### 5. Checklist

- [ ] Markdown file created in `content/`
- [ ] Entry added to `content/lectures.json`
- [ ] Presentation HTML created in `presentations/<folder>/`
- [ ] Deck-switcher updated in ALL presentations (part0, part1, and new)
- [ ] Home page `index.html` updated with new card
- [ ] Reading page works: `/reading/?part=<id>`
- [ ] Presentation loads without errors
