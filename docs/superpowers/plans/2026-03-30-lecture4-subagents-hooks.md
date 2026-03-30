# Lecture 4: Субагенты, оркестрация и Hooks — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create lecture 4 content — markdown conspect and Reveal.js slides — covering subagents, orchestration patterns, and hooks across Claude Code, Roo Code, Kilo Code, and OpenCode.

**Architecture:** Static site, no build step. Conspect is a markdown file loaded by the SPA reader at `/reading/`. Slides are a standalone Reveal.js HTML file. Both follow the exact patterns of lectures 1-3.

**Tech Stack:** Markdown, HTML/CSS/JS, Reveal.js 5.2.1, Manrope + Unbounded fonts.

**Spec:** `docs/superpowers/specs/2026-03-30-lecture4-subagents-hooks-design.md`

---

## File Map

| Action | File | Responsibility |
|--------|------|---------------|
| Create | `/content/part4_subagents_hooks.md` | Markdown conspect (~600-700 lines) |
| Create | `/presentations/part4/index.html` | Reveal.js slides (~35-45 slides, ~1800 lines) |
| Modify | `/content/lectures.json` | Add lecture 4 entry |
| Modify | `/presentations/part3/index.html` | Add part4 link to closing slide |
| Modify | `/index.html:344-348` | Move lecture 4 card from "Планируются" to "В работе", add links |

---

### Task 1: Update lectures.json

**Files:**
- Modify: `/content/lectures.json`

- [ ] **Step 1: Add lecture 4 entry to lectures.json**

Add as 4th element in the JSON array:

```json
{
  "id": "part4_subagents_hooks",
  "title": "Лекция 4 · Субагенты, оркестрация и Hooks",
  "description": "Делегирование субагентам, паттерны оркестрации, hooks и автоматизация в Claude Code, Roo Code, Kilo Code, OpenCode",
  "file": "part4_subagents_hooks.md",
  "slides": "../presentations/part4/"
}
```

- [ ] **Step 2: Verify reading page loads**

