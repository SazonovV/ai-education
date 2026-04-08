# Lecture 4 Fact-Check v2 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce a focused fact-check report for Lecture 4 § 3.4 (tools comparison table) and § 5 (hooks), with per-claim status + ready-to-apply recommendations, feeding into a later content edit spec.

**Architecture:** Research/verification task, not code. Output is a single markdown file. Each task verifies a block of claims against official primary sources, then appends entries to the report file.

**Tech Stack:** Markdown, WebFetch, Context7 MCP, WebSearch (fallback only).

**Spec:** `docs/superpowers/specs/2026-04-08-lecture4-factcheck-v2-design.md`

**Baseline report to cross-reference:** `docs/superpowers/reports/2026-03-31-lecture4-factcheck.md`

**Source file to fact-check:** `content/part4_subagents_hooks.md` (read-only for this plan)

---

## File Map

| Action | File | Responsibility |
|--------|------|---------------|
| Create | `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md` | The fact-check report — the only file this plan writes |
| Read | `content/part4_subagents_hooks.md` | Source of truth for the claims being verified (§ 3.4, § 5) |
| Read | `docs/superpowers/reports/2026-03-31-lecture4-factcheck.md` | Baseline for the Delta section |

---

## Primary source URLs — first places to check

Use these as the starting point for WebFetch. Don't rely on memory. If a page has moved, use WebSearch with `site:docs.claude.com`, `site:docs.roocode.com`, etc. to find the current location.

- **Claude Code subagents:** https://docs.claude.com/en/docs/claude-code/sub-agents
- **Claude Code hooks:** https://docs.claude.com/en/docs/claude-code/hooks
- **Claude Code hooks reference:** https://docs.claude.com/en/docs/claude-code/hooks-guide
- **Claude Code settings:** https://docs.claude.com/en/docs/claude-code/settings
- **Roo Code custom modes:** https://docs.roocode.com/features/custom-modes
- **Roo Code Boomerang Tasks:** https://docs.roocode.com/features/boomerang-tasks
- **Roo Code changelog / GitHub:** https://github.com/RooCodeInc/Roo-Code/releases
- **Kilo Code docs:** https://docs.kilocode.ai
- **Kilo Code GitHub:** https://github.com/Kilo-Org/kilocode
- **OpenCode agents:** https://docs.opencode.ai/agents
- **OpenCode plugins:** https://docs.opencode.ai/plugins
- **OpenCode GitHub:** https://github.com/sst/opencode

---

## Report entry template

Every per-claim entry in the report uses this exact shape. When a step says "record a per-claim entry", use this:

```markdown
- **Claim:** "<exact quote from lecture>" (part4_subagents_hooks.md:LINE)
  - **Status:** VERIFIED | PARTIALLY VERIFIED | OUTDATED | INACCURATE | UNVERIFIABLE
  - **Source:** <primary-source URL>
  - **Evidence:** <1-2 sentence summary or short quote from the source>
  - **Recommendation:** — (if verified) | "<ready-to-apply replacement text>"
```

Rules:
- Every non-UNVERIFIABLE entry must have at least one primary-source URL
- `Recommendation: —` means no fix needed (claim verified)
- Recommendations must be ready-to-paste, not "rephrase this" hand-waving

---

## Task 1: Create report skeleton

**Files:**
- Create: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

- [ ] **Step 1: Write the skeleton file**

Create the file with this exact content:

