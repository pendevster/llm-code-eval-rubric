# LLM Code-Evaluation Rubric — Worked Examples

A small, hand-built set of **pairwise code evaluations**: for each coding prompt,
two candidate answers ("Model A" and "Model B") are scored against a fixed rubric,
compared axis by axis, and a winner is chosen with written justification.

This is the core task shape in coding RLHF / AI code-evaluation work: read a prompt,
read two model responses, decide which is better, and explain *why* in terms a training
pipeline can act on.

## Why this exists

I build production software (React / Next.js / Node.js) and hold an MSc in Cyber Security.
Day to day that means judging code for correctness, security, efficiency, and clarity —
the same judgement code-evaluation work depends on. This repo makes that judgement explicit
and reproducible.

## How to read it

- **[`RUBRIC.md`](RUBRIC.md)** — the evaluation framework: the six axes, the 1–5 scale, and
  how a pairwise verdict is decided (totals inform the call; a correctness or security failure
  can override them).
- **[`evaluations/`](evaluations/)** — one file per prompt. Each contains the prompt, both
  candidate answers, a per-axis score table for each, and the comparison + verdict.
- **[`TEMPLATE.md`](TEMPLATE.md)** — blank evaluation template for adding new items.

## Coverage

| # | Prompt | Category | Primary axes exercised |
|---|--------|----------|------------------------|
| 01 | De-duplicate, preserve order | Algorithms / data | Correctness, edge cases, efficiency |
| 02 | Parse a URL query string | Parsing / idiom | Idiomatic style, edge cases |
| 03 | Express `POST /users` handler | API / **security** | Security, correctness, instruction-following |
| 04 | Implement `debounce` | Concurrency / real-time | Correctness, idiomatic style |
| 05 | `fetchWithRetry` + backoff | Robustness | Security & robustness, correctness |
| 06 | Date-range overlap check | Edge cases | Edge cases, correctness |
| 07 | Append to an optional list (**Python**) | Language idiom | Correctness, edge cases, idiomatic style |
| 08 | Read a `key=value` config file (**Go**) | Error handling | Security & robustness, edge cases, instruction-following |
| 09 | Compare a client-supplied API token | Security / **override** | Security override beats surface elegance (winner: B) |
| 10 | Title-case the words in a string | Idiom / **close call** | Tie-handling, edge cases (verdict: tie) |

## A note on method

Every score and rationale reflects my own judgement. Most candidate pairs were produced as
evaluation *stimuli* (one deliberately stronger, one with a planted flaw) so the rubric has
something to bite on; the *evaluations themselves* are the work. Not every pair is lopsided,
by design: **09** is a case where the more elegant answer loses on a security override (so the
override rule earns its place, rather than just agreeing with the higher total), and **10** is
two genuinely competent answers that resolve to a tie — the harder, more realistic judgement
call.

## Stack-agnostic

Most examples are in JavaScript/TypeScript and Node, with one **Python** (07) and one **Go**
(08) evaluation applying the same six axes unchanged — evidence that the rubric is language-
neutral, not just claimed to be. It transfers to any language without modification.

## Author

**Abdullah Oyewale** — GitHub [@pendevster](https://github.com/pendevster).
Production software engineer (React / Next.js / Node.js), MSc in Cyber Security.
The rubric, scores, and rationale here are my own work. Feedback and issues welcome.
