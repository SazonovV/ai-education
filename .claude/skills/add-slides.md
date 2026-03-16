---
name: add-slides
description: Add slides to an existing reveal.js presentation — reference for all slide patterns, card styles, layouts, and the project's design system
---

# Add slides to a presentation

Use this skill when adding new slides to an existing presentation in `presentations/partN/index.html`.

## Design system

### Colors (CSS variables)

```
--teal:  #37d6c4   — primary accent, default card border
--amber: #f6a44a   — secondary accent
--rose:  #f6757f   — tertiary accent
--green: #53d36f   — success / positive
--red:   #ff6a6a   — danger / negative
```

Cards cycle through colors: teal → amber → rose → green → teal → ...

### Typography

- `h2.section-title` — slide title (teal colored)
- `h3` inside cards — card heading
- `p` / `li` — body text (muted color)
- `p.note` — small gray annotation text
- `blockquote` — highlighted quote with amber left border

## Slide patterns

### Chapter divider (section intro)

Used before each major section to introduce it.

```html
<section>
  <div class="chapter-shell">
    <span class="kicker">Раздел 0N</span>
    <h1>Section Title</h1>
    <p class="note">Short subtitle or description</p>
  </div>
</section>
```

- `chapter-shell` centers content, adds gradient border at top
- `kicker` is a small pill label

### Cards (2 columns)

Most common layout for comparing two ideas or showing pros/cons.

```html
<section>
  <div class="slide-shell">
    <h2 class="section-title">Slide Title</h2>
    <div class="cards c2">
      <div class="card teal fragment">
        <h3>Card heading</h3>
        <p>Card text</p>
      </div>
      <div class="card amber fragment">
        <h3>Card heading</h3>
        <ul>
          <li>Bullet point</li>
          <li>Bullet point</li>
        </ul>
      </div>
    </div>
  </div>
</section>
```

### Cards (3 columns)

For lists of 3+ items (audience types, tool categories, etc.).

```html
<div class="cards c3">
  <div class="card teal fragment"><h3>Title</h3><p>Text</p></div>
  <div class="card amber fragment"><h3>Title</h3><p>Text</p></div>
  <div class="card rose fragment"><h3>Title</h3><p>Text</p></div>
</div>
```

### Grid 2 columns (no card styling)

For two content blocks side by side without card borders.

```html
<div class="grid-2">
  <div class="card teal fragment">...</div>
  <div class="card amber fragment">...</div>
</div>
```

### Table

```html
<div class="table-wrap">
  <table class="mini">
    <tr><th>Column 1</th><th>Column 2</th><th>Column 3</th></tr>
    <tr class="fragment"><td>Data</td><td>Data</td><td>Data</td></tr>
    <tr class="fragment"><td>Data</td><td>Data</td><td>Data</td></tr>
  </table>
</div>
```

### Flow diagram (3 steps with arrows)

For showing a process or pipeline.

```html
<div class="flow">
  <div class="flow-step fragment"><h4>Step 1</h4><p>Description</p></div>
  <div class="arrow fragment">→</div>
  <div class="flow-step fragment"><h4>Step 2</h4><p>Description</p></div>
  <div class="arrow fragment">→</div>
  <div class="flow-step fragment"><h4>Step 3</h4><p>Description</p></div>
</div>
```

### Code block

```html
<pre><code class="language-text">Code content here
Multiple lines supported</code></pre>
```

For syntax highlighting, use appropriate language: `language-bash`, `language-json`, `language-typescript`, `language-python`, `language-markdown`.

### Blockquote

```html
<blockquote class="fragment"><p>Quote text or key takeaway.</p></blockquote>
```

### Checklist

Two-column checklist layout.

```html
<ul class="checklist">
  <li class="fragment">✓ Item one</li>
  <li class="fragment">✓ Item two</li>
  <li class="fragment">✓ Item three</li>
  <li class="fragment">✓ Item four</li>
</ul>
```

## Rules

1. **Always use `fragment` class** on content elements (cards, table rows, blockquotes) for progressive reveal. Exceptions: chapter dividers and static content like code blocks.

2. **Card color rotation**: cycle through `teal → amber → rose → green` to keep slides visually varied. Use `red` only for negative/danger items.

3. **One idea per slide**. If content doesn't fit, split into two slides rather than shrinking.

4. **slide-shell vs chapter-shell**: regular content uses `slide-shell` (left-aligned), section intros use `chapter-shell` (centered).

5. **Code blocks on slides** should be concise — show the essential pattern, not the full implementation. Full code goes in the markdown content file.

6. **Add `p.note`** at the bottom of slides for extra context, caveats, or attributions.

7. **Slide dimensions**: 1360×768. Content should fit without scrolling. If it doesn't fit, split the slide.

8. **Dark code blocks in reading pages**: when editing `reading/` pages, code blocks must have dark background. See memory for CSS rules.
