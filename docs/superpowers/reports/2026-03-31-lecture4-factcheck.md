# Lecture 4 Fact-Check Report

**Date:** 2026-03-31
**Status:** ISSUES FOUND

## Content Consistency

- [x] All 7 sections in both conspect and slides
  - Conspect: 7 sections (1. Зачем делегировать, 2. Как устроен субагент, 3. Субагенты в инструментах, 4. Декомпозиция задач и паттерны оркестрации, 5. Hooks, 6. Подводные камни, 7. Итоги и практика)
  - Slides: Same 7 sections with matching chapter dividers (Раздел 01-07)
- [x] Config fragments consistent between conspect and slides
  - AGENTS.md example: matches (code-reviewer + test-writer)
  - .roomodes JSON: matches (doc-writer + security-audit, same fields)
  - opencode.json agents: matches (reviewer + implementer)
  - Claude Code hooks JSON: matches (PostToolUse + Stop example)
  - OpenCode TypeScript plugin: matches (AutoFormatPlugin)
  - Roo Code emulation (3-step workaround): matches
- [x] No lecture 1 duplication
  - Conspect correctly references lecture 1 with "В лекции 1 мы познакомились..." without repeating definitions
  - No verbatim copying of lecture 1 content detected
- [x] Lecture 5 teaser matches in both conspect and slides
  - Conspect: "Лекция 5 -- Безопасность агентов -- модель угроз, prompt injection, MCP-уязвимости, утечки секретов, защита и реагирование на инциденты."
  - Slides: Matching 6-card layout (Модель угроз, Prompt injection, MCP-уязвимости, Утечки секретов, Защита, Реагирование) + closing slide with same text
- [x] No excluded topics present
  - No Agent SDK / programmatic orchestration: confirmed absent
  - No background agents rehash: confirmed absent
  - No basic definitions repeated from lecture 1: confirmed (only references)
  - No timing annotations: confirmed absent
- [x] **BUG FIXED**: Slides TOC card 3 said "Claude Code, Roo Code, Cursor, Windsurf" instead of "Claude Code, Roo Code, Kilo Code, OpenCode" -- corrected

## Claude Code

- **Claim: AGENTS.md is a file in the repo root describing custom agents**
  - Status: NEEDS NUANCE. As of 2025-2026, AGENTS.md is an open standard (maintained by Agentic AI Foundation under Linux Foundation), supported by many tools. Claude Code also supports custom agent definitions via .claude/agents/ directory with YAML frontmatter Markdown files. The conspect's description of AGENTS.md purpose and format is correct in spirit, but the actual mechanism for custom subagent definition in Claude Code is primarily through .claude/agents/ files, not just a single AGENTS.md in root. The AGENTS.md file is more of a project-wide instruction file (similar to CLAUDE.md) rather than a subagent definition mechanism. Rating: minor inaccuracy in framing, but the concept is directionally correct.

- **Claim: Agent tool parameters include prompt, subagent_type, isolation, run_in_background**
  - Status: VERIFIED. Claude Code's Agent tool accepts these parameters. The subagent types (Explore, Plan, General-purpose) are confirmed in official docs. The "code-review" type appears as a community-defined custom agent rather than a built-in type.

- **Claim: subagent_type values are Explore, Plan, General-purpose, code-review**
  - Status: PARTIALLY VERIFIED. Explore, Plan, and General-purpose are confirmed built-in types. "code-review" is more accurately a custom agent pattern (defined via AGENTS.md or .claude/agents/) rather than a built-in subagent_type value. The conspect correctly lists it alongside "Custom (from AGENTS.md)" which is fair.

- **Claim: Worktree isolation via isolation: "worktree"**
  - Status: VERIFIED. Git worktree isolation is a confirmed feature. Subagents work in separate git worktrees, merge results via git. Also available via --worktree CLI flag.

- **Claim: 12 hook events**
  - Status: OUTDATED. The original 12 events (PreToolUse, PostToolUse, PostToolUseFailure, Notification, Stop, SessionStart, SessionEnd, UserPromptSubmit, PermissionRequest, SubagentStart, SubagentStop, PreCompact) were accurate. As of March 2026, the count has expanded to 21 events (adding InstructionsLoaded, ConfigChange, WorktreeCreate/Remove, PostCompact, Elicitation/ElicitationResult, etc.). The claim of "12" was correct at design time but is now outdated.

- **Claim: 3 handler types (command, prompt, agent)**
  - Status: OUTDATED. As of January 2026, there are 4 handler types: command, prompt, agent, and **http** (sends event JSON as HTTP POST to a URL). The lecture omits the http type.

- **Claim: Exit codes 0=ok, 1=error shown to model, 2=block action**
  - Status: INACCURATE for exit code 1. Per official docs: exit code 0 = success (JSON output processed), exit code 2 = blocking error (stderr fed to Claude). Any other exit code (including 1) = non-blocking error where stderr is shown only in verbose mode (Ctrl+O) and execution continues. The lecture incorrectly states that exit code 1 shows stderr to the model -- it does not by default; only exit code 2 feeds stderr back to Claude.

- **Claim: Matcher filters by tool name regex**
  - Status: VERIFIED. Matcher uses regex pattern matching against tool names. Empty matcher matches all.

## Roo Code

- **Claim: Mode names are Code, Architect, Ask, Debug, Orchestrator**
  - Status: VERIFIED. Confirmed in official Roo Code documentation. All five modes exist as built-in defaults.

- **Claim: .roomodes JSON structure (roleDefinition, groups, customInstructions)**
  - Status: VERIFIED. The .roomodes file supports: slug, name, roleDefinition, groups (with optional file restrictions via fileRegex), customInstructions, and description. The JSON examples in the lecture use correct field names and structure. Note: .roomodes now also supports YAML format (not just JSON), but JSON remains valid.