```markdown
# Lecture 4 Fact-Check v2 — § 3.4 & § 5

**Date:** 2026-04-08
**Scope:** § 3.4 (tools comparison table) + § 5 (hooks)
**Baseline:** 2026-03-31-lecture4-factcheck.md
**Status:** IN PROGRESS

## § 3.4 — Tools comparison table

### Criterion: Модель субагентности

<entries to be filled in Task 2>

### Criterion: Конфигурация

<entries to be filled in Task 3>

### Criterion: Параллельность

<entries to be filled in Task 4>

### Criterion: Изоляция

<entries to be filled in Task 5>

### Criterion: Кастомные типы

<entries to be filled in Task 6>

### Criterion: Фоновый запуск

<entries to be filled in Task 7>

### Post-table paragraph

<entry to be filled in Task 8>

## § 5 — Hooks

### § 5.1 Brief reminder

<entries to be filled in Task 9>

### § 5.2 Advanced scenarios (7 scenarios)

<entries to be filled in Task 9>

### § 5.3 Claude Code — advanced configs

<entries to be filled in Task 10>

### § 5.4 OpenCode — plugins

<entries to be filled in Task 11>

### § 5.5 Roo Code / Kilo Code — emulation

<entries to be filled in Task 12>

### § 5.6 Hooks comparison table

<entries to be filled in Task 13>

### § 5.7 Selection strategy

<entries to be filled in Task 14>

## Summary

<to be filled in Task 15>

## Delta vs 2026-03-31 baseline

<to be filled in Task 16>
```

- [ ] **Step 2: Commit the skeleton**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: scaffold lecture 4 fact-check v2 report"
```

---

## Task 2: § 3.4 — Criterion "Модель субагентности"

**Files:**
- Read: `content/part4_subagents_hooks.md` (lines 295-307 for § 3.4)
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify (3):**
1. **Claude Code:** "Явный spawn через Agent tool" (line 299)
2. **Roo Code / Kilo Code:** "Режимы + Orchestrator / Boomerang" (line 299)
3. **OpenCode:** "Агенты в конфигурации" (line 299)

- [ ] **Step 1: Read the lecture source**

Read `content/part4_subagents_hooks.md` lines 295-307 to confirm exact text, then record the three quotes with their line numbers.

- [ ] **Step 2: Verify Claude Code claim**

Fetch `https://docs.claude.com/en/docs/claude-code/sub-agents`. Look for confirmation that Claude Code spawns subagents via an explicit `Agent` tool (or equivalent — the tool may be named `Task` in some docs). Record the URL and a 1-sentence evidence summary.

- [ ] **Step 3: Verify Roo Code / Kilo Code claim**

Fetch `https://docs.roocode.com/features/boomerang-tasks` and `https://docs.roocode.com/features/custom-modes`. Confirm that Roo Code uses Modes + Orchestrator / Boomerang Tasks for subagent behavior. Also fetch the Kilo Code equivalent page (start with `https://docs.kilocode.ai`, navigate to orchestrator/modes).

- [ ] **Step 4: Verify OpenCode claim**

Fetch `https://docs.opencode.ai/agents`. Confirm that OpenCode uses a config-driven agent model (not dynamic spawning like Claude Code).

- [ ] **Step 5: Append three per-claim entries to the report**

Replace `<entries to be filled in Task 2>` under `### Criterion: Модель субагентности` with three entries, each following the report entry template. For verified claims, `Recommendation: —`. For any issue, write a ready-to-apply replacement.

- [ ] **Step 6: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 3.4 criterion — модель субагентности"
```

---

## Task 3: § 3.4 — Criterion "Конфигурация"

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify (3):**
1. **Claude Code:** "`AGENTS.md` + `subagent_type`" (line 300)
2. **Roo Code / Kilo Code:** "`.roomodes` (JSON)" (line 300)
3. **OpenCode:** "`opencode.json` / `.opencode/agents/`" (line 300)

**Baseline reference:** The 2026-03-31 baseline flagged the AGENTS.md framing as "LOW — slightly misleading" (custom subagents in Claude Code are primarily defined via `.claude/agents/` with YAML frontmatter). Re-verify this.

- [ ] **Step 1: Verify Claude Code config claim**

Fetch `https://docs.claude.com/en/docs/claude-code/sub-agents` again (looking specifically for where custom subagents are defined). Check whether `.claude/agents/` with YAML frontmatter is the primary mechanism, or whether `AGENTS.md` has been adopted. Record evidence.

- [ ] **Step 2: Verify Roo Code / Kilo Code config claim**

Fetch `https://docs.roocode.com/features/custom-modes`. Confirm `.roomodes` is the file, and check whether JSON is still supported or if YAML has become the recommended format (baseline noted both are supported). Record evidence.

