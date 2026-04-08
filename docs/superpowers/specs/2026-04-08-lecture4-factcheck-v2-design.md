# Lecture 4 Fact-Check v2 — Design Spec

**Date:** 2026-04-08
**Status:** Approved
**Scope:** Focused fact-check of Lecture 4 § 3.4 (tools comparison table) and § 5 (hooks)
**Baseline:** `docs/superpowers/reports/2026-03-31-lecture4-factcheck.md`
**Next step:** Writing plan → execution → content edit spec (separate)

## 1. Goal

Produce a reproducible, focused fact-check report for two sections of Lecture 4:

- **§ 3.4** — tools comparison table (Claude Code / Roo Code / Kilo Code / OpenCode)
- **§ 5** — hooks, entire section (5.1–5.7)

The report becomes the input for the next spec — content edits to the markdown and slides. Every issue in the report must carry a ready-to-apply recommendation, so the edit spec can use it as acceptance criteria.

## 2. Scope

### In scope

- All claims in `content/part4_subagents_hooks.md` § 3.4 — six criteria (Модель субагентности, Конфигурация, Параллельность, Изоляция, Кастомные типы, Фоновый запуск) × three columns (Claude Code, Roo Code / Kilo Code combined, OpenCode) = 18 cells, plus the paragraph that follows the table
- All claims in § 5.1 through § 5.7 — reminder, advanced scenarios, Claude Code configs, OpenCode plugins, Roo/Kilo emulation, summary table, selection strategy
- Code fragments in § 5: Claude Code hooks JSON, OpenCode `AutoFormatPlugin` TypeScript plugin, Roo Code `lint_check` MCP tool and `.roomodes` snippet — syntax and field-name validation
- Re-verification of every HIGH/MEDIUM/LOW issue from the 2026-03-31 baseline report that falls inside these two sections

### Out of scope

- Stylistic quality of the hooks section (todo.md task 4 — "too AI-ish" — handled in a separate spec)
- New content additions (todo.md tasks 1, 3, 6 — models update, oh-my-opencode, subagent MCP access — handled in the edit spec)
- Lecture sections § 1, § 2, § 3.1–3.3, § 4, § 6, § 7 — a claim from § 3.1–3.3 is only touched if it is the source of truth for a cell in § 3.4
- Any edits to `content/part4_subagents_hooks.md` or `presentations/part4/index.html`
- The existing baseline report — not modified, referenced

## 3. Sources and verification method

### Primary sources (authoritative)

- **Claude Code** — `docs.claude.com/en/docs/claude-code/*`, `github.com/anthropics/claude-code`
- **Roo Code** — `docs.roocode.com`, `github.com/RooCodeInc/Roo-Code`
- **Kilo Code** — `docs.kilocode.ai`, `github.com/Kilo-Org/kilocode`
- **OpenCode** — `docs.opencode.ai`, `github.com/sst/opencode`

### Tools

- `WebFetch` — fetch specific doc pages when URL is known
- `mcp__plugin_context7_context7__*` — doc lookup for indexed libraries
- `WebSearch` — fallback for discovery (finding the right doc page or changelog); never the sole source for a claim
- **Blogs, Medium, community posts** — not authoritative; cite only when the post itself links to a primary source, and then cite the primary source

### Per-claim process

1. Extract the concrete claim from markdown — quote plus line number
2. Find confirmation or refutation in a primary source — save URL and, when possible, permalink or anchor
3. Assign status: `VERIFIED` / `PARTIALLY VERIFIED` / `OUTDATED` / `INACCURATE` / `UNVERIFIABLE`
4. If there is an issue, write a ready-to-apply recommendation

### Evidence rule

Every claim with status `VERIFIED`, `PARTIALLY VERIFIED`, `OUTDATED`, or `INACCURATE` must carry at least one primary-source URL. `UNVERIFIABLE` is allowed only when the primary source is actually unreachable or absent — not as a shortcut for "didn't find it fast enough".

### Baseline delta

Re-verify each issue from `2026-03-31-lecture4-factcheck.md` that falls inside § 3.4 or § 5. Tools change — an 8-day gap is usually nothing, but we do not assume. When the status matches, record it. When it changed, note the delta explicitly in the report's Delta section.

## 4. Report structure

**File:** `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Template:**

```markdown
# Lecture 4 Fact-Check v2 — § 3.4 & § 5

**Date:** 2026-04-08
**Scope:** § 3.4 (tools comparison table) + § 5 (hooks)
**Baseline:** 2026-03-31-lecture4-factcheck.md
**Status:** <filled in after completion>

## § 3.4 — Tools comparison table

### Criterion: Модель субагентности
- **Claim (Claude Code):** "Явный spawn через Agent tool" (part4_subagents_hooks.md:299)
  - **Status:** VERIFIED | PARTIALLY VERIFIED | OUTDATED | INACCURATE | UNVERIFIABLE
  - **Source:** <URL>
  - **Evidence:** <short quote or summary>
  - **Recommendation:** — | <ready-to-apply replacement text>
- **Claim (Roo Code / Kilo Code):** "Режимы + Orchestrator / Boomerang"
- **Claim (OpenCode):** "Агенты в конфигурации"

