# Harmony Response Format Reference

GPT OSS 20B was trained exclusively on the harmony response format. It will not work correctly without it. This reference covers the structural details needed when writing or reviewing prompts.

Sources: [Official format docs](https://github.com/openai/harmony/blob/main/docs/format.md), [openai-harmony PyPI package](https://pypi.org/project/oss-harmony/)

## Message Structure

Every message follows the pattern:

```
<|start|>{role}{header}<|message|>{content}<|end|>
```

Stop tokens: `<|return|>` (model done generating), `<|call|>` (tool call pending).

## Roles and Priority

| Role | Purpose | Priority |
|---|---|---|
| `system` | Identity, dates, reasoning effort, built-in tools, channel rules | Highest |
| `developer` | Instructions ("system prompt"), function tools, output schemas | High |
| `user` | User input | Medium |
| `assistant` | Model output (tagged with channel) | Low |
| `tool` | Tool response (uses tool name as sub-role) | Lowest |

## System Message Template

The minimal correct system message:

```
<|start|>system<|message|>You are ChatGPT, a large language model trained by OpenAI.
Knowledge cutoff: 2024-06
Current date: {YYYY-MM-DD}

Reasoning: {low|medium|high}

# Valid channels: analysis, commentary, final. Channel must be included for every message.
Calls to these tools must go to the commentary channel: 'functions'.<|end|>
```

Notes:
- The identity line ("You are ChatGPT...") should remain default unless you override via `model_identity` kwarg in the chat template. To change persona, use the developer message instead.
- The `Calls to these tools...` line is only needed when function tools are defined.
- Built-in tools (browser, python) are defined in the system message under `# Tools` if needed.

## Developer Message Template

```
<|start|>developer<|message|># Instructions

{your instructions here}

# Tools

## functions

namespace functions {

// {description}
type {name} = (_: {
// {param description}
{param}: {type},
}) => any;

} // namespace functions

# Response Formats

## {format_name}

// {description}
{JSON Schema}<|end|>
```

Sections are optional — include only what you need:
- `# Instructions` — always present if you have any behavioral guidance.
- `# Tools` — only if you define function tools.
- `# Response Formats` — only for structured output enforcement.

## Tool Definition Syntax

The model expects TypeScript-like type definitions:

```typescript
namespace functions {

// Gets current weather for a location.
type get_weather = (_: {
// City and state, e.g. "San Francisco, CA"
location: string,
// Temperature unit
format?: "celsius" | "fahrenheit", // default: celsius
}) => any;

// Lists recent notifications.
type get_notifications = () => any;

} // namespace functions
```

Rules:
- Single argument always named `_`.
- Inline the object type — don't reference external type definitions.
- Use `//` comment on the line above for field descriptions.
- Optional fields use `?:` suffix.
- Default values as trailing `// default: {value}` comments.
- Return type always `any`.
- Empty line between each function.
- Close with `} // namespace functions`.

## Channels

Assistant output is tagged with a channel in the header:

```
<|start|>assistant<|channel|>{channel}<|message|>{content}<|end|>
```

| Channel | Purpose | Show to user? |
|---|---|---|
| `analysis` | Chain-of-thought reasoning | No — not safety-trained |
| `commentary` | Tool calls, action preambles | Preambles yes, raw calls no |
| `final` | User-facing response | Yes |

## Tool Call Flow

1. Model outputs analysis (optional):
   ```
   <|channel|>analysis<|message|>{reasoning}<|end|>
   ```

2. Model calls tool:
   ```
   <|start|>assistant<|channel|>commentary to=functions.{name} <|constrain|>json<|message|>{args_json}<|call|>
   ```

3. You provide tool result:
   ```
   <|start|>functions.{name} to=assistant<|channel|>commentary<|message|>{result_json}<|end|>
   ```

4. Resume inference from `<|start|>assistant` — model continues with analysis or final answer.

Important: retain the analysis/commentary from the tool-call turn when feeding back for subsequent inference. Only drop CoT when the previous turn ended with a `final` channel message.

## Structured Output

Define in developer message:

```
# Response Formats

## shopping_list

// A list of items to buy
{"properties":{"items":{"type":"array","items":{"type":"string"}}},"type":"object"}
```

This influences the model but does not guarantee schema adherence — enforce via constrained decoding/grammar at inference time for strict compliance.

## Multi-Turn History Management

- After a `final` channel response: drop all previous `analysis` messages from history. Keep only `final` content.
- After a tool call (ended with `<|call|>`): retain the preceding `analysis` and `commentary` messages — the model needs them to continue its reasoning chain.
- Replace trailing `<|return|>` with `<|end|>` when storing messages in history.