- [ ] **Step 3: Verify OpenCode config claim**

Fetch `https://docs.opencode.ai/agents`. Confirm that agents can live in `opencode.json` and/or `.opencode/agents/`. Record evidence.

- [ ] **Step 4: Append three per-claim entries**

Replace `<entries to be filled in Task 3>` with three entries. If Claude Code claim is inaccurate, recommendation should be a concrete replacement like `"\`.claude/agents/*.md\` + \`subagent_type\`"` or similar.

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 3.4 criterion — конфигурация"
```

---

## Task 4: § 3.4 — Criterion "Параллельность"

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify (3):**
1. **Claude Code:** "Да (несколько Agent-вызовов)" (line 301)
2. **Roo Code / Kilo Code:** "Последовательная (Boomerang)" (line 301)
3. **OpenCode:** "Зависит от реализации" (line 301)

- [ ] **Step 1: Verify Claude Code parallelism**

Confirm via `https://docs.claude.com/en/docs/claude-code/sub-agents` or subagent tool docs that Claude Code supports multiple parallel Agent tool invocations in the same model turn.

- [ ] **Step 2: Verify Roo Code / Kilo Code parallelism**

Fetch Boomerang Tasks doc. Confirm that Boomerang tasks are sequential by design. Check Kilo Code docs for whether their Task Manager UI adds any parallelism.

- [ ] **Step 3: Verify OpenCode parallelism**

Fetch `https://docs.opencode.ai/agents`. Determine the actual parallelism story — "depends on implementation" is vague. If there's a concrete answer (e.g., "no parallel spawning; agents are profiles in the same session"), recommendation should replace the vague text.

- [ ] **Step 4: Append three per-claim entries**

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 3.4 criterion — параллельность"
```

---

## Task 5: § 3.4 — Criterion "Изоляция"

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify (3):**
1. **Claude Code:** "Worktree (git-копия)" (line 302)
2. **Roo Code / Kilo Code:** "Отдельный контекст (общая FS)" (line 302)
3. **OpenCode:** "Отдельный контекст (общая FS)" (line 302)

**Baseline reference:** Baseline verified Claude Code worktree isolation.

- [ ] **Step 1: Verify Claude Code worktree isolation**

Fetch Claude Code docs on worktree or `isolation: "worktree"` parameter. Confirm still supported. Record URL.

- [ ] **Step 2: Verify Roo Code / Kilo Code isolation**

Confirm via Roo Code custom-modes or Boomerang docs that subtasks share the filesystem and get a fresh model context (not a git isolation).

- [ ] **Step 3: Verify OpenCode isolation**

Confirm via OpenCode agents doc whether agents share filesystem and context or not.

- [ ] **Step 4: Append three per-claim entries**

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 3.4 criterion — изоляция"
```

---

## Task 6: § 3.4 — Criterion "Кастомные типы"

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify (3):**
1. **Claude Code:** "`AGENTS.md`" (line 303)
2. **Roo Code / Kilo Code:** "`.roomodes`" (line 303)
3. **OpenCode:** "Агенты в конфиге" (line 303)

**Note:** This row overlaps with the "Конфигурация" row and may need cross-referencing. If the claim is already an inaccuracy flagged in Task 3, carry the same recommendation here.

- [ ] **Step 1: Cross-reference with Task 3 findings**

Read the entries written in Task 3 and determine if the claims in Task 6 have the same issue. If so, reuse the same recommendation.

- [ ] **Step 2: Verify each claim independently if not covered**

For any claim not already resolved by Task 3, fetch the appropriate primary source.

