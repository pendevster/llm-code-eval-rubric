# 10 — Title-case the words in a string

**Category:** Idiom / judgment (close call)
**Prompt given to both models:**

> Write `capitalizeWords(str)` that uppercases the first letter of each word in a string and
> leaves the rest untouched. E.g. `"hello world"` → `"Hello World"`.

---

## Model A

```javascript
function capitalizeWords(str) {
  return str.replace(/\b\w/g, (c) => c.toUpperCase());
}
```

## Model B

```javascript
function capitalizeWords(str) {
  return str
    .split(" ")
    .map((w) => w.charAt(0).toUpperCase() + w.slice(1))
    .join(" ");
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 4 | 4 | Both meet the example; each mishandles a *different* input class |
| Edge-case handling | 4 | 4 | A over-capitalizes after `'`/`-`; B misses tab/newline delimiters |
| Idiomatic style & readability | 4 | 5 | B's split/map/join reads at a glance; A's regex is terser but denser |
| Efficiency | 5 | 4 | A single-passes; B allocates an intermediate array |
| Security & robustness | 4 | 4 | `\b\w` is linear (no ReDoS); neither validates non-string input |
| Instruction-following | 5 | 5 | Both match the signature and the stated example |
| **Total (/30)** | **26** | **26** | |

## Comparison

Unlike the rest of this set, there's no planted flaw here — both answers are competent, correct
on the prompt's example, and defensible. They diverge only on inputs the prompt leaves
unspecified, and they fail on *different* ones, which is what makes this a genuine tie rather
than a disguised winner.

- **Delimiters.** A's `\b\w` fires at every word boundary, so it capitalizes across any
  separator — spaces, tabs, newlines, hyphens: `"hello-world"` → `"Hello-World"`,
  `"a\tb"` → `"A\tB"`. B splits on a literal single space, so tab- or newline-separated words
  aren't capitalized (`"a\tb"` → `"A\tb"`) and runs of multiple spaces produce empty segments.
- **Apostrophes.** The mirror image: A treats the letter after an apostrophe as a new word,
  so `"don't"` → `"Don'T"` — an over-capitalization bug. B, splitting only on spaces, leaves
  `"don't"` → `"Don't"` correctly.
- **Readability.** B's `split/map/join` pipeline is the more legible to a general reviewer; A
  is more compact but assumes the reader is fluent in `\b`. That's B's one clear edge.
- **Efficiency.** A does it in a single regex pass; B allocates an intermediate array. Real but
  negligible at any realistic string length. That's A's one clear edge.

The two clear edges — B on readability, A on efficiency — cancel, and each carries an
equal-and-opposite correctness gap on unspecified input. The totals land at 26–26 honestly, not
by construction.

## Verdict

**Winner: Tie**

Neither answer dominates: they trade a readability edge (B) for an efficiency edge (A), and each
mishandles a different unspecified input class — A over-capitalizes after apostrophes, B ignores
non-space delimiters. On the prompt as literally written (space-separated words, the given
example) they are equivalent, so per the rubric's ±1–2 rule this is a tie. What would break it:
**pin down the input contract.** If the text can contain apostrophes or names like `"O'Brien"`,
B's predictable per-space behaviour wins; if it can contain tabs/newlines or hyphenated
compounds that must each be capitalized, A's boundary-based approach wins. Absent that
specification, picking one over the other would be false precision.
