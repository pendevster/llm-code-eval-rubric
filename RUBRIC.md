# Evaluation Rubric

Each candidate answer is scored **1–5 on six axes**. Scores inform — but do not mechanically
decide — the pairwise verdict.

## The six axes

1. **Correctness** — Does it solve the stated problem and return the right result for typical,
   valid inputs? Does it meet every explicit requirement in the prompt?
2. **Edge-case handling** — Behaviour on empty / null / boundary / malformed / unexpected input.
   Does it fail loudly and safely, or silently produce wrong output?
3. **Idiomatic style & readability** — Language conventions, naming, structure, and clarity.
   Would a competent reviewer merge this without stylistic rework?
4. **Efficiency** — Appropriate time and space complexity for the problem. No accidental O(n²)
   where O(n) is natural; no needless allocation or repeated work.
5. **Security & robustness** — Input validation, injection / authorization safety, secret
   handling, and error handling. Absence of unsafe patterns (string-built SQL, plaintext
   secrets, unbounded input, swallowed errors).
6. **Instruction-following** — Did it do exactly what was asked — the requested signature,
   return shape, format, and constraints — without scope drift?

## Scale (per axis)

| Score | Meaning |
|-------|---------|
| 5 | Excellent — no issues worth noting |
| 4 | Good — a minor nit that wouldn't block a merge |
| 3 | Acceptable — a real gap, but the core intent is met |
| 2 | Weak — a substantive problem a user would hit |
| 1 | Poor — incorrect, unsafe, or ignores the prompt |

## Deciding the verdict

1. Record per-axis scores for both candidates and note the **total** (out of 30).
2. Totals are a starting point, not the ruling. Apply two override rules:
   - **A correctness failure loses.** An answer that returns wrong results for valid input
     cannot win on style or brevity.
   - **A security failure loses** where the prompt implies untrusted input (an API handler,
     a query builder, anything touching user data), even against a more elegant answer.
3. If neither override applies, the higher total wins; within ~1–2 points, call it a **tie**
   and say what would break the tie.
4. Write a **2–4 sentence justification** naming the single most decision-relevant difference.
   The justification is the deliverable — a training pipeline needs the *reason*, not just the pick.

## Writing good rationale

- Point to the specific line or behaviour, not a vague impression.
- Separate **major** issues (wrong output, injection, data loss) from **minor** ones
  (naming, a redundant variable). Don't let a minor issue outweigh a major one.
- State what would *change* your verdict — this exposes your calibration.