- [ ] **Step 3: Append three per-claim entries**

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 3.4 criterion — кастомные типы"
```

---

## Task 7: § 3.4 — Criterion "Фоновый запуск"

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify (3):**
1. **Claude Code:** "`run_in_background: true`" (line 304)
2. **Roo Code / Kilo Code:** "Нет" (line 304)
3. **OpenCode:** "Нет" (line 304)

- [ ] **Step 1: Verify Claude Code background mode**

Fetch Claude Code Agent tool doc. Confirm `run_in_background: true` is still the parameter name. Record URL.

- [ ] **Step 2: Verify Roo Code / Kilo Code background claim**

Confirm no background mode exists in Roo/Kilo Code. Check their GitHub release notes for "background" mentions as a safety check.

- [ ] **Step 3: Verify OpenCode background claim**

Confirm no background mode in OpenCode. Note: a "fire and forget" or async agent feature may exist — check for it.

- [ ] **Step 4: Append three per-claim entries**

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 3.4 criterion — фоновый запуск"
```

---

## Task 8: § 3.4 — Post-table paragraph

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claim to verify (1):**
Line 306: "Самая продвинутая модель субагентности — у Claude Code: полная изоляция через worktree, параллелизм и фоновый режим. Roo Code / Kilo Code берут удобством Orchestrator-паттерна и визуализацией в Kilo. OpenCode — минимальный, но достаточный подход для проектов, где нужны предустановленные профили без динамического порождения агентов."

This is opinion-laced — verify only the factual sub-claims:
- Claude Code has worktree isolation, parallelism, background mode (covered by Tasks 2, 4, 5, 7)
- Kilo has visualization (Task Manager UI) — verified in baseline, re-confirm
- OpenCode uses preset profiles without dynamic spawning

- [ ] **Step 1: Verify Kilo Task Manager UI claim**

Fetch `https://docs.kilocode.ai` and find the Task Management UI page. Confirm it exists.

- [ ] **Step 2: Verify OpenCode "preset profiles without dynamic spawning" claim**

Re-check OpenCode agents doc. Confirm agents are defined ahead of time, not spawned dynamically at runtime.

- [ ] **Step 3: Append a per-claim entry for the paragraph**

Record as a single entry with status reflecting overall factual accuracy. Opinion framing (e.g., "самая продвинутая") is not an accuracy issue by itself — flag separately as "framing" note if needed.

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 3.4 post-table paragraph"
```

---

## Task 9: § 5.1 Reminder + § 5.2 Advanced scenarios

**Files:**
- Read: `content/part4_subagents_hooks.md` lines 422-450
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify:**

§ 5.1 (lines 428-432):
1. "Hook — реакция на событие жизненного цикла агента, выполняемая хостом (средой исполнения), а не моделью."
2. "Модель не решает, запускать ли hook, — он срабатывает автоматически при наступлении события."

§ 5.2 — seven scenarios (lines 438-450). Each is a general pattern claim, not a tool-specific one. Verification target: does each scenario describe a real, implementable use case across at least Claude Code or OpenCode?

- [ ] **Step 1: Read the source text**

Read `content/part4_subagents_hooks.md` lines 422-450.

- [ ] **Step 2: Verify § 5.1 reminder claims**

These are conceptual claims covered by Claude Code hooks docs and OpenCode plugin docs. Fetch `https://docs.claude.com/en/docs/claude-code/hooks` and `https://docs.opencode.ai/plugins`. Confirm the "host executes, not model" framing is accurate.

- [ ] **Step 3: Verify § 5.2 scenarios are implementable**

For each of the 7 scenarios, identify which event name in Claude Code (or OpenCode) would implement it. Record as a single block entry:
- Scenario 1 (auto-format after Edit) → PostToolUse with `Edit|Write` matcher
- Scenario 2 (lint after write) → PostToolUse
- Scenario 3 (auto-commit on Stop) → Stop event
- Scenario 4 (notification on error) → Notification event or error handler
- Scenario 5 (run tests after code change) → PostToolUse
- Scenario 6 (context injection on SessionStart) → SessionStart event
- Scenario 7 (block dangerous ops) → PreToolUse with exit code 2

If any scenario references an event that no longer exists, flag it.

- [ ] **Step 4: Append entries for § 5.1 and § 5.2**

