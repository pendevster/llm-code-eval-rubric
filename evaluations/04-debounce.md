# 04 — Implement `debounce`

**Category:** Concurrency / real-time
**Prompt given to both models:**

> Implement a `debounce(fn, wait)` function in JavaScript. The returned function should
> delay calling `fn` until `wait` ms have passed since the last call, and should preserve
> the `this` context and arguments of the most recent call.

---

## Model A

```javascript
function debounce(fn, wait) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), wait);
  };
}
```

## Model B

```javascript
function debounce(fn, wait) {
  let timer;
  return (...args) => {
    timer = setTimeout(() => fn(...args), wait);
  };
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 1 | B never clears the timer — it doesn't debounce |
| Edge-case handling | 4 | 2 | Rapid calls: A collapses to one; B fires all |
| Idiomatic style & readability | 5 | 4 | Both readable; B's arrow hides a `this` trap |
| Efficiency | 5 | 3 | B leaks a timer per call |
| Security & robustness | 4 | 4 | Neither validates `wait`; shared minor |
| Instruction-following | 5 | 2 | B breaks both explicit requirements |
| **Total (/30)** | **28** | **16** | |

## Comparison

The prompt states two hard requirements: **(1)** collapse rapid calls into one after `wait`,
and **(2)** preserve `this` and the latest arguments. Model B fails both:

- **No debouncing.** B never calls `clearTimeout`, so every invocation schedules its own
  timer. Ten rapid calls fire `fn` ten times — this is throttling-that-isn't, not debouncing.
  This is a correctness failure against the primary requirement.
- **Lost `this`.** B's returned function is an arrow, so it has no own `this`; and it calls
  `fn(...args)` rather than `fn.apply(this, args)`. If `fn` is a method relying on `this`,
  the context is wrong.

Model A clears the pending timer on each call (so only the last one survives), uses a regular
`function` to capture the call-site `this`, and forwards it via `fn.apply(this, args)`. It
correctly preserves the most-recent arguments because `args` is rebound on every call.

Shared minor: neither guards against a non-numeric `wait`, and neither exposes a `cancel()`
method (not required by the prompt).

## Verdict

**Winner: A**

A satisfies both stated requirements; B satisfies neither. B's missing `clearTimeout` means it
doesn't actually debounce — the single behaviour the function exists to provide — so under the
correctness-override rule it loses regardless of how clean it looks. The `this` handling is the
secondary but still explicit miss. What would flip the call: if the prompt had asked for a
plain "delay each call by `wait`" with no context preservation, B would be closer — but it would
still need the timer-clearing to match A.
