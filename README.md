# LLM Prompt Guide

A lightweight repo that hosts a polished GPT‑5 prompting guide focused on agentic workflows, coding performance, and precise instruction-following. Use it as a reference for designing your own prompts, system messages, and tool-calling behaviors.

## Quick links

- Read the full guide: [GPT-5 Prompting Guide](./GPT-5-prompt-guide.md)
- Jump to key sections:
  - [Agentic workflow predictability](./GPT-5-prompt-guide.md#agentic-workflow-predictability)
  - [Controlling agentic eagerness](./GPT-5-prompt-guide.md#controlling-agentic-eagerness)
    - [Less eagerness](./GPT-5-prompt-guide.md#prompting-for-less-eagerness)
    - [More eagerness](./GPT-5-prompt-guide.md#prompting-for-more-eagerness)
  - [Tool preambles](./GPT-5-prompt-guide.md#tool-preambles)
  - [Reasoning effort](./GPT-5-prompt-guide.md#reasoning-effort)
  - [Responses API context reuse](./GPT-5-prompt-guide.md#reusing-reasoning-context-with-the-responses-api)
  - [Coding performance](./GPT-5-prompt-guide.md#maximizing-coding-performance-from-planning-to-execution)
  - [Cursor’s prompt tuning](./GPT-5-prompt-guide.md#collaborative-coding-in-production-cursors-gpt-5-prompt-tuning)
  - [Instruction pitfalls](./GPT-5-prompt-guide.md#instruction-following-and-pitfalls)
  - [Minimal reasoning](./GPT-5-prompt-guide.md#minimal-reasoning)
  - [Metaprompting](./GPT-5-prompt-guide.md#metaprompting)
  - [Appendix](./GPT-5-prompt-guide.md#appendix)

## What’s inside

- Practical prompts and patterns to calibrate autonomy vs. control
- Tool preamble patterns for progress visibility
- Guidance on `reasoning_effort` and verbosity tradeoffs
- Responses API tips to persist reasoning between tool calls
- Coding defaults (frontend stack, UI/UX patterns), plus “blend-in” standards
- Instruction-following pitfalls with concrete diffs to resolve conflicts

## How to use

- Copy–paste snippets from the guide and adapt them to your stack and product.
- Set defaults up front (frameworks, style, rules) so code output blends into your repo.
- Calibrate agentic “eagerness” with clear stop conditions and safe/unsafe actions.
- For multi-step tasks, split work across turns and reuse context with the Responses API.

## Repo structure

```text
.
├── GPT-5-prompt-guide.md   # Main guide
└── README.md               # You are here
```

## Contributing

Issues and PRs are welcome—suggest improvements, add examples, or refine language. Please keep edits concise and lint-friendly (single H1, proper heading levels, no bare URLs, language-tagged code fences).

## Attribution

Content is curated and adapted from the OpenAI Cookbook’s GPT‑5 prompting resources. See: [OpenAI Cookbook – GPT‑5 Prompting Guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)

Last updated: 2025-10-20
