# 09 — Compare a client-supplied API token

**Category:** Security / API
**Prompt given to both models:**

> Write `verifyToken(provided, expected)` that checks whether a client-supplied API token
> (`provided`) matches the expected secret token (`expected`). Return `true` on a match.

---

## Model A

```javascript
function verifyToken(provided, expected) {
  return provided === expected;
}
```

## Model B

```javascript
const crypto = require("crypto");

function verifyToken(provided, expected) {
  const a = Buffer.from(String(provided), "utf8");
  const b = Buffer.from(String(expected), "utf8");
  return a.length === b.length && crypto.timingSafeEqual(a, b);
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 5 | Both return the right boolean for a genuine match/mismatch |
| Edge-case handling | 4 | 5 | B coerces non-strings and guards length; A is unguarded |
| Idiomatic style & readability | 5 | 3 | A is the obvious one-liner; B needs `crypto` + buffers |
| Efficiency | 5 | 4 | B allocates two buffers per call (negligible in practice) |
| Security & robustness | 1 | 5 | A compares a secret in non-constant time — timing side channel |
| Instruction-following | 5 | 5 | Both match the signature and return a boolean |
| **Total (/30)** | **25** | **27** | |

## Comparison

This is the case the rubric's **security override** exists for — and it's the one a shallow
review gets wrong. On pure craftsmanship, Model A is the better-looking answer: it's the
obvious one-liner, maximally readable, allocation-free, and it returns the correct result for
every functional test you'd write. A style-first scorer tallying readability and brevity would
pick A without hesitation.

That instinct is the trap. The prompt says **client-supplied** token compared against a
**secret** — untrusted input touching sensitive data — so the security axis is decisive:

- **Timing side channel.** `provided === expected` short-circuits at the first differing byte.
  The comparison therefore takes measurably longer the more leading bytes are correct. Against
  an endpoint an attacker can call repeatedly, that timing gradient leaks the secret one byte
  at a time — a classic, practical token/HMAC recovery attack. No amount of readability offsets
  a handler that leaks its own secret.
- **The fix is a constant-time compare.** Model B uses `crypto.timingSafeEqual`, whose runtime
  is independent of where the first mismatch falls. It coerces both operands to buffers and
  length-checks first (`timingSafeEqual` throws on unequal lengths).

Model B is not flawless, and it's worth naming the imperfection rather than scoring it as
perfect: the `a.length === b.length` pre-check still leaks the token's *length* through timing.
That's a known, accepted trade-off — length is far less sensitive than content, and
`timingSafeEqual` requires equal-length inputs — but it's the honest asterisk on B's security 5.

## Verdict

**Winner: B**

A is the cleaner, more readable solution and would win a review that weighed only style and
brevity — which is exactly why this pair matters. Comparing a secret token with `===` is a
timing-oracle vulnerability on an endpoint that takes untrusted input, and under the security
override that disqualifies A regardless of how much tidier it reads. B pays a small
readability and allocation cost to compare in constant time, which is the right trade for
authentication code. What would flip the call: if `provided` and `expected` were **not**
secrets — say, comparing two public identifiers — there'd be no timing channel to exploit and
A's simplicity would win outright.