Use a block entry for § 5.2 (one entry per scenario or one summary entry listing all 7).

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 5.1 reminder + § 5.2 scenarios"
```

---

## Task 10: § 5.3 Claude Code — advanced configs

**Files:**
- Read: `content/part4_subagents_hooks.md` lines 452-492
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify:**

1. **Config location** (line 454): `.claude/settings.json` (project) or `~/.claude/settings.json` (global)
2. **JSON snippet** (lines 458-477): `hooks.PostToolUse` + `hooks.Stop`, `matcher`, nested `hooks` array with `type: "command"` and `command` — all field names and structure
3. **"20+ событий"** (line 483): count of hook events; baseline said old report used "12", fact-check v1 said 21, lecture says 20+
4. **Event names listed** (line 483): PreToolUse, PostToolUse, Stop, SessionStart, UserPromptSubmit, PermissionRequest, Notification, PostCompact — each must still exist
5. **"4 типа обработчиков"** (line 484): command, prompt, agent, http — this is the corrected count
6. **Matcher semantics** (line 485): regex filter by tool name, empty matcher = all
7. **Env variables** (line 486): `$FILEPATH`, `$TOOL_NAME`, `$TOOL_INPUT` — each must still be supported
8. **Exit codes** (lines 487-490):
   - `0` — OK, continue
   - `1` — error (non-blocking, visible only in verbose)
   - `2` — block action, stderr fed to model
9. **`agent` handler type** (line 492): unique to Claude Code, runs subagent with tools

**Baseline reference:** Baseline flagged exit code 1 behavior as INACCURATE (fix not applied) — re-verify. Baseline said 4 handler types, lecture now shows 4 — check if lecture was already updated.

- [ ] **Step 1: Read the source text**

Read `content/part4_subagents_hooks.md` lines 452-492.

- [ ] **Step 2: Fetch Claude Code hooks doc**

Fetch `https://docs.claude.com/en/docs/claude-code/hooks` and/or `https://docs.claude.com/en/docs/claude-code/hooks-guide`. Save the section on:
- Config file location
- Event list (get exact current count)
- Handler types
- Matcher semantics
- Environment variables
- Exit codes

- [ ] **Step 3: Verify each sub-claim**

Check each of the 9 numbered claims against the fetched doc. For each issue, prepare a ready-to-apply recommendation.

