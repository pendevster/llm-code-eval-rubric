# 02 — Parse a URL query string

**Category:** Parsing / idiom
**Prompt given to both models:**

> Write `parseQuery(qs)` that turns a URL query string (e.g. `"?a=1&b=hello%20world&a=2"`)
> into an object. Decode percent-encoding, and represent a key that appears more than once
> as an array of its values.

---

## Model A

```javascript
function parseQuery(qs) {
  const params = new URLSearchParams(qs.startsWith("?") ? qs.slice(1) : qs);
  const out = {};
  for (const [key, value] of params) {
    if (key in out) {
      out[key] = [].concat(out[key], value);
    } else {
      out[key] = value;
    }
  }
  return out;
}
```

## Model B

```javascript
function parseQuery(qs) {
  const out = {};
  qs.replace(/^\?/, "").split("&").forEach(pair => {
    const [key, value] = pair.split("=");
    out[key] = value;
  });
  return out;
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 1 | B ignores both explicit requirements |
| Edge-case handling | 5 | 2 | B truncates values containing `=`; bare key → `undefined` |
| Idiomatic style & readability | 5 | 3 | A uses the platform parser; B abuses `.replace` for iteration |
| Efficiency | 5 | 5 | Both O(n); not a differentiator |
| Security & robustness | 4 | 3 | Neither validates input type or guards `__proto__` keys |
| Instruction-following | 5 | 1 | B satisfies neither of the two stated instructions |
| **Total (/30)** | **29** | **15** | |

## Comparison

The prompt states two explicit requirements — **decode percent-encoding** and **collapse
repeated keys into an array**. This makes the evaluation almost purely about instruction-
following, and Model B misses both:

- **No decoding.** B never percent-decodes. On the prompt's own example, `b=hello%20world`
  yields the literal `"hello%20world"` instead of `"hello world"`. Model A delegates to
  `URLSearchParams`, which decodes `%20` (and `+`) correctly.
- **Repeated keys are lost, not arrayed.** For `?a=1&a=2`, B assigns `out.a = "1"` then
  overwrites it with `"2"` — last-write-wins, so the first value is silently dropped. A
  detects the existing key (`key in out`) and builds `["1", "2"]`, extending correctly to
  three-plus occurrences via `[].concat`.
- **Values containing `=` are truncated.** B does `pair.split("=")` with a two-element
  destructure, so a base64-ish value like `token=abc==` becomes `key="token", value="abc"`
  — everything after the first `=` is discarded. `URLSearchParams` splits on the first `=`
  only and keeps the remainder intact.

Model B does one thing acceptably — stripping a leading `?` — but the rest is a hand-rolled
parser that reimplements (and gets wrong) what the platform already provides. Its use of
`String.prototype.replace` purely to iterate for side effects is a recognised anti-idiom.

Shared minor: neither validates that `qs` is a string (`qs.startsWith`/`qs.replace` throw on
`null`), and neither guards against a `__proto__` key. These don't separate the two.

## Verdict

**Winner: A** — decisively.

The prompt names exactly two behaviours and Model B implements neither: it doesn't decode
percent-encoding and it drops repeated keys instead of arraying them, so it returns wrong
output on the prompt's own example string. Under the correctness/instruction-following
weighting this is disqualifying regardless of B's brevity. Model A is also the idiomatic
answer — reaching for `URLSearchParams` rather than re-deriving a fragile splitter. What
would flip the call: nothing short of B adding `decodeURIComponent`, splitting on only the
first `=`, and accumulating repeats into an array — at which point it converges on A.
