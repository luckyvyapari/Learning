# Humanize Text Prompt

Source: https://huggingface.co/jialinyyzz/humanizer

## Usage

No system prompt. Text completion only. Build prompt as:

```
prompt = instruction + "\n\n" + draft.strip() + "\n\n### Rewritten:\n\n"
```

## Instruction

```
Rewrite the text below so it reads like a person wrote it, not a language model.

Reorganize it as you see fit. Vary sentence length on purpose. Cut hedging,
throat-clearing, and any sentence that only announces what comes next.
Prefer the concrete word over the abstract one. It is fine to sound uneven.

Every fact, number, unit, date, name and quotation must survive unchanged.

<YOUR DRAFT>

### Rewritten:

```

## Sampling Parameters

| Param | Value |
|-------|-------|
| temperature | 1.0 |
| top_p | 0.95 |
| top_k | 0 |
| min_p | 0 |
| repeat_penalty | 1.0 |
| num_ctx | 8192 |
| num_predict | 2048 |

## Notes

- Same English instruction works for Chinese drafts too
- Stop on EOS only — do NOT use `###` as a stop string
- SHA256 check: first 16 hex chars of `sha256(prompt for draft "X")` = `cc51d66b4c593fbe`
