---
name: gpt-oss-20b-prompts
description: Write, modify, and optimize system prompts for GPT OSS 20B. Use when crafting developer messages, choosing reasoning effort, defining tools, structuring instructions, or debugging prompt issues for gpt-oss-20b or gpt-oss-120b.
---

# Prompts for GPT OSS 20B

Write system prompts that exploit the model's harmony format, its instruction hierarchy, and the empirical quirks of a 3.6B-active MoE reasoner. The leading word is **harmony** — every decision flows from what the harmony format rewards.

## Steps

### 1. Clarify the deployment context

Before writing, establish:
- **Inference method**: API provider (handles harmony automatically) vs. raw inference (you must format harmony tokens yourself)?
- **Use case category**: chat, agentic/tool-use, structured output, or reasoning-heavy?
- **Latency budget**: determines reasoning effort level.

Completion criterion: you know whether you're writing a developer message string (API scenario) or a full harmony prompt (raw inference), and which channel/tool features the prompt needs.

### 2. Draft the prompt in the correct harmony slot

Place content in the right message based on the hierarchy (see [HARMONY-FORMAT.md](HARMONY-FORMAT.md)):

| Content type | Goes in | Why |
|---|---|---|
| Identity override | `model_identity` kwarg or system message | Highest priority |
| Reasoning effort | System message: `Reasoning: low/medium/high` | Model-level control |
| Instructions, persona, rules | Developer message under `# Instructions` | Correct slot per hierarchy |
| Function/tool definitions | Developer message under `# Tools` | Format requires TypeScript-like syntax in namespace |
| Output schema | Developer message under `# Response Formats` | Structured output slot |

Rules for the developer message body:
- Open with `# Instructions` header.
- Use markdown `#`/`##` headers to separate concerns — the model attends to structure.
- Front-load the most critical constraints in the first paragraph; attention dilutes over length.
- State behavioral rules as directives, not descriptions ("Respond in JSON" not "You should try to respond in JSON").
- Never embed raw harmony tokens (`<|start|>`, `<|channel|>`, etc.) inside instruction text — these are structural, not content.

Completion criterion: the prompt is placed in the correct harmony slot(s) and uses proper structural headers.

### 3. Select reasoning effort

Choose based on empirical findings for the 20B model:

| Effort | When to use |
|---|---|
| `low` | Default for most tasks. On 20B, low often matches 120B-medium quality at far lower latency. Best for classification, extraction, simple QA, chat. |
| `medium` | Multi-step reasoning, code generation, complex analysis where you need the CoT but speed still matters. |
| `high` | Mathematical proof, adversarial robustness, multi-hop reasoning over long contexts. Rarely improves quality on 20B — use only when medium measurably fails. |

The setting goes in the system message as `Reasoning: low`. If using an API that exposes `reasoning_effort` as a parameter, set it there instead of manually.

Completion criterion: reasoning effort is chosen and documented with rationale.

### 4. Define tools (if applicable)

Use the TypeScript-like syntax the model was trained on:

```
# Tools

## functions

namespace functions {

// Description of what the tool does
type tool_name = (_: {
// Description of param
param_name: string,
optional_param?: "option_a" | "option_b", // default: option_a
}) => any;

} // namespace functions
```

Rules:
- Argument is always named `_` with an inline type.
- No-arg functions: `type name = () => any;`
- Use `//` comments for descriptions — one line above the field.
- Return type is always `any`.
- Keep an empty line between function definitions.
- Add to system message: `Calls to these tools must go to the commentary channel: 'functions'.`

Completion criterion: every tool the model needs is defined in the correct TypeScript syntax inside a namespace block.

### 5. Apply the pitfall checklist

Review the draft against known failure modes of GPT OSS 20B:

- **Quant fever**: Does the prompt contain hard numerical targets (percentages, counts, thresholds)? If so, qualify them with contextual constraints or replace with qualitative guidance. The model fixates on numbers and may override safety/quality to hit them.
- **Context dilution**: Is the prompt longer than ~2K tokens of instructions? If so, restructure — move reference material behind tool calls or split into multi-turn. Critical rules must live in the first 500 tokens of the developer message.
- **Reasoning blackholes**: Does the task have circular dependencies or unbounded search? Add explicit termination conditions ("stop after 3 attempts", "if uncertain, output your best guess").
- **Compliance ambiguity**: Are constraints phrased as suggestions ("try to", "ideally")? Rewrite as imperatives. The model interprets soft language as optional.
- **Identity confusion**: Did you override identity in the developer message AND system message? Pick one location. The developer message `# Instructions` section is the right place for persona; leave the system identity as default unless you have a specific reason.

Completion criterion: every item checked, violations fixed.

### 6. Validate format correctness

If writing raw harmony (not going through an API):
- System message: opens with `<|start|>system<|message|>`, ends with `<|end|>`.
- Developer message: opens with `<|start|>developer<|message|>`, ends with `<|end|>`.
- No whitespace between `<|start|>` and the role name.
- Channel declaration in system: `# Valid channels: analysis, commentary, final. Channel must be included for every message.`
- Tool routing note present if functions defined.

If writing for an API/framework (Ollama, vLLM, Transformers):
- Instructions go in the first message with role `developer` or `system` (the chat template maps `system` role in messages array to the developer slot).
- Tools go in the `tools` parameter as JSON schemas — the chat template renders them to TypeScript automatically.

Completion criterion: the prompt can be fed to the model or API without format errors.

## Reference

### The 20B efficiency advantage

GPT OSS 20B activates only 3.6B of its 21B parameters per token (4 of 128 experts via MoE routing). Empirical benchmarks show it often matches or exceeds gpt-oss-120b on tasks like HumanEval and MMLU despite being 6x smaller in active compute. The practical implication: prefer shorter, crisper prompts over verbose ones — the model's limited active capacity benefits from concise, well-structured input more than larger models do.

### Instruction hierarchy

The model enforces a strict priority: **system > developer > user > assistant > tool**. When instructions conflict across levels, higher priority wins. This means:
- Safety constraints in the system message cannot be overridden by developer or user messages.
- Developer instructions override anything a user says.
- Never put application logic in the system message — it's for identity, dates, reasoning config, and built-in tools only.

### Context window

131K tokens maximum (YaRN position embeddings). However, attention quality degrades over very long contexts due to the MoE routing overhead and sliding window attention (window size 128). For best results, keep the combined system+developer message under 4K tokens and put the densest, most important instructions early.

### Channel semantics

- `analysis`: chain-of-thought reasoning (never show to end users — not safety-trained)
- `commentary`: tool calls, preambles to multi-step actions
- `final`: user-facing responses (safety-trained, show this to users)

### Sources

- [HuggingFace model card](https://huggingface.co/openai/gpt-oss-20b)
- [Harmony format specification](https://github.com/openai/harmony/blob/main/docs/format.md)
- [OpenAI blog: Introducing gpt-oss](https://openai.com/index/introducing-gpt-oss/)
- [arXiv 2508.10925: Model Card](https://arxiv.org/abs/2508.10925)
- [arXiv 2509.23882: Quant Fever, Reasoning Blackholes (failure modes)](https://arxiv.org/abs/2509.23882)
- [DataRobot: reasoning effort benchmarks](https://www.datarobot.com/blog/testing-gpt-oss-models/)
- [arXiv 2512.04254: CoT assessment at different reasoning levels](https://arxiv.org/html/2512.04254)
