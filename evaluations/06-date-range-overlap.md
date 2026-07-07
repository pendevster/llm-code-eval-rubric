# 06 — Date-range overlap check

**Category:** Edge cases
**Prompt given to both models:**

> Write `rangesOverlap(a, b)` where `a` and `b` are ranges of the shape `{ start, end }`
> (numbers or comparable dates). Return `true` if the two ranges overlap, treating ranges
> that merely touch at an endpoint as overlapping.

---

## Model A

```javascript
function rangesOverlap(a, b) {
  if (a.start > a.end || b.start > b.end) {
    throw new Error("range start must be <= end");
  }
  return a.start <= b.end && b.start <= a.end;
}
```

## Model B

```javascript
function rangesOverlap(a, b) {
  return (a.start >= b.start && a.start <= b.end) ||
         (a.end >= b.start && a.end <= b.end);
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 2 | B returns `false` when one range fully contains the other |
| Edge-case handling | 5 | 2 | B misses containment; no inverted-range handling |
| Idiomatic style & readability | 5 | 3 | A is the canonical formula; B's endpoint check reads plausible but is wrong |
| Efficiency | 5 | 5 | Both O(1) |
| Security & robustness | 5 | 3 | A validates inverted ranges; B silently returns wrong answers |
| Instruction-following | 4 | 4 | Both match the signature and treat touching as overlap; A adds an unasked guard |
| **Total (/30)** | **29** | **19** | |

## Comparison

Both answers look reasonable and both handle the touching-endpoint case the prompt calls out,
so the score sheet hinges on a case neither the prompt nor B's author traced: **full
containment**.

- **The containment bug.** B tests only whether one of `a`'s endpoints falls inside `b`. When
  `b` sits entirely inside `a`, neither of `a`'s endpoints is inside `b`, so B returns `false`
  for ranges that plainly overlap. Concretely, `rangesOverlap({start:1,end:10},
  {start:3,end:4})` returns **`false`** for B and **`true`** for A. The bug is also asymmetric:
  the mirror call `({start:3,end:4}, {start:1,end:10})` happens to return `true`, so B's result
  depends on argument order — a subtle, nasty defect.
- **Touching endpoints.** Both satisfy the explicit requirement. `a=[1,5]`, `b=[5,9]` → both
  return `true`. This is where B's structure looks fine and lulls a reader into trusting it.
- **The canonical formula.** A uses `a.start <= b.end && b.start <= a.end`, the standard
  interval-overlap test. It is correct for every arrangement — disjoint, touching, partial
  overlap, and either direction of containment — and is the one-liner a reviewer expects.
- **Inverted-range guard.** A additionally throws on `start > end`, failing loudly on malformed
  input rather than returning a misleading boolean. B has no such guard.

## Verdict

**Winner: A**

A implements the canonical, order-independent overlap test and is correct on every case; B
carries a genuine correctness bug — it returns `false` when one range fully contains the other,
and its answer depends on argument order. The single input that exposes it is
`rangesOverlap({start:1,end:10}, {start:3,end:4})`: the ranges clearly overlap, yet B says they
don't. Under the correctness override that decides the verdict on its own. A's inverted-range
`throw` is a small scope addition the prompt didn't request — defensible robustness, and the
only thing keeping its instruction-following off a 5. What would flip the call: nothing short of
B switching to the `start <= end && start <= end` form, i.e. becoming A.