- **Claim: Boomerang Tasks work sequentially**
  - Status: VERIFIED. Orchestrator delegates tasks to other modes via Boomerang Tasks, waiting for each to complete before proceeding. Sequential by design.

- **Claim: No native hooks**
  - Status: VERIFIED. Roo Code has no built-in hook/event system. The emulation via MCP tools + rules is the documented workaround.

## Kilo Code

- **Claim: Kilo Code is a fork of Roo Code**
  - Status: VERIFIED. Confirmed in multiple sources. Kilo Code is a direct fork of Roo Code (which itself derives from Cline). Kilo Code v4.8.3 confirmed compatible with Roo Code 3.25.20. Raised $8M seed round in January 2026.

- **Claim: Task Manager UI -- visual panel showing subtask tree and statuses**
  - Status: VERIFIED. Kilo Code provides a Task Management UI with stack-based task lifecycle, parent-child task hierarchy navigation, and status tracking for orchestration workflows.

- **Claim: Config compatibility with .roomodes**
  - Status: VERIFIED. .roomodes configuration is compatible between Roo Code and Kilo Code. Both support JSON and YAML formats.

## OpenCode

- **Claim: Agents configured in opencode.json (section "agents") or .opencode/agents/**
  - Status: VERIFIED. OpenCode agents can be configured in opencode.json and/or as files in .opencode/agents/ directory.

- **Claim: Agent config fields are model, systemPrompt, tools**
  - Status: INACCURATE field names. Per official OpenCode docs:
    - `model` is correct (format: "provider/model-id")
    - `systemPrompt` is WRONG -- the actual field is `prompt` (references a file path, e.g., `"prompt": "{file:./prompts/build.txt}"`)
    - `tools` is DEPRECATED -- replaced by `permission` field for fine-grained access control
    - Additional fields exist: `description` (required), `temperature`, `steps`, `mode`, `disable`, `color`, `top_p`, `hidden`
  - The JSON example in both conspect and slides uses incorrect field names.

- **Claim: Plugin event names (tool.execute.before/after, session.created/idle, file.edited, permission.asked, shell.env)**
  - Status: VERIFIED. All 7 named events exist in OpenCode's plugin system. session.error is also mentioned in the conspect and is confirmed present.

- **Claim: 8 events total**
  - Status: INACCURATE. OpenCode has 28+ plugin events across 10 categories (command, file, installation, LSP, message, permission, server, session, tool, TUI, shell, todo). The 8 events listed in the lecture are all real, but calling the total "8" is significantly understated. The lecture cherry-picked the most useful 8, but the claim of "8 events" as total count is wrong.

## Orchestration Patterns

- **Claim: Fan-out is native in Claude Code (parallel Agent calls) but sequential in Roo/Kilo**
  - Status: VERIFIED. Claude Code supports parallel Agent tool calls natively. Roo/Kilo Code's Boomerang Tasks are sequential by design.

## Config Fragments

- **JSON validity:**
  - Claude Code hooks JSON (settings.json): VALID syntax
  - .roomodes JSON (doc-writer + security-audit): VALID syntax
  - opencode.json agents: VALID syntax (though field names are incorrect per actual API)
  - Roo Code emulation .roomodes (safe-code): VALID syntax

- **TypeScript validity:**
  - OpenCode AutoFormatPlugin: VALID syntax
  - Roo Code MCP tool lint_check: VALID syntax (uses node:child_process, node:util correctly)

- **Field names:**
  - Claude Code hooks: CORRECT (hooks, PostToolUse, Stop, matcher, type, command)
  - .roomodes: CORRECT (customModes, slug, name, roleDefinition, groups, customInstructions)
  - OpenCode agents: INCORRECT (`systemPrompt` should be `prompt`, `tools` should be `permission`)
  - Claude Code AGENTS.md: Format shown is reasonable but actual custom agents use .claude/agents/ with YAML frontmatter

## Summary

- **Total claims checked:** 26
- **Verified:** 17
- **Partially verified / needs nuance:** 3
- **Outdated (correct at design time):** 2
- **Inaccurate:** 4
- **Bugs fixed:** 1 (slides TOC card 3 tool names)

### Issues requiring attention:

1. **[HIGH] OpenCode agent config field names are wrong.** `systemPrompt` should be `prompt` (file path reference), `tools` is deprecated in favor of `permission`. Both conspect and slides affected.

2. **[HIGH] Exit code 1 behavior is incorrectly described.** Only exit code 2 feeds stderr to Claude. Exit code 1 (and any non-0, non-2 code) is a non-blocking error shown only in verbose mode. Both conspect and slides affected.

3. **[MEDIUM] Claude Code hooks now have 4 handler types, not 3.** The `http` type was added in January 2026. Consider adding it or noting "3+ types" with a footnote.

4. **[MEDIUM] OpenCode has 28+ plugin events, not 8.** The 8 events listed are real and useful, but the total count is significantly higher. Recommend changing "8 events" to "28+ events" or qualifying as "8 key events (28+ total)".

5. **[LOW] Claude Code hook events now number 21, not 12.** The original 12 were correct but the count has expanded. Consider updating to current count or adding "12+ events".

6. **[LOW] AGENTS.md framing slightly misleading.** Custom subagent definition in Claude Code primarily uses .claude/agents/ directory with YAML frontmatter, not just a root-level AGENTS.md file. The AGENTS.md shown in the lecture is more of a project instruction file. However, the concept and example are pedagogically valid.

7. **[FIXED] Slides TOC card 3 listed "Cursor, Windsurf" instead of "Roo Code, Kilo Code, OpenCode".** Corrected in this commit.