### Criterion: Конфигурация
  - Claims: Claude Code `AGENTS.md` + `subagent_type`; Roo/Kilo `.roomodes` (JSON); OpenCode `opencode.json` / `.opencode/agents/`

### Criterion: Параллельность
  - Claims: Claude Code "Да (несколько Agent-вызовов)"; Roo/Kilo "Последовательная (Boomerang)"; OpenCode "Зависит от реализации"

### Criterion: Изоляция
  - Claims: Claude Code "Worktree (git-копия)"; Roo/Kilo "Отдельный контекст (общая FS)"; OpenCode "Отдельный контекст (общая FS)"

### Criterion: Кастомные типы
  - Claims: Claude Code `AGENTS.md`; Roo/Kilo `.roomodes`; OpenCode "Агенты в конфиге"

### Criterion: Фоновый запуск
  - Claims: Claude Code `run_in_background: true`; Roo/Kilo "Нет"; OpenCode "Нет"

### Post-table paragraph
- **Claim:** "Самая продвинутая модель субагентности — у Claude Code: полная изоляция через worktree, параллелизм и фоновый режим..." — verify framing as opinion vs fact

## § 5 — Hooks

### § 5.1 Brief reminder
### § 5.2 Advanced scenarios (7 scenarios)
### § 5.3 Claude Code — advanced configs
  - Per-claim: 4 handler types, 21 events, exit codes, matcher semantics, JSON validity
### § 5.4 OpenCode — plugins
  - Per-claim: event names, event count, plugin config shape, TS code validity
### § 5.5 Roo/Kilo — emulation
  - Per-claim: no native hooks, MCP tool approach, .roomodes snippet validity
### § 5.6 Hooks comparison table
  - Per-cell: mechanism, handler language, guaranteed execution, etc.
### § 5.7 Selection strategy
  - Per-claim

## Summary

- **Total claims checked:** N
- **Verified:** N
- **Partially verified:** N
- **Outdated:** N
- **Inaccurate:** N
- **Unverifiable:** N

### Issues (HIGH)
1. **[HIGH]** <short title>
   - **Location:** part4_subagents_hooks.md:LINE (+ presentations/part4/index.html if applicable)
   - **Current text:** "<quote>"
   - **Recommendation:** "<ready-to-apply replacement>"
   - **Source:** <URL>

### Issues (MEDIUM)
...

### Issues (LOW)
...

## Delta vs 2026-03-31 baseline

- **Confirmed still valid:** <list of baseline issues still present>
- **Changed status:** <baseline issue X: was OUTDATED, now VERIFIED because Y>
- **New findings:** <issues not present in baseline>
```

### Structural notes

- The **per-claim atom** is the core unit — quote, line number, status, source, evidence, recommendation
- The **Delta section** makes the relationship to the baseline explicit — readers don't have to guess what changed
- **Issues in the Summary** are sorted by severity and contain the ready-to-apply replacement text — formatted for copy-paste into the edit spec
- Slides are mentioned in a recommendation only when the edit touches the slide too (for example, the same incorrect `systemPrompt` field in `presentations/part4/index.html`)

## 5. Deliverables

1. New report at `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`, following the template above
2. Report committed as its own commit with message: `docs: add lecture 4 fact-check v2 (§ 3.4 + § 5)`
3. No edits to `content/part4_subagents_hooks.md`, `presentations/part4/index.html`, or the baseline report

## 6. Acceptance criteria

- [ ] All 18 cells (6 criteria × 3 columns) of § 3.4, plus the post-table paragraph, covered by per-claim entries
- [ ] All seven subsections of § 5 covered by per-claim entries
- [ ] Every issue contains: severity, location (file:line), quote of the current text, ready-to-apply recommendation, and a primary-source URL
- [ ] Every claim with status `VERIFIED` / `OUTDATED` / `INACCURATE` / `PARTIALLY VERIFIED` carries at least one primary-source URL; `UNVERIFIABLE` is used only when the primary source is genuinely unreachable
- [ ] Delta vs 2026-03-31 baseline section is filled — for every HIGH/MEDIUM issue from the baseline that falls inside § 3.4 or § 5, current status is explicitly stated
- [ ] Code fragments in § 5 (Claude Code hooks JSON, OpenCode TypeScript plugin, Roo MCP tool + .roomodes) are validated at least for syntax and field names
- [ ] Summary with final counts is filled
- [ ] Report is committed as its own commit

## 7. Anti-criteria — explicit non-goals

- No edits to lecture text or slides
- No new sections or content added to the lecture
- No rewriting of the "too AI-ish" hooks prose
- No fact-check of other lecture sections (§ 1, § 2, § 3.1–3.3, § 4, § 6, § 7)
- No modification of the baseline report

## 8. Relation to todo.md

This spec covers two items from `todo.md`:

- **Task 2** — "Сделай факт чек таблицы сравнения" → § 3.4 coverage
- **Task 5** — "Проведи фактчек раздела hooks, подготовь отчёт" → § 5 coverage

Tasks 1, 3, 4, 6 remain for the subsequent edit spec.