Open `/reading/?part=part4_subagents_hooks` in browser — should show empty/error (content file doesn't exist yet). This confirms the reader will pick up the new entry.

- [ ] **Step 3: Commit**

```bash
git add content/lectures.json
git commit -m "feat: add lecture 4 entry to lectures.json"
```

---

### Task 2: Write conspect — sections 1-2 (Зачем делегировать + Как устроен субагент)

**Files:**
- Create: `/content/part4_subagents_hooks.md`

- [ ] **Step 1: Create file with header, ToC, and sections 1-2**

Start the conspect file with:
- H1 title: `# Лекция 4 · Субагенты, оркестрация и Hooks`
- Blockquote metadata with 3 lines (context, audience, focus) — as specified in spec header block (3 lines, not 2 like part3)
- `## Содержание` with numbered anchor links to all 7 sections — follow format from part3_tools_mcp.md lines 8-15
- `---`
- `## 1. Зачем делегировать` — content per spec section 1:
  - Opening paragraph referencing lecture 1 briefly: "В лекции 1 мы познакомились с субагентами... здесь разберём, как настраивать и какие паттерны использовать"
  - 1.1 Проблема единого контекста — context window as limited resource, degradation with complex tasks
  - 1.2 Аналогия с командой — one dev vs team with specialization
  - 1.3 Два способа решения — manual decomposition vs agent-initiated delegation
  - ASCII diagram: single overloaded agent vs multiple focused agents
- `## 2. Как устроен субагент` — content per spec section 2:
  - 2.1 Отличие от основного агента
  - 2.2 Жизненный цикл — ASCII diagram (Spawn → Work → Return)
  - 2.3 Таблица наследования (Rules, Permissions, Skills, Context, Other results) — markdown table
  - 2.4 Параллельность
  - 2.5 Ключевое ограничение — returns result only, not full context

Reference for style and depth: `/content/part3_tools_mcp.md` sections 1-2 (~130 lines). Target: ~120-140 lines for sections 1-2.

- [ ] **Step 2: Verify markdown renders correctly**

Open `/reading/?part=part4_subagents_hooks` in browser. Check:
- Title and metadata render
- ToC anchors work (even though later sections are missing)
- ASCII diagrams display in code blocks
- Tables render properly

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "feat: add lecture 4 conspect — sections 1-2 (delegation, subagent model)"
```

---

### Task 3: Write conspect — section 3 (Субагенты в инструментах)

**Files:**
- Modify: `/content/part4_subagents_hooks.md`

- [ ] **Step 1: Write section 3**

Append `## 3. Субагенты в инструментах` with subsections:

- 3.1 Claude Code — Agent tool, AGENTS.md (include example fragment), subagent_type, parallel launch, worktree isolation. Include code/config fragments.
- 3.2 Roo Code — modes (Code, Architect, Ask, Debug, Orchestrator), Boomerang Tasks, custom modes via `.roomodes`. Include `.roomodes` JSON fragment with roleDefinition, groups, customInstructions.
- 3.3 Kilo Code — fork of Roo, differences (Task Manager UI, extended modes). If differences are minimal, merge with 3.2 as "Roo Code / Kilo Code".
- 3.4 OpenCode — agents in `opencode.json` or `.opencode/agents/*.md`. Include config fragment.
- 3.5 Сводная таблица — markdown table comparing: model, configuration, parallelism, isolation, custom types.

Key: each tool section must include a **real config fragment** that readers can copy. Follow the pattern from part3_tools_mcp.md sections 3.4-3.5 (MCP server config examples).

Reference for depth: part3_tools_mcp.md section 3 (~140 lines). Target: ~160-180 lines for section 3.

- [ ] **Step 2: Verify in browser**

Check that config code blocks render with syntax highlighting and tables display correctly.

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "feat: add lecture 4 section 3 — subagents across tools"
```

---

### Task 4: Write conspect — section 4 (Декомпозиция задач и паттерны)

**Files:**
- Modify: `/content/part4_subagents_hooks.md`

- [ ] **Step 1: Write section 4**

Append `## 4. Декомпозиция задач и паттерны оркестрации` with:

- 4.1 Когда разбивать руками — domain knowledge, strict ordering, conflict risk
- 4.2 Когда доверять агенту — clear task, agent knows project, independent subtasks
- 4.3 Паттерны оркестрации — four patterns, each with:
  - ASCII diagram (simple block diagram)
  - One-paragraph description
  - Concrete example
  - "Поддержка" note (which tools support natively)
  - Patterns: Fan-out/fan-in, Pipeline, Specialist, Supervisor
- Summary table at end: pattern × how it works × example × tool support

Target: ~100-120 lines.

- [ ] **Step 2: Verify in browser**

Check ASCII diagrams and tables render.

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "feat: add lecture 4 section 4 — task decomposition and orchestration patterns"
```

---

### Task 5: Write conspect — section 5 (Hooks)

**Files:**
- Modify: `/content/part4_subagents_hooks.md`

- [ ] **Step 1: Write section 5**

Append `## 5. Hooks` with:

- 5.1 Краткое напоминание — concept from lecture 1, "промпты — предложения, hooks — гарантии"
- 5.2 Продвинутые сценарии — list of 7 scenarios (autoformat, lint, autocommit, notify, tests, context injection, block dangerous ops)
- 5.3 Claude Code — settings.json hooks config, 12 events, 3 handler types, matcher. Include **advanced example**: chain of hooks (lint → test → commit on Stop). Real JSON config fragment.
- 5.4 OpenCode — plugin system. Include **real plugin code** (TypeScript). Show tool.execute.after example. Note: JS/TS not shell, higher flexibility but higher threshold.
- 5.5 Roo Code / Kilo Code — no native hooks. Show full workaround: custom MCP tool definition + `.roo/rules-code/` rule + `.roomodes` config. Note limitation: model can skip.
- 5.6 Сводная таблица hooks — mechanism, guarantee, language, event count, complexity
- 5.7 Стратегия выбора — when to use which approach

Key: this section builds on lecture 1 (sections 3.6-3.8) but goes DEEPER. Don't repeat basic concepts — reference them. Focus on advanced configs and cross-tool comparison.

Reference: lecture 1 hooks section is ~130 lines. Target for section 5: ~140-160 lines (deeper but DRY vs lecture 1).

- [ ] **Step 2: Verify in browser**

Check JSON and TypeScript code blocks render with highlighting.

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "feat: add lecture 4 section 5 — hooks across all tools"
```

---

### Task 6: Write conspect — sections 6-7 (Подводные камни + Итоги)

**Files:**
- Modify: `/content/part4_subagents_hooks.md`

- [ ] **Step 1: Write sections 6 and 7**

Append `## 6. Подводные камни` with 4 subsections, each following **Проблема → Симптом → Решение** pattern:
- 6.1 Потеря контекста между агентами
- 6.2 Дублирование работы
- 6.3 Зацикливание
- 6.4 Избыточная декомпозиция

Append `## 7. Итоги и практика` with:
- 7.1 Ключевые выводы — 6 numbered takeaways (per spec section 7.1)
- 7.2 Практическое задание — Задание 1 (configure subagent) and Задание 2 (configure hook), each with tool-specific variants for Claude Code, Roo/Kilo, OpenCode
- 7.3 Следующая лекция — "Лекция 5 · Безопасность агентов" teaser

Target: ~100-120 lines for both sections.

- [ ] **Step 2: Final conspect review**

Open `/reading/?part=part4_subagents_hooks` in browser. Full check:
- All 7 sections render
- All ToC anchor links work
- All tables render
- All code blocks have proper highlighting
- ASCII diagrams are readable
- Total length should be ~600-700 lines (comparable to part3: 703 lines)

- [ ] **Step 3: Commit**

```bash
git add content/part4_subagents_hooks.md
git commit -m "feat: complete lecture 4 conspect — pitfalls, takeaways, practice assignments"
```

---

### Task 7: Create presentation — skeleton and sections 1-2

**Files:**
- Create: `/presentations/part4/index.html`

- [ ] **Step 1: Create presentation file with skeleton + first slides**

Copy the full HTML/CSS structure from `/presentations/part3/index.html` as starting point. Then:

1. Update `<title>` to `Субагенты, оркестрация и Hooks · Лекция 4`
2. Keep ALL CSS styles (same design system — root vars, reveal overrides, slide-shell, chapter-shell, kicker, etc.)
3. Keep Yandex.Metrika script (same counter ID)
4. Keep Reveal.js config at the bottom (same settings)
5. Replace ALL slide `<section>` elements with new content

Create these slides:

**Title slide (chapter-shell):**
- Kicker: "AI Engineering Course"
- Hero title: "Субагенты, оркестрация и Hooks"
- Hero sub: "Лекция 4"
- Meta row: "Аудитория: разработчики · Фокус: делегирование и автоматизация"

**Содержание slide:**
- Numbered list of 7 sections

**Section 1 slides (3-4 slides):**
- "Зачем делегировать" chapter divider
- Problem slide: one agent with overloaded context (visual diagram)
- Solution slide: multiple focused agents
- Two approaches: manual vs automatic decomposition

**Section 2 slides (3-4 slides):**
- "Как устроен субагент" chapter divider
- Lifecycle diagram slide
- Inheritance table slide
- Key constraint: returns result only

Target: ~12-14 slides for skeleton + sections 1-2.

- [ ] **Step 2: Verify presentation opens**

Open `/presentations/part4/index.html` in browser. Check:
- Slides render with correct styling
- Navigation works (arrows, hash-based)
- Slide counter shows correct total

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "feat: add lecture 4 slides — skeleton + sections 1-2"
```

---

### Task 8: Add presentation slides — section 3 (Субагенты в инструментах)

**Files:**
- Modify: `/presentations/part4/index.html`

- [ ] **Step 1: Add section 3 slides**

Insert after section 2 slides:

**Section 3 slides (~8-10 slides):**
- "Субагенты в инструментах" chapter divider
- Claude Code slide: Agent tool + AGENTS.md (code fragment)
- Claude Code slide: subagent_type + worktree isolation
- Roo Code slide: modes + Orchestrator diagram
- Roo Code slide: .roomodes config fragment
- Kilo Code slide: differences from Roo (or merged slide "Roo Code / Kilo Code")
- OpenCode slide: agents config fragment
- Comparison table slide (4 tools × 5 criteria)

Use `<pre><code class="language-json">` for config fragments and `<pre><code class="language-markdown">` for AGENTS.md examples. Follow code block styling from part3.

- [ ] **Step 2: Verify slides**

Check all new slides render, code has syntax highlighting, table is readable.

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "feat: add lecture 4 slides — section 3 (subagents in tools)"
```

---

### Task 9: Add presentation slides — section 4 (Паттерны оркестрации)

**Files:**
- Modify: `/presentations/part4/index.html`

- [ ] **Step 1: Add section 4 slides**

**Section 4 slides (~5-6 slides):**
- "Декомпозиция задач" chapter divider
- When to decompose manually vs trust the agent (two-column or bullet comparison)
- Fan-out / fan-in pattern (diagram + example)
- Pipeline pattern (diagram + example)
- Specialist + Supervisor patterns (combined slide or two slides)
- Summary table: pattern × support

ASCII/text diagrams should use styled `<pre>` blocks or simple HTML tables, matching part3's visual approach.

- [ ] **Step 2: Verify slides**

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "feat: add lecture 4 slides — section 4 (orchestration patterns)"
```

---

### Task 10: Add presentation slides — section 5 (Hooks)

**Files:**
- Modify: `/presentations/part4/index.html`

- [ ] **Step 1: Add section 5 slides**

**Section 5 slides (~6-8 slides):**
- "Hooks" chapter divider
- Concept reminder: "промпты — предложения, hooks — гарантии"
- Scenarios slide (list of 7 use cases)
- Claude Code hooks slide: settings.json config example
- OpenCode plugins slide: TypeScript plugin example
- Roo/Kilo workaround slide: MCP tool + rules approach
- Hooks comparison table slide (5 tools × 5 criteria)
- Strategy decision tree slide

- [ ] **Step 2: Verify slides**

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "feat: add lecture 4 slides — section 5 (hooks)"
```

---

### Task 11: Add presentation slides — sections 6-7 + closing

**Files:**
- Modify: `/presentations/part4/index.html`

- [ ] **Step 1: Add sections 6-7 and closing slides**

**Section 6 slides (~4 slides):**
- "Подводные камни" chapter divider
- Pitfall slides: one slide per pitfall (problem → symptom → solution) or two combined slides
  - Потеря контекста + Дублирование
  - Зацикливание + Избыточная декомпозиция

**Section 7 slides (~3-4 slides):**
- "Итоги" chapter divider
- Key takeaways (6 points)
- Practice assignments slide
- Next lecture teaser: "Лекция 5 · Безопасность агентов"

**Closing slides (2 slides) — copy from part3:**
- "Дальше → Лекция 5" slide with navigation pills to lectures 1, 2, 3 + home (not self-link to part4)
- "Остаёмся на связи" slide with author photo + Telegram QR code

Total slides should be ~35-45 (comparable to part3: 43 slides).

- [ ] **Step 2: Final presentation review**

Full check:
- Total slide count reasonable (~35-45)
- Navigation works end-to-end
- All code blocks have syntax highlighting
- Slide counter shows correct c/t
- Last slide has correct links

- [ ] **Step 3: Commit**

```bash
git add presentations/part4/index.html
git commit -m "feat: complete lecture 4 slides — pitfalls, takeaways, closing"
```

---

### Task 12: Update part3 closing slide to link to part4

**Files:**
- Modify: `/presentations/part3/index.html`

- [ ] **Step 1: Add part4 navigation pill to part3's "Дальше" slide**

In `/presentations/part3/index.html`, find the "Дальше" section (near line 1781). The `<div class="meta-row">` contains pills for lectures 1, 2, and home. Add a pill for lecture 3 (self) is not needed, but add a link to part4:

Add before the "Главная" pill:
```html
<a class="pill" href="../part4/">Слайды · Лекция 4</a>
```

- [ ] **Step 2: Verify**

Open `/presentations/part3/` and navigate to the "Дальше" slide. Confirm the "Слайды · Лекция 4" pill appears and links correctly.

- [ ] **Step 3: Commit**

```bash
git add presentations/part3/index.html
git commit -m "fix: add lecture 4 link to part3 closing slide"
```

---

### Task 13: Update index.html — move lecture 4 card to active

> Note: Task numbering updated — this was previously Task 12.

**Files:**
- Modify: `/index.html:344-348`

- [ ] **Step 1a: Insert active lecture 4 card into "В работе" grid**

Insert the following HTML **before** the `</section>` closing tag of the "В работе" grid (line 340 of index.html), after the lecture 3 article:

```html
      <article class="card" style="--card-accent: var(--rose)">
        <span class="complexity high">Высокая сложность</span>
        <h3>Лекция 4 · Субагенты, оркестрация и Hooks</h3>
        <p>Делегирование субагентам, паттерны оркестрации, hooks и автоматизация в Claude Code, Roo Code, Kilo Code, OpenCode.</p>
        <div class="row">
          <a class="btn" href="./presentations/part4/">Презентация</a>
          <a class="btn secondary" href="./reading/?part=part4_subagents_hooks">Конспект</a>
        </div>
      </article>
```

- [ ] **Step 1b: Delete the old planned lecture 4 card from "Планируются" grid**

Remove the entire `<article class="card planned">` block for lecture 4 (originally lines 344-348). This prevents the card appearing in both sections.

- [ ] **Step 2: Verify index page**

Open `/index.html` in browser. Check:
- Lecture 4 card appears in "В работе" section with correct accent color
- "Презентация" link goes to `/presentations/part4/`
- "Конспект" link goes to `/reading/?part=part4_subagents_hooks`
- Lecture 4 no longer appears in "Планируются"
- Footer still says "9 лекций"

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: activate lecture 4 card on homepage"
```

---

### Task 14: Final cross-check

- [ ] **Step 1: Verify all navigation paths**

Check in browser:
1. `/index.html` → click "Презентация" for lecture 4 → slides load
2. `/index.html` → click "Конспект" for lecture 4 → reading page loads
3. `/reading/` → lecture 4 card appears in the list
4. `/reading/?part=part4_subagents_hooks` → full conspect renders
5. `/presentations/part4/` → slides work, navigation pills link correctly
6. Lecture 3 slides → "Дальше → Лекция 4" link works (from part3 closing slide)

- [ ] **Step 2: Content consistency check**

Verify:
- Conspect and slides cover the same 7 sections
- All config fragments in conspect match those in slides
- No content from lecture 1 is duplicated verbatim (only referenced). Compare against the delta table in the spec.
- Lecture 5 teaser matches in both conspect and slides
- Excluded topics are NOT present (per spec "Что НЕ входит"): no Agent SDK / programmatic orchestration, no background agents rehash, no basic definitions repeated from lecture 1, no timing annotations

- [ ] **Step 3: Fact-check**

Verify factual accuracy of all technical claims in the conspect:
- **Claude Code:** AGENTS.md format, Agent tool parameters (subagent_type values, isolation, mode), worktree behavior, hook events and handler types — cross-reference with current Claude Code documentation / behavior
- **Roo Code:** mode names (Code, Architect, Ask, Debug, Orchestrator), .roomodes JSON structure (roleDefinition, groups, customInstructions), Boomerang Tasks behavior — cross-reference with Roo Code docs
- **Kilo Code:** claimed differences from Roo Code (Task Manager UI, extended modes) — verify these are accurate and current
- **OpenCode:** agents config format (opencode.json vs .opencode/agents/), plugin event names (tool.execute.before/after, session.created/idle, etc.) — cross-reference with OpenCode docs
- **Hooks comparison table:** event counts, guarantee claims, mechanism descriptions — verify each cell
- **Orchestration patterns:** verify that "fan-out is native in Claude Code but sequential in Roo/Kilo" is accurate
- Config fragments: verify syntax is valid JSON/TypeScript/Markdown, field names are correct

Use web search and official documentation to verify. Flag any inaccuracies found.

- [ ] **Step 4: Output fact-check report**

Write a report to `docs/superpowers/reports/2026-03-30-lecture4-factcheck.md` with:
- **Status:** PASS / ISSUES FOUND
- For each tool (Claude Code, Roo Code, Kilo Code, OpenCode):
  - Claims checked
  - Verified ✅ or Inaccurate ❌ (with correction)
- Config fragments: valid ✅ or invalid ❌ (with fix)
- Summary: total claims checked, total verified, total issues

Present the report to the user for review.

- [ ] **Step 5: Final commit (if any fixes needed)**

```bash
git add -A
git commit -m "fix: lecture 4 cross-check and fact-check fixes"
```
