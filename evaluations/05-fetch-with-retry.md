# 05 — `fetchWithRetry` with exponential backoff

**Category:** Robustness
**Prompt given to both models:**

> Write `fetchWithRetry(url, { retries = 3, baseDelay = 300 })` that calls `fetch(url)` and,
> on failure, retries up to `retries` times with exponential backoff starting at `baseDelay`.
> Treat a non-OK HTTP status as a failure. After the final attempt fails, throw.

---

## Model A

```javascript
async function fetchWithRetry(url, { retries = 3, baseDelay = 300 } = {}) {
  let lastErr;
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res;
    } catch (err) {
      lastErr = err;
      if (attempt === retries) break;
      const delay = baseDelay * 2 ** attempt + Math.random() * 100;
      await new Promise(r => setTimeout(r, delay));
    }
  }
  throw lastErr;
}
```

## Model B

```javascript
async function fetchWithRetry(url, options) {
  const retries = options.retries;
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url);
    } catch (e) {
      console.log("retrying");
    }
  }
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 1 | B has no backoff, treats 500 as success, returns `undefined` |
| Edge-case handling | 5 | 1 | B throws immediately when called without an options object |
| Idiomatic style & readability | 5 | 2 | B logs from a library and swallows the error |
| Efficiency | 5 | 2 | B retries in a tight loop with no delay — hammers the server |
| Security & robustness | 4 | 1 | A adds jitter + a real throw; A nit: retries non-retryable 4xx |
| Instruction-following | 5 | 1 | B breaks all four explicit requirements |
| **Total (/30)** | **29** | **8** | |

## Comparison

The prompt sets four explicit requirements: **(1)** exponential backoff starting at
`baseDelay`, **(2)** treat a non-OK HTTP status as failure, **(3)** throw after the final
attempt, and **(4)** the two default parameters. Model B fails every one:

- **Defaults break the no-arg call.** `const retries = options.retries` dereferences
  `options` directly, so `fetchWithRetry(url)` throws `TypeError: Cannot read properties of
  undefined` before a single request is made. A uses `= {}` plus destructuring defaults, so
  the documented `retries = 3, baseDelay = 300` actually apply.
- **No backoff at all.** B loops with no delay, so on failure it re-requests immediately —
  turning a retry policy into a tight retry storm against the server. A computes
  `baseDelay * 2 ** attempt` (300 → 600 → 1200 ms) and adds jitter to avoid a thundering herd.
- **A non-OK status is treated as success.** `fetch` only rejects on a network error, not on
  an HTTP 4xx/5xx. B does `return await fetch(url)`, so a `500` is returned as if it
  succeeded. A explicitly checks `if (!res.ok) throw`, so a bad status triggers a retry as
  required.
- **No throw on exhaustion.** When every attempt fails, B falls out of the loop and returns
  `undefined` — the caller gets a silent non-value instead of an error. A tracks `lastErr`
  and re-throws it after the last attempt.

B also swallows the real error into `console.log("retrying")` — no delay, no context, and
logging from inside a library is itself a smell.

Model A is near-flawless. The one honest nit keeping it off a perfect sheet: it retries on
*any* failure, including non-retryable client errors like `400`/`404`, and sets no
per-request timeout — for a hardened utility you'd usually retry only on network errors and
5xx, and cap each attempt. Neither is required by the prompt.

## Verdict

**Winner: A** — decisively.

Model B breaks all four stated requirements: it throws on the documented no-argument call,
never backs off, treats HTTP 500 as success, and returns `undefined` instead of throwing when
retries are exhausted — any one of which is disqualifying for a retry utility. Under the
correctness/robustness weighting the verdict isn't close. Model A implements the full spec and
adds jitter as a sensible bonus. What would flip the call: nothing achievable by polish — B
would need backoff, an `res.ok` check, a real `throw`, and working defaults, i.e. a rewrite
into A. The only refinement A itself invites is restricting retries to network/5xx failures
and adding a timeout.
