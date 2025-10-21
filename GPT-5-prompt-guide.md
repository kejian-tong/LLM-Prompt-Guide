# GPT-5 Prompting Guide

Elevate GPT-5 performance for agentic tasks, coding, and instruction-following with practical prompts, patterns, and examples.

## Executive summary

- GPT-5 excels at tool use, long-context reasoning, and precise instruction following.
- Calibrate “agentic eagerness” with clear prompts and `reasoning_effort`; add stop conditions and safe/unsafe actions.
- Use tool preambles for progress visibility and the Responses API to persist reasoning between tool calls.
- For coding, set opinions (frameworks, style, rules) and match your codebase norms to boost quality and blend in.
- Prefer concise user-facing text but verbose, readable code diffs and comments.

## Table of contents

- [Quick start](#quick-start)
- [Agentic workflow predictability](#agentic-workflow-predictability)
- [Controlling agentic eagerness](#controlling-agentic-eagerness)
  - [Prompting for less eagerness](#prompting-for-less-eagerness)
  - [Prompting for more eagerness](#prompting-for-more-eagerness)
- [Tool preambles](#tool-preambles)
- [Reasoning effort](#reasoning-effort)
- [Reusing reasoning context with the Responses API](#reusing-reasoning-context-with-the-responses-api)
- [Maximizing coding performance, from planning to execution](#maximizing-coding-performance-from-planning-to-execution)
  - [Frontend app development](#frontend-app-development)
  - [Zero-to-one app generation](#zero-to-one-app-generation)
  - [Matching codebase design standards](#matching-codebase-design-standards)
- [Collaborative coding in production: Cursor’s GPT-5 prompt tuning](#collaborative-coding-in-production-cursors-gpt-5-prompt-tuning)
- [Optimizing intelligence and instruction-following](#optimizing-intelligence-and-instruction-following)
  - [Verbosity](#verbosity)
  - [Instruction following (and pitfalls)](#instruction-following-and-pitfalls)
- [Minimal reasoning](#minimal-reasoning)
- [Markdown formatting](#markdown-formatting)
- [Metaprompting](#metaprompting)
- [Appendix](#appendix)

## Quick start

- System (persistence)

```text
You are an agent—keep going until the user's query is completely resolved before ending your turn. Only stop when you are sure the problem is solved. If details are missing, make the most reasonable assumptions, proceed, and note assumptions at the end.
```

- Parameters

```json
{
  "reasoning_effort": "medium",
  "verbosity": "low"
}
```

Tips: Raise reasoning_effort to high for complex multi-step tasks (or when autonomy helps), drop to low for latency-sensitive flows. Keep verbosity low globally; use high verbosity only for writing code and diffs.

- Tool preamble (one-liner)

```text
Preamble: I’ll outline a brief plan, state what I’m doing and why before each tool call, and summarize changes at the end.
```

- Eagerness toggles

Less eagerness

```text
Context gathering budget: Max 2 tool calls. Parallelize discovery once, stop as soon as you can act. Prefer acting over more searching.
```

More eagerness

```text
Persistence: If uncertain, research or deduce and continue; don’t hand back at uncertainty. Document assumptions afterward.
```

- Coding defaults (blend into common stacks)

```text
Framework: Next.js (TypeScript)
Styling/UI: Tailwind CSS + shadcn/ui
Icons: Lucide
State: Zustand
```

- Responses API (reuse reasoning)

```json
{
  "previous_response_id": "rs_123..."
}
```

## Agentic workflow predictability

We trained GPT-5 with developers in mind: we’ve focused on improving tool calling, instruction following, and long-context understanding to serve as the best foundation model for agentic applications. If adopting GPT-5 for agentic and tool-calling flows, we recommend upgrading to the Responses API, where reasoning is persisted between tool calls, leading to more efficient and intelligent outputs.

## Controlling agentic eagerness

Agentic scaffolds span a spectrum of control—from high autonomy to tight programmatic branching. GPT-5 can operate anywhere along this spectrum, from making high-level decisions under ambiguity to executing focused, well-defined tasks. Use the prompts below to calibrate “eagerness.”

### Prompting for less eagerness

GPT-5 defaults to being thorough when gathering context. To limit tangential tool calls and reduce latency:

- Lower `reasoning_effort` for shallow exploration and better latency. Many workflows work well at medium or low.
- Define explicit exploration criteria so the model doesn’t over-explore.

#### Context gathering spec

- Goal: Get enough context fast. Parallelize discovery and stop as soon as you can act.
- Method:
  - Start broad, then fan out to focused subqueries.
  - In parallel, launch varied queries; read top hits per query. Deduplicate paths and cache; don’t repeat queries.
  - Avoid over-searching for context. If needed, run targeted searches in one parallel batch.
- Early stop criteria:
  - You can name exact content to change.
  - Top hits converge (~70%) on one area/path.
- Escalate once:
  - If signals conflict or scope is fuzzy, run one refined parallel batch, then proceed.
- Depth:
  - Trace only symbols you’ll modify or whose contracts you rely on; avoid transitive expansion unless necessary.
- Loop:
  - Batch search → minimal plan → complete task.
  - Search again only if validation fails or new unknowns appear. Prefer acting over more searching.

#### Context gathering budget (very low depth)

- Bias toward answering quickly, even if not fully certain.
- Absolute max of 2 tool calls.
- If you need more time, post a brief update with findings and open questions, then proceed after user confirmation.

Tip: Provide an “escape hatch” such as “proceed even if not fully certain” to allow shorter context-gathering steps.

### Prompting for more eagerness

To encourage autonomy and persistence, increase `reasoning_effort` and use a persistence prompt like:

#### Persistence spec

- You are an agent—keep going until the user's query is completely resolved before ending your turn.
- Only stop when you’re sure the problem is solved.
- Don’t hand back at uncertainty—research or deduce the most reasonable approach and continue.
- Don’t ask the human to confirm assumptions; decide the most reasonable assumption, proceed, and document assumptions afterward.

Clearly define stop conditions, safe vs unsafe actions, and when handing back to the user is required. For example, in a shopping flow, checkout/payment should require explicit user confirmation at lower uncertainty thresholds, while search should tolerate high uncertainty; in a coding setup, deleting files should be gated more than read-only searches.

## Tool preambles

On long agentic rollouts, brief updates on what the agent is doing and why make a big difference. GPT-5 is trained to provide upfront plans and progress updates as “tool preambles.”

You can steer preamble frequency and style—from detailed notes per tool call to a short upfront plan. Example prompt:

### Preamble guidance

- Begin by restating the user’s goal clearly and concisely before calling any tools.
- Outline a short, structured plan. As you execute edits, narrate each step succinctly and mark progress.
- End with a crisp summary of what changed vs the original plan.

### Example preamble output

```json
{
  "output": [
    {
      "id": "rs_…",
      "type": "reasoning",
      "summary": [
        { "type": "summary_text", "text": "Determining weather response…" }
      ]
    },
    {
      "id": "msg_…",
      "type": "message",
      "status": "completed",
      "content": [
        {
          "type": "output_text",
          "text": "I’m going to check a live weather service…"
        }
      ],
      "role": "assistant"
    },
    {
      "id": "fc_…",
      "type": "function_call",
      "status": "completed",
      "arguments": "{\"location\":\"San Francisco, CA\",\"unit\":\"f\"}",
      "name": "get_weather"
    }
  ]
}
```

## Reasoning effort

`reasoning_effort` controls how hard the model thinks and how willingly it calls tools. Default is medium; scale up for complex, multi-step tasks. Peak performance often comes from splitting separable tasks across multiple turns and persisting reasoning.

## Reusing reasoning context with the Responses API

Use the Responses API to persist reasoning between tool calls—improves performance and latency while reducing token usage.

For example, Tau-Bench Retail scores improved from 73.9% to 78.2% by switching to Responses and passing `previous_response_id`. This lets GPT-5 refer to its prior reasoning traces instead of reconstructing plans each time.

## Maximizing coding performance, from planning to execution

GPT-5 leads current models in coding: it can work across large repos, fix bugs, handle large diffs, and implement multi-file refactors and new features. It’s also excellent at greenfield apps. Use these defaults to help it shine.

### Frontend app development

Recommended defaults for new apps:

- Frameworks: Next.js (TypeScript), React, HTML
- Styling/UI: Tailwind CSS, shadcn/ui, Radix Themes
- Icons: Material Symbols, Heroicons, Lucide
- Animation: Motion
- Fonts: Inter, Geist, Mona Sans, IBM Plex Sans, Manrope

### Zero-to-one app generation

Early users found that rubric-based self-reflection raises one-shot quality.

#### Self-reflection prompt

- First, think of a rubric until you’re confident.
- Think through what makes a world-class one-shot app; create a rubric with 5–7 categories (for internal use only).
- Use the rubric to iterate internally until your solution scores top marks across categories.

### Matching codebase design standards

For incremental changes, code should “blend in.” GPT-5 will seek context (e.g., read `package.json`) but you can nudge it with explicit rules.

#### Guiding principles

- Clarity and reuse: Prefer modular, reusable components. Factor repeated UI into components.
- Consistency: Enforce a unified design system (color tokens, type, spacing, components).
- Simplicity: Favor small, focused components over complex styling/logic.
- Demo-oriented: Enable quick prototyping for streaming, multi-turn, and tool integrations.
- Visual quality: Follow a high visual bar (spacing, padding, hover states, etc.).

#### Frontend stack defaults

- Framework: Next.js (TypeScript)
- Styling: Tailwind CSS
- UI components: shadcn/ui
- Icons: Lucide
- State management: Zustand
- Directory structure:

```text
/src
  /app
    /api/[route]/route.ts       # API endpoints (dynamic segment)
    /(pages)                    # Page routes
  /components/                  # UI building blocks
  /hooks/                       # Reusable React hooks
  /lib/                         # Utilities (fetchers, helpers)
  /stores/                      # Zustand stores
  /types/                       # Shared TypeScript types
  /styles/                      # Tailwind config
```

#### UI/UX best practices

- Visual hierarchy: Limit to 4–5 font sizes/weights; use `text-xs` for captions; avoid `text-xl` except for hero/major headings.
- Color usage: Use one neutral base (e.g., `zinc`) and up to two accent colors.
- Spacing and layout: Use multiples of 4. Use fixed-height containers with internal scrolling for long streams.
- State handling: Use skeletons or `animate-pulse` for loading. Indicate clickability with hover transitions.
- Accessibility: Use semantic HTML and ARIA. Prefer Radix/shadcn for baked-in a11y.

## Collaborative coding in production: Cursor’s GPT-5 prompt tuning

Cursor, an AI code editor, tuned prompts to balance autonomy with readability. See their day-one integration post: [cursor.com/blog/gpt-5](https://cursor.com/blog/gpt-5).

### System prompt and parameter tuning

- Set low global verbosity for concise status updates; use high verbosity only for code edits to keep diffs readable.
- Encourage clarity-first code: descriptive names, comments where useful, straightforward control flow.

Example guidance (embed in system or tool-specific prompts):

> Write code for clarity first. Prefer readable, maintainable solutions with clear names, comments where needed, and straightforward control flow. Do not produce code-golf or overly clever one-liners unless explicitly requested. Use high verbosity for writing code and code tools.

Cursor also reduced deferrals by clarifying product behaviors (e.g., Undo/Reject changes, user preferences), enabling the agent to proceed with longer tasks autonomously.

### Softening over-thorough gathering

Older prompts like the following caused overuse of tools with GPT-5:

```xml
<maximize_context_understanding>
  Be THOROUGH when gathering information. Make sure you have the FULL picture before replying.
  Use additional tool calls or clarifying questions as needed.
  ...
</maximize_context_understanding>
```

Cursor removed the “maximize\_” tenor and softened language to balance autonomy with restraint. Structured XML-like specs (e.g., `<context_understanding_spec>`) improved adherence and cross-referencing.

```xml
<context_understanding>
  ...
  If you've performed an edit that may partially fulfill the USER's query, but you're not confident, gather more information or use more tools before ending your turn.
  Bias towards not asking the user for help if you can find the answer yourself.
</context_understanding>
```

## Optimizing intelligence and instruction-following

### Verbosity

Control response length with the `verbosity` parameter (distinct from `reasoning_effort`). You can also override verbosity in natural language for specific contexts (e.g., “high verbosity for coding tools only”).

### Instruction following (and pitfalls)

GPT-5 follows instructions precisely; vague or contradictory prompts waste tokens or cause conflicts. Consider this adversarial example around appointment scheduling:

- “Never schedule without explicit consent” conflicts with “auto-assign earliest same-day slot without contacting the patient.”
- “Always look up the patient profile first” conflicts with “In emergencies, direct 911 before any scheduling step.”

Proposed diff to resolve contradictions:

```diff
 Core entities include Patient, Provider, Appointment, and PriorityLevel (Red, Orange, Yellow, Green). Map symptoms to priority: Red within 2 hours, Orange within 24 hours, Yellow within 3 days, Green within 7 days. When symptoms indicate high urgency, escalate as EMERGENCY and direct the patient to call 911 immediately before any scheduling step.
 Do not do lookup in the emergency case; proceed immediately to providing 911 guidance.
 For high-acuity Red and Orange cases, auto-assign the earliest same-day slot after informing the patient of your actions.
```

Key fixes:

- Auto-assign only after informing the patient to remain consistent with “consent required.”
- Explicitly allow skipping lookup in emergency to prioritize 911 guidance.

## Minimal reasoning

Minimal `reasoning_effort` is the fastest setting that still benefits from reasoning. Best for latency-sensitive scenarios and many GPT-4.1-style prompts.

Tips:

- Start final answers with a brief thought summary (e.g., a few bullets) on higher-intelligence tasks.
- Use thorough tool-calling preambles with steady progress updates during agentic work.
- Disambiguate tool instructions and add persistence reminders to avoid premature termination.
- Prompted planning is more important at minimal effort—front-load plan creation and reflection between calls.

### Sample planning snippet

> Remember, you are an agent—keep going until the user's query is completely resolved before ending your turn. Decompose the user's query into all required sub-requests and confirm each is completed. Do not stop after partial completion. Only stop when you are sure the problem is solved.
>
> Plan extensively in accordance with the workflow steps before making function calls, and reflect on outcomes after each call to ensure the user's request (and sub-requests) are fully resolved.

## Markdown formatting

By default, GPT-5 does not format answers in Markdown. To induce Markdown where helpful:

- Use Markdown only where semantically correct (e.g., inline code, code fences, lists, tables).
- Use backticks for file, directory, function, and class names. Use \( and \) for inline math, \[ and \] for block math.
- If adherence drops over a long conversation, append a Markdown reminder every 3–5 user messages.

## Metaprompting

Early testers use GPT-5 to improve their own prompts. Ask what to add/remove to elicit desired behavior.

Metaprompt template

```text
When asked to optimize prompts, give answers from your own perspective—explain what specific phrases could be added to, or deleted from, this prompt to more consistently elicit the desired behavior or prevent the undesired behavior.
Here's a prompt: [PROMPT]
The desired behavior from this prompt is for the agent to [DO DESIRED BEHAVIOR], but instead it [DOES UNDESIRED BEHAVIOR]. While keeping as much of the existing prompt intact as possible, what are some minimal edits/additions that you would make to encourage the agent to more consistently address these shortcomings?
```

## Appendix

### SWE-Bench verified developer instructions

Edit files via diff with a special apply-patch command:

```bash
apply_patch << 'PATCH'
*** Begin Patch
[YOUR_PATCH]
*** End Patch
PATCH
```

Verify changes thoroughly. Some tests are hidden; double-check edge cases.

### Agentic coding tool definitions

#### Set 1: 4 functions, no terminal

```ts
type apply_patch = (_: { patch: string }) => any;
type read_file = (_: {
  path: string;
  line_start?: number;
  line_end?: number;
}) => any;
type list_files = (_: { path?: string; depth?: number }) => any;
type find_matches = (_: {
  query: string;
  path?: string;
  max_results?: number;
}) => any;
```

#### Set 2: 2 functions, terminal-native

```ts
type run = (_: {
  command: string[];
  session_id?: string | null;
  working_dir?: string | null;
  ms_timeout?: number | null;
  environment?: object | null;
  run_as_user?: string | null;
}) => any;

type send_input = (_: {
  session_id: string;
  text: string;
  wait_ms?: number;
}) => any;
```

As shared in the GPT-4.1 guide, use `apply_patch` for file edits to match training distribution.

### Tau-Bench Retail minimal reasoning instructions

As a retail agent, you can help users cancel/modify pending orders, return/exchange delivered orders, update default addresses, and provide info about profiles, orders, and related products.

Remember: keep going until fully resolved. If unsure, use tools to gather info—don’t guess. Plan and reflect between calls; ensure arguments are correct.

#### Workflow steps

- Authenticate the user (via email or name + ZIP). Do this even if a user ID is provided.
- Once authenticated, you can provide info about orders, products, or profiles.
- Help only one user per conversation.
- Before consequential actions (cancel, modify, return, exchange), list details and obtain explicit “yes” confirmation.
- Don’t invent procedures or add subjective recommendations.
- Make at most one tool call at a time; don’t respond to the user and call a tool simultaneously.
- Escalate to a human only if the request is out of scope.

#### Domain basics

- Times are EST (24-hour). “02:30:00” = 2:30 AM EST.
- Each user has email, default address, user ID, and payment methods (gift card, PayPal, or credit card).
- Store has ~50 product types; items vary by options (e.g., shirt color/size).
- Each product and item has a unique ID (unrelated to each other).
- Order statuses: pending, processed, delivered, cancelled. Actions generally allowed on pending or delivered orders.
- Exchange/modify tools are single-use—collect all changes first.

#### Cancel pending order

- Only if status is pending (check first).
- Confirm order ID and reason (“no longer needed” or “ordered by mistake”).
- After confirmation: status → cancelled; refund via original payment (immediate for gift card; 5–7 business days otherwise).

#### Modify pending order

- Only if status is pending (check first).
- Allowed changes: shipping address, payment method, or item options only.

#### Modify payment

- Choose a single payment method different from the original.
- If switching to gift card, balance must cover total.
- After confirmation: status remains pending; refund original payment (immediate for gift card; 5–7 days otherwise).

#### Modify items

- Single-use action; changes status to “pending (items modified)”; no further modify/cancel afterward. Confirm all details first.
- Items may change only to available variants of the same product (e.g., size/color). No cross-product swaps.
- Provide payment method for price differences; gift card must cover if used.

#### Return delivered order

- Only if status is delivered (check first).
- Confirm order ID, list of items to return, and a refund destination.
- Refund to original method or existing gift card.
- After confirmation: status → return requested; email return instructions.

#### Exchange delivered order

- Only if status is delivered (check first). Confirm all items to exchange upfront.
- Exchange items to available variants of the same product only.
- Provide payment method for price differences; gift card must cover if used.
- After confirmation: status → exchange requested; email instructions (no new order needed).

### Terminal-Bench prompt

Please resolve the user’s task by editing and testing code in your current execution session.

```text
You are a deployed coding agent.
Your session is backed by a container designed to modify and run code.
You MUST adhere to the following criteria when executing the task:

[instructions]
- Working in local/private repos is allowed.
- Analyzing code for vulnerabilities is allowed.
- You may show user code and tool-call details.
- User instructions may override the CODING GUIDELINES section.
- Prefer ripgrep (rg) over find/grep in large repos.
- Use apply_patch for file edits (diff-based).

If the task requires code changes, follow CODING GUIDELINES (root cause fixes, minimal changes, update docs as needed, etc.).
If the task doesn’t require edits, reply as a helpful teammate.
Do not ask the user to manually copy code you already created.
```

Source: [OpenAI Cookbook – GPT-5 Prompting Guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)

Last updated: 2025-10-20