Key expectations based on baseline (re-verify live, don't trust baseline blindly):
- Exit code 1: only exit 2 feeds stderr to Claude; exit 1 is non-blocking, visible only in verbose. If lecture says exit 1 shows stderr to model → INACCURATE.
- Events count: should be 20+ or current number from doc.
- Handler types: should be 4 (including http).

- [ ] **Step 4: Validate the JSON snippet**

Verify:
- Top-level is `hooks` object
- Inside, `PostToolUse` and `Stop` are arrays
- Each entry has `matcher` and nested `hooks` array
- Inner hook has `type` and `command`

Copy the JSON snippet from lines 458-477 and validate field names against the Claude Code schema. If the schema is documented, link to it.

- [ ] **Step 5: Append per-claim entries**

Replace `<entries to be filled in Task 10>` with one entry per numbered claim above. Include a specific entry for the JSON snippet validity.

- [ ] **Step 6: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 5.3 Claude Code hooks"
```

---

## Task 11: § 5.4 OpenCode — plugins

**Files:**
- Read: `content/part4_subagents_hooks.md` lines 494-527
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify:**

1. **Mechanism** (line 496): "hooks реализованы через систему плагинов — JS/TS-модули"
2. **Plugin signature** (lines 501-515): `async ({ $ }) =>` returning an object with event handlers
3. **Event names in TS code** (lines 503, 511): `tool.execute.after`, `session.idle`
4. **Plugin uses `$` shell template literal** (lines 507, 512)
5. **Event names listed in bullet** (line 522): `tool.execute.before`, `tool.execute.after`, `session.created`, `session.idle`, `session.error`, `file.edited`, `permission.asked`, `shell.env`
6. **SDK access** (line 523): `$` (shell), `client` (API), `project` (metadata)
7. **npm packages supported** (line 524)
8. **Plugin loading** (line 525): local `.opencode/plugins/` or npm `"plugin": ["package-name"]` in `opencode.json`

**Baseline reference:** Baseline flagged OpenCode event count as INACCURATE (28+ events across 10 categories, not 8). Re-verify count. Baseline also flagged `systemPrompt` → `prompt` and `tools` → `permission` in agent config, but that's § 3.3, not § 5.4 — only relevant if § 5.4 has the same issue.

- [ ] **Step 1: Read the source text**

Read `content/part4_subagents_hooks.md` lines 494-527.

- [ ] **Step 2: Fetch OpenCode plugins doc**

Fetch `https://docs.opencode.ai/plugins`. Also check `https://github.com/sst/opencode` for plugin examples if doc is thin.

- [ ] **Step 3: Verify event names and count**

For each of the 8 event names listed in line 522, confirm it exists in the current OpenCode plugin API. Get the current total count of plugin events. Baseline said 28+ — confirm or update.

- [ ] **Step 4: Validate the TypeScript snippet**

Check that `async ({ $ }) => { return { ... } }` is the current plugin shape. Confirm `tool.execute.after` receives `(input, output)` arguments as shown. Confirm `input.tool`, `output.args.filePath` field names.

- [ ] **Step 5: Verify loading mechanism**

Confirm `.opencode/plugins/` directory and npm `plugin` array in `opencode.json` are both supported.

- [ ] **Step 6: Append per-claim entries**

One entry per numbered claim above. For the event count, explicitly compare lecture text "и другие" framing vs actual total.

- [ ] **Step 7: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 5.4 OpenCode plugins"
```

---

## Task 12: § 5.5 Roo Code / Kilo Code — emulation

**Files:**
- Read: `content/part4_subagents_hooks.md` lines 529-584
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify:**

1. **"Не имеют нативных hooks"** (line 531) — core claim
2. **MCP tool workaround** — that a custom MCP tool + strict rules is the documented/viable workaround
3. **TypeScript MCP tool snippet** (lines 537-555): uses `node:child_process` + `promisify`, returns `ok:` / `fail:` prefixes
4. **`.roo/rules-code/lint-before-write.md` rule file** (lines 557-568): correct path and convention
5. **`.roomodes` snippet** (lines 572-582): `customModes` array with `slug`, `name`, `roleDefinition`, `groups` (with `fileRegex`), `customInstructions`
6. **"Не гарантия"** disclaimer (line 584): model can skip the check

**Baseline reference:** Baseline verified "no native hooks" and the emulation approach. Re-verify that Roo Code has not added native hooks between 2026-03-31 and 2026-04-08.

- [ ] **Step 1: Read the source text**

Read `content/part4_subagents_hooks.md` lines 529-584.

- [ ] **Step 2: Verify no native hooks (re-check)**

Fetch `https://docs.roocode.com` — look for "hooks" or "events" in the docs. Check `https://github.com/RooCodeInc/Roo-Code/releases` for the last few releases to catch any new hooks feature. Do the same for Kilo Code.

- [ ] **Step 3: Validate `.roomodes` field names**

Fetch `https://docs.roocode.com/features/custom-modes`. Confirm fields: `customModes`, `slug`, `name`, `roleDefinition`, `groups`, `customInstructions`. Confirm `groups` can contain a `["edit", { "fileRegex": "..." }]` tuple form. If the format has changed (e.g., moved to YAML-only), flag it.

- [ ] **Step 4: Validate rules file path convention**

Confirm `.roo/rules-code/*.md` is the current rules directory convention. If it has moved (e.g., `.roo/rules/` without `-code` suffix), flag.

- [ ] **Step 5: Validate MCP tool snippet**

The snippet is abbreviated ("Упрощённый MCP-tool") — verify that `execute({ scope })` signature style is compatible with MCP tool registration conventions. If it's purely pseudocode, mark the entry as "illustrative code, not a runnable example" to set reader expectations.

- [ ] **Step 6: Append per-claim entries**

One entry per numbered claim above.

- [ ] **Step 7: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 5.5 Roo/Kilo emulation"
```

---

## Task 13: § 5.6 Hooks comparison table

**Files:**
- Read: `content/part4_subagents_hooks.md` lines 586-594
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify (5 rows × 3 columns = 15 cells):**

| Критерий | Claude Code | OpenCode | Roo Code / Kilo Code |
|---|---|---|---|
| Механизм | Нативные hooks (settings.json) | Плагины (JS/TS) | Эмуляция (rules + custom tools) |
| Гарантия срабатывания | Да | Да | Нет (зависит от модели) |
| Язык обработчика | Shell / prompt / agent / http | JS/TS | Shell (через MCP tool) |
| Количество событий | 20+ | 20+ | Нет нативных |
| Сложность настройки | Низкая (JSON конфиг) | Средняя (код на TS) | Высокая (workaround) |

Each cell is a claim. Most will be validated by earlier tasks (§ 5.3, § 5.4, § 5.5) — this task cross-references and records per-cell entries.

- [ ] **Step 1: Read the source table**

Read `content/part4_subagents_hooks.md` lines 586-594.

- [ ] **Step 2: Cross-reference with Tasks 10, 11, 12**

For each cell:
- "Механизм" row — fully covered by Task 10/11/12 findings
- "Гарантия срабатывания" row — covered
- "Язык обработчика" row — Claude Code should be `Shell / prompt / agent / http` (4 types); OpenCode `JS/TS`; Roo/Kilo `Shell (через MCP tool)` — re-verify
- "Количество событий" row — Claude Code "20+" and OpenCode "20+" need cross-check with Task 10/11 counts. Baseline said Claude Code 21 and OpenCode 28+. If lecture says "20+" for both it's technically correct but understated for OpenCode.
- "Сложность настройки" row — subjective; flag as opinion, not a factual claim

- [ ] **Step 3: Append 15 per-cell entries**

One entry per cell. Where the claim is duplicated from an earlier task, cite the prior entry briefly (don't re-run the WebFetch).

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 5.6 hooks comparison table"
```

---

## Task 14: § 5.7 Selection strategy

**Files:**
- Read: `content/part4_subagents_hooks.md` lines 596-605
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

**Claims to verify:**

1. **"Нужна гарантия → Claude Code hooks или OpenCode plugins"** — cross-ref with § 5.6 "Гарантия" row
2. **"Нужна гибкость → OpenCode plugins"** — implies full TS/npm ecosystem
3. Any mention of specific tool capabilities (coverage check, Grafana metrics example, etc.)

This section is mostly strategic recommendation (opinion), but it builds on factual claims. Only verify the factual sub-claims.

- [ ] **Step 1: Read the source text**

Read `content/part4_subagents_hooks.md` lines 596-605.

- [ ] **Step 2: Verify factual sub-claims**

Confirm that the facts referenced (OpenCode plugins are TypeScript, have npm access, Claude Code hooks are config-driven) are consistent with Task 10/11 findings.

- [ ] **Step 3: Append per-claim entries**

Use one summary entry since most content is opinion/recommendation framing. Flag any factual claim that doesn't hold.

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: fact-check § 5.7 selection strategy"
```

---

## Task 15: Summary section

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

- [ ] **Step 1: Count statuses**

Read back through all per-claim entries written in Tasks 2-14. Count:
- Total claims checked
- VERIFIED
- PARTIALLY VERIFIED
- OUTDATED
- INACCURATE
- UNVERIFIABLE

- [ ] **Step 2: Compile issue list**

For every entry with status other than VERIFIED, promote to the Summary as an issue with severity:
- **HIGH** — factually wrong, user learning from the lecture will be misinformed on a core concept (e.g., exit code behavior, wrong field names in a code snippet they'll copy)
- **MEDIUM** — count/name mismatch, incomplete coverage of current state (e.g., 20+ vs 28+ events)
- **LOW** — minor framing or nuance (e.g., "AGENTS.md" vs ".claude/agents/" convention)

Each issue must include:
- Severity tag
- Short title
- Location (file:line, plus `presentations/part4/index.html` if the slide has the same claim — check the slides quickly if needed)
- Current text quote
- Ready-to-apply recommendation
- Primary-source URL

- [ ] **Step 3: Write Summary section**

Replace `<to be filled in Task 15>` under `## Summary` with:

```markdown
- **Total claims checked:** N
- **Verified:** N
- **Partially verified:** N
- **Outdated:** N
- **Inaccurate:** N
- **Unverifiable:** N

### Issues (HIGH)
<numbered list of HIGH issues>

### Issues (MEDIUM)
<numbered list of MEDIUM issues>

### Issues (LOW)
<numbered list of LOW issues>
```

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: compile lecture 4 fact-check v2 summary"
```

---

## Task 16: Delta vs 2026-03-31 baseline

**Files:**
- Read: `docs/superpowers/reports/2026-03-31-lecture4-factcheck.md`
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

- [ ] **Step 1: Extract baseline issues in scope**

Read the baseline report. List every HIGH/MEDIUM/LOW issue that falls inside § 3.4 or § 5. Known ones from earlier reading:
- Exit code 1 behavior (HIGH)
- 4 handler types not 3 (MEDIUM)
- OpenCode 28+ events not 8 (MEDIUM)
- Claude Code 21 events not 12 (LOW)
- AGENTS.md framing (LOW)

(There may be more — the baseline is the source of truth, not this list.)

- [ ] **Step 2: Cross-check each baseline issue against new findings**

For each baseline issue in scope:
- **Confirmed still valid** — the new report also flagged it
- **Changed status** — the new report has a different status (e.g., was OUTDATED, now VERIFIED because the lecture was updated in the meantime)
- **New findings** — issues in the new report that were not in the baseline

- [ ] **Step 3: Write the Delta section**

Replace `<to be filled in Task 16>` under `## Delta vs 2026-03-31 baseline` with:

```markdown
### Confirmed still valid
- <baseline issue X — current status + reference to entry in this report>
- ...

### Changed status
- <baseline issue Y — was <old status>, now <new status>, reason>
- ...

### New findings (not in baseline)
- <new issue Z — reference to entry in this report>
- ...
```

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: add delta section vs 2026-03-31 baseline"
```

---

## Task 17: Finalize report

**Files:**
- Modify: `docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`

- [ ] **Step 1: Update status header**

Change the top-matter `**Status:** IN PROGRESS` to the final status based on findings:
- `COMPLETE — NO ISSUES` if every claim is VERIFIED
- `COMPLETE — ISSUES FOUND` if there are any non-verified claims
- `COMPLETE — N HIGH / M MEDIUM / K LOW` for more specificity

- [ ] **Step 2: Remove any remaining `<to be filled>` placeholders**

Grep the file for `<to be filled` and `<entries to be filled` — none should remain.

Run: `grep -n "to be filled" docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md`
Expected: no output

- [ ] **Step 3: Run acceptance-criteria checklist**

Open the spec at `docs/superpowers/specs/2026-04-08-lecture4-factcheck-v2-design.md` and walk through the acceptance criteria in section 6. For each checkbox, confirm the report meets it.

- [ ] **Step 4: Commit final**

```bash
git add docs/superpowers/reports/2026-04-08-lecture4-factcheck-v2.md
git commit -m "docs: finalize lecture 4 fact-check v2 report"
```

---

## Self-review notes

- This plan produces a report, not code. There are no tests and no TDD — the "verification step" is the test equivalent (fetch primary source, compare against claim).
- Each task commits independently so partial progress is reviewable.
- Tasks 2-7 (§ 3.4 criteria) are each small but kept separate to match one commit = one logical unit. If the executor finds this too granular, they can batch 2-7 into a single commit.
- Task 10 (§ 5.3 Claude Code) is the heaviest task because hooks have the most sub-claims and the baseline flagged multiple issues here. Extra care warranted.
- Baseline cross-referencing is consolidated in Task 16 rather than scattered through every task. Individual tasks note baseline context but the delta compilation happens once at the end.
