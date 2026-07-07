# 01 — De-duplicate an array, preserving first-occurrence order

**Category:** Algorithms / data handling
**Prompt given to both models:**

> Write a JavaScript function `dedupe(arr)` that returns a new array with duplicate
> values removed, preserving the order of first occurrence. It should work for arrays
> of primitive values (numbers, strings, booleans).

---

## Model A

```javascript
function dedupe(arr) {
  return [...new Set(arr)];
}
```

## Model B

```javascript
function dedupe(arr) {
  const seen = {};
  const out = [];
  for (const x of arr) {
    if (!seen[x]) {
      seen[x] = true;
      out.push(x);
    }
  }
  return out;
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 2 | B collapses distinct values via object-key coercion |
| Edge-case handling | 5 | 2 | B mishandles `0`, `false`, and `1` vs `"1"` |
| Idiomatic style & readability | 5 | 4 | A is the canonical idiom; B is fine but verbose |
| Efficiency | 5 | 5 | Both O(n) |
| Security & robustness | 4 | 4 | Neither validates that input is an array |
| Instruction-following | 5 | 5 | Both return a new array, preserve order |
| **Total (/30)** | **29** | **22** | |

## Comparison

Both are O(n) and both preserve insertion order, so on the surface they look close. The
decisive difference is **correctness on the exact input class the prompt names** — primitives.

Model B uses a plain object as its "seen" set, which forces every key through string
coercion. Two concrete failures follow:

- **Type collision:** `dedupe([1, "1"])` returns `[1]` for B — the number `1` and string
  `"1"` become the same key `"1"`. A correctly returns `[1, "1"]`.
- **Falsy-value bug:** the guard `if (!seen[x])` is truthiness-based, but that's not the
  actual bug source here since `seen[x]` starts `undefined` — the real defect is the coercion
  above. (Worth noting the `!seen[x]` pattern is fragile and would bite if someone "optimised"
  it to store values instead of `true`.)

Model A delegates to `Set`, which uses SameValueZero equality — no coercion, `0`/`false`/`""`
stay distinct, and `1` ≠ `"1"`. It's also the idiomatic one-liner a reviewer expects.

Neither validates that `arr` is iterable; both throw on `dedupe(null)`. That's a shared minor,
so it doesn't separate them.

## Verdict

**Winner: A**

A wins on **correctness**, which overrides B's respectable structure. B silently returns wrong
results for mixed-type primitive arrays — exactly the input the prompt calls out — because object
keys coerce to strings. That's a major, data-corrupting bug, not a style nit. What would flip
the call: if the prompt guaranteed a single primitive type *and* B added an array-type guard,
the two would be near-equal and A would win only on idiom.
