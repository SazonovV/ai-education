# Kilo Code claims — deferred verification

**Date:** 2026-04-08
**Reason:** During the lecture 4 fact-check v2, fetches against `docs.kilocode.ai` / `kilo.ai/docs` and `docs.roocode.com` consistently hung. User decision: drop Roo Code from the lecture entirely, keep Kilo Code as a standalone tool, and defer Kilo Code claim verification to a later session.
**Parent report:** `2026-04-08-lecture4-factcheck-v2.md`

## How to use this file

Each entry below is a claim from `content/part4_subagents_hooks.md` that mentions Kilo Code (currently bundled with Roo Code as "Roo Code / Kilo Code" in the text). When picking this work back up:

1. Try fetching `https://kilo.ai/docs/...` directly (the canonical host as of 2026-04-08; `docs.kilocode.ai` and `kilocode.ai/docs` redirect there)
2. If WebFetch still hangs, use the GitHub repo as the primary source: `https://github.com/Kilo-Org/kilocode`
3. For each claim, fill in `Status`, `Source`, `Evidence`, `Recommendation` using the same template as the parent report
4. Once all entries are verified, fold them into the parent report under the appropriate criterion / section, then delete this file

## § 3.4 — Сводная таблица (Kilo Code column)

Each row of the table currently has the combined "Roo Code / Kilo Code" cell. After Roo Code is dropped, the column needs to be relabeled to "Kilo Code" and each cell verified against Kilo Code docs (not Roo Code docs).

### Criterion: Модель субагентности

- **Claim:** "Режимы + Orchestrator / Boomerang" (part4_subagents_hooks.md:299)
  - **Status:** DEFERRED
  - **Notes:** Verify that Kilo Code inherits the Mode + Orchestrator + Boomerang Tasks model from Roo Code. Look for: built-in modes (Code, Architect, Ask, Debug, Orchestrator), Boomerang Tasks pause-and-resume mechanic.

### Criterion: Конфигурация

- **Claim:** "`.roomodes` (JSON)" (part4_subagents_hooks.md:300)
  - **Status:** DEFERRED
  - **Notes:** Verify the file is named `.roomodes` (not `.kilocodemodes` or `.kilomodes`) in Kilo Code, and that the JSON structure with `customModes` array, `slug`, `name`, `roleDefinition`, `groups`, `customInstructions` is identical.

### Criterion: Параллельность

- **Claim:** "Последовательная (Boomerang)" (part4_subagents_hooks.md:301)
  - **Status:** DEFERRED
  - **Notes:** Verify that Kilo Code's Boomerang Tasks are sequential (no parallel subtask execution), as inherited from Roo Code.

### Criterion: Изоляция

- **Claim:** "Отдельный контекст (общая FS)" (part4_subagents_hooks.md:302)
  - **Status:** DEFERRED
  - **Notes:** Verify that Kilo Code subagent contexts are separate but share the file system (no worktree-style isolation).

### Criterion: Кастомные типы

- **Claim:** "`.roomodes`" (part4_subagents_hooks.md:303)
  - **Status:** DEFERRED
  - **Notes:** Same as Конфигурация — confirm `.roomodes` is the file name in Kilo Code.

### Criterion: Фоновый запуск

- **Claim:** "Нет" (part4_subagents_hooks.md:304)
  - **Status:** DEFERRED
  - **Notes:** Verify Kilo Code has no background/async subagent execution mode.

## § 3.2 — Roo Code / Kilo Code section (full rewrite as Kilo Code)

The entire § 3.2 (lines 200-259) currently describes "Roo Code / Kilo Code" as one unit. After Roo Code is dropped, this section needs to be rewritten as "Kilo Code" only. The technical claims that need verification:

- **Claim:** Kilo Code is a Roo Code fork with identical mode architecture and additional UI for task management (part4_subagents_hooks.md:202)
  - **Status:** DEFERRED
  - **Notes:** Verify the fork relationship and the Task Manager UI feature.

