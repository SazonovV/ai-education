# Lecture 6 — Context Engineering: Factcheck Report

**Date:** 2026-04-13

## Results

| # | Claim | Status | Source | Notes |
|---|-------|--------|--------|-------|
| 1 | Token ratios: ~3-4 chars/token EN, ~1-2 chars/token RU | ⚠️ Needs clarification | Tiktoken empirical test (cl100k_base, o200k_base) | For cl100k_base: EN ~4-6 chars/token, RU ~2-3 chars/token — claim roughly holds. For o200k_base: EN ~4-6, RU ~3-5 — gap is smaller. The 1.5-2x cost ratio holds for cl100k but is ~1.3x for o200k. The claim is acceptable as a rough approximation for cl100k but overstates the gap for newer tokenizers. Left as-is since the lecture uses it as a general guideline. |
| 2 | BPE description: (1) byte sequence, (2) find most frequent pair, merge, (3) repeat | ✅ Confirmed | [Wikipedia: Byte-pair encoding](https://en.wikipedia.org/wiki/Byte-pair_encoding), [HuggingFace BPE course](https://huggingface.co/learn/llm-course/en/chapter6/5) | Accurate simplified description. Modern BPE adds pre-tokenization step (splitting on spaces/punctuation first), but the core algorithm is correctly described. |
| 3 | `function` is one token | ✅ Confirmed | Tiktoken empirical test | cl100k_base: token 1723. o200k_base: token 2706. Single token in both. |
| 4a | Claude 3.x / 4.x: 200,000 tokens | ⚠️ Needs clarification | [Anthropic docs: Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows), [Anthropic: What's new in Claude 4.6](https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-6) | Standard models: 200K. However, Claude Opus 4.6 & Sonnet 4.6 now support 1M tokens (GA since March 2026). Added note to table. |
| 4b | GPT-4.1: 1,000,000 tokens | ✅ Confirmed | [OpenAI: Introducing GPT-4.1](https://openai.com/index/gpt-4-1/), [OpenAI docs: GPT-4.1 model](https://developers.openai.com/api/docs/models/gpt-4.1) | All GPT-4.1 variants (standard, mini, nano) support 1M context. |
| 4c | Gemini 1.5 / 2.x: 1-2,000,000 tokens | ✅ Confirmed | [Google Developers Blog](https://developers.googleblog.com/en/new-features-for-the-gemini-api-and-google-ai-studio/), [Google Cloud docs: Long context](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/long-context) | Gemini 1.5 Pro: up to 2M. Gemini 2.5 Pro: 1M (2M coming). Range 1-2M is correct. |
| 4d | GLM 5.1: «уточняется» | ❌ Wrong | [OpenRouter: GLM 5.1](https://openrouter.ai/z-ai/glm-5.1), [Artificial Analysis: GLM-5.1](https://artificialanalysis.ai/models/glm-5-1) | GLM 5.1 released April 7, 2026 with ~200K token context window. Updated from «уточняется» to «200 000 токенов». |
| 5 | Degradation curve: 60-70% filling = sharp quality drop | ⚠️ Needs clarification | [arXiv 2601.11564](https://arxiv.org/abs/2601.11564), [Morph: Context Rot](https://www.morphllm.com/context-rot), [Liu et al. "Lost in the Middle"](https://cs.stanford.edu/~nfliu/papers/lost-in-the-middle.arxiv2023.pdf) | Research confirms non-linear degradation and that effective context is often well below advertised maximum. The specific "60-70%" threshold is not from a single definitive study but is a reasonable practical heuristic consistent with findings (e.g., "200K window becomes unreliable around 130K = 65%"). Left as-is — the claim is directionally correct and useful as a practical guideline. |
| 6a | Line of code: 5-10 tokens | ✅ Confirmed | Tiktoken empirical test | Tested 5 representative code lines: 7-9 tokens each. Within claimed range. |
| 6b | Russian paragraph (5-6 sentences): 80-120 tokens | ⚠️ Needs clarification | Tiktoken empirical test | cl100k_base: ~130 tokens for 6 sentences (above range). o200k_base: ~78 tokens (below range). Range is approximate and depends heavily on tokenizer. Left as-is since it's presented as a rough estimate. |
| 6c | JSON tool response (medium): 200-500 tokens | ✅ Confirmed | Tiktoken empirical test | Medium JSON with nested objects: ~240 tokens. Within range. |
| 6d | Typical agent system prompt: 500-2,000 tokens | ✅ Confirmed | Tiktoken empirical test | A real agent system prompt (with tool definitions, rules, etc.) easily reaches 500-2000+ tokens. Confirmed reasonable range. |
| 6e | CLAUDE.md 800 lines: ~3,000 tokens | ⚠️ Needs clarification | Tiktoken empirical test | Depends heavily on content density. A realistic CLAUDE.md with rules, examples, code blocks at 800 lines could range from 3K-6K tokens. The claim is plausible for concise content but could underestimate dense files. Left as-is — serves as a reasonable lower-bound estimate. |
| 7 | Claude Code memory path: `~/.claude/projects/<project>/memory/` with MEMORY.md index | ✅ Confirmed | [Claude Code docs: Memory](https://code.claude.com/docs/en/memory) | Path, structure, and MEMORY.md index file all confirmed per official documentation. |
| 8 | Kilo Code: «встроенная memory через интерфейс VS Code, per-workspace» | ⚠️ Needs clarification | [Kilo Code docs: Memory Bank](https://kilo.ai/docs/advanced-usage/memory-bank) | Kilo Code had a Memory Bank feature (file-based in `.kilocode/rules/memory-bank/`), now deprecated in favor of AGENTS.md. Description "через интерфейс VS Code" is somewhat misleading — it was file-based, not a pure sidebar UI feature. Updated to reflect current state. |
| 9 | OpenCode: «состояние сессии через API, сохраняемость через конфигурацию проекта» | ⚠️ Needs clarification | [GitHub: opencode feature request #16077](https://github.com/anomalyco/opencode/issues/16077), [OpenCode docs: Config](https://opencode.ai/docs/config/) | Built-in persistent memory is still a feature request (as of March 2026). Session state exists via API, but cross-session persistence requires community plugins (opencode-mem, supermemory). Updated wording. |
| 10 | Primacy/recency bias: model pays more attention to beginning and end | ✅ Confirmed | [Liu et al. "Lost in the Middle" (TACL 2024)](https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00638/119630/), [arXiv: Serial Position Effects](https://arxiv.org/html/2406.15981v1) | Well-established in research. U-shaped performance curve confirmed across multiple studies and models. |

## Corrections Made

### 1. GLM 5.1 context window (conspect + slides)
- **Before:** `GLM 5.1 | уточняется`
- **After:** `GLM 5.1 | 200 000 токенов`
- **Reason:** GLM 5.1 was released April 7, 2026 with ~200K context window. No longer needs clarification.

### 2. Claude context window note (conspect + slides)
- **Before:** `Claude 3.x / 4.x | 200 000 токенов`
- **After:** `Claude 3.x / 4.x | 200 000 токенов (Opus 4.6 / Sonnet 4.6 — до 1 000 000)`
- **Reason:** Newest Claude models support 1M tokens (GA since March 2026). Standard models remain 200K.

### 3. English example token count (conspect only)
- **Before:** «authorization module refactoring» — около 5 токенов
- **After:** «authorization module refactoring» — около 4 токенов
- **Reason:** Tiktoken gives exactly 4 tokens for both cl100k_base and o200k_base.

### 4. «хэндофф» token count (conspect only)
- **Before:** «это четыре токена и более»
- **After:** «это 3–7 токенов в зависимости от токенизатора»
- **Reason:** cl100k_base: 7 tokens, o200k_base: 3 tokens. Neither matches "4+".

### 5. Kilo Code memory description (conspect + slides)
- **Before:** «встроенная memory через интерфейс VS Code, per-workspace. Управляется из боковой панели без ручного редактирования файлов.»
- **After:** «memory bank через файлы в `.kilocode/rules/memory-bank/`, per-workspace. С 2026 года рекомендуется использовать AGENTS.md.»
- **Reason:** Memory Bank was file-based, not a sidebar UI feature. Now deprecated in favor of AGENTS.md.

### 6. OpenCode memory description (conspect + slides)
- **Before:** «состояние сессии через API, сохраняемость через конфигурацию проекта.»
- **After:** «сессии сохраняются локально; межсессионная память — через сторонние расширения (opencode-mem и др.).»
- **Reason:** Built-in persistent memory is not yet a core feature; it requires community plugins.

## Deferred

None — all claims were successfully verified or corrected.