- **Claim:** Built-in modes — Code, Architect, Ask, Debug, Orchestrator (part4_subagents_hooks.md:206-210)
  - **Status:** DEFERRED
  - **Notes:** Verify these five built-in modes exist in Kilo Code with the described capabilities.

- **Claim:** Orchestrator + Boomerang Tasks mechanism (part4_subagents_hooks.md:212-230)
  - **Status:** DEFERRED
  - **Notes:** Verify Orchestrator role, Boomerang Tasks delegation flow, sequential execution.

- **Claim:** Custom modes via `.roomodes` JSON file with `slug`, `name`, `roleDefinition`, `groups`, `customInstructions` (part4_subagents_hooks.md:232-253)
  - **Status:** DEFERRED
  - **Notes:** Verify field names and `.roomodes` file location in Kilo Code.

- **Claim:** `groups` field values — `read`, `edit`, `command`, `browser`, `mcp` (part4_subagents_hooks.md:255)
  - **Status:** DEFERRED
  - **Notes:** Verify these exact group names are valid in Kilo Code.

- **Claim:** Kilo Code's Task Manager UI — visual panel showing subtask tree, status, results (part4_subagents_hooks.md:259)
  - **Status:** DEFERRED
  - **Notes:** Verify Task Manager UI exists and has the described features. Confirm `.roomodes` is fully compatible between Roo Code and Kilo Code.

## § 5.5 — Kilo Code (formerly Roo Code / Kilo Code) — эмуляция

The entire § 5.5 (lines 529-584) currently uses Roo Code idioms. After dropping Roo Code, the section needs Kilo Code-specific verification:

- **Claim:** Kilo Code has no native hooks system (part4_subagents_hooks.md:531)
  - **Status:** DEFERRED
  - **Notes:** Verify Kilo Code has no native hook mechanism (no settings.json hooks block, no plugin system, etc).

- **Claim:** TypeScript MCP tool snippet using `child_process.exec` to run `npm run lint` (part4_subagents_hooks.md:537-555)
  - **Status:** DEFERRED
  - **Notes:** Verify this is a valid MCP tool pattern that Kilo Code can consume. (MCP itself is standardized, so this should be straightforward.)

- **Claim:** `.roo/rules-code/lint-before-write.md` rule file path (part4_subagents_hooks.md:557)
  - **Status:** DEFERRED
  - **Notes:** Verify Kilo Code uses the same `.roo/rules-code/` directory layout for mode rules, or whether it uses a Kilo-specific path. This is the most likely place where Kilo's path could differ from Roo's.

- **Claim:** `.roomodes` snippet with `customModes` array, `slug: "safe-code"`, `groups` field with `["read", ["edit", { "fileRegex": "..." }], "command", "mcp"]` (part4_subagents_hooks.md:572-582)
  - **Status:** DEFERRED
  - **Notes:** Verify the snippet is valid for Kilo Code, especially the nested `groups` array with `fileRegex` constraint.

## § 5.6 — Сводная таблица hooks (Kilo Code column)

After dropping Roo Code, the column "Roo Code / Kilo Code" becomes "Kilo Code". Per-cell verification needed:

- **Mechanism:** "Эмуляция (rules + custom tools)" — DEFERRED
- **Гарантия срабатывания:** "Нет (зависит от модели)" — DEFERRED
- **Язык обработчика:** "Shell (через MCP tool)" — DEFERRED (verify Kilo's MCP support and the typical handler language)
- **Количество событий:** "Нет нативных" — DEFERRED
- **Сложность настройки:** "Высокая (workaround)" — DEFERRED (subjective but consistent with the emulation framing)

## § 5.7 — Стратегия выбора (Kilo Code bullet)

- **Claim:** "Roo Code / Kilo Code → эмуляция с оговорками. Работает для «мягких» сценариев … для критичных — комбинируйте с CI/CD" (part4_subagents_hooks.md:604)
  - **Status:** DEFERRED
  - **Notes:** Rewrite as "Kilo Code" only and verify the framing. The recommendation to combine with CI/CD is sound advice independent of the tool.
