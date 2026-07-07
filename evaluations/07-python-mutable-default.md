# 07 — Append to an optional list (Python)

**Category:** Language idiom / Python
**Prompt given to both models:**

> Write a Python function `append_to(element, target)` where `target` is an optional list.
> Append `element` to `target` and return it; if the caller doesn't provide a list, start
> from a fresh empty one.

---

## Model A

```python
def append_to(element, target=None):
    if target is None:
        target = []
    target.append(element)
    return target
```

## Model B

```python
def append_to(element, target=[]):
    target.append(element)
    return target
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 2 | B works once, then silently accumulates across calls |
| Edge-case handling | 5 | 1 | B's "fresh" list is shared state — the core failure mode |
| Idiomatic style & readability | 4 | 2 | A uses the `None` sentinel; B is a textbook anti-pattern. A nit: no type hints |
| Efficiency | 5 | 5 | Both O(1) append |
| Security & robustness | 5 | 2 | B leaks state between unrelated callers |
| Instruction-following | 5 | 2 | "start from a fresh empty one" — B doesn't, after the first call |
| **Total (/30)** | **29** | **14** | |

## Comparison

This prompt is a single, deliberate Python trap: **the default value of a parameter is
evaluated once, at function-definition time, not on each call.** Model B falls straight into it.

- **The shared-default bug.** `target=[]` creates *one* list, reused for every call that omits
  `target`. So the "fresh empty" list isn't fresh:

  ```python
  append_to(1)   # -> [1]
  append_to(2)   # -> [1, 2]   (expected [2])
  append_to(3)   # -> [1, 2, 3]
  ```

  The function works exactly once and is wrong forever after. This is the single most
  well-known Python footgun; every linter (`pylint` W0102, Ruff `B006`) flags it on sight.
- **Cross-caller state leak.** Because the list is module-level state hidden in a default,
  two unrelated call sites that both rely on the default share and mutate the same list — a
  latent data-leak/robustness bug, not merely a wrong return value.

Model A uses the canonical fix: default `target=None`, then `if target is None: target = []`
allocates a genuinely new list per call. It behaves correctly on the first call and every call
after, and a caller who *does* pass a list gets that list appended to and returned, as asked.

The one honest nit against A: no type hints or docstring. For a throwaway snippet that's fine,
but production Python would write `def append_to(element, target: list | None = None) -> list:`
— which is why A takes a 4, not a 5, on idiomatic style. That's the only thing separating it
from a perfect sheet.

## Verdict

**Winner: A**

A allocates a new list per call via the `None`-sentinel idiom; B shares one list created at
definition time, so its "fresh empty" list silently accumulates values across calls and leaks
state between unrelated callers. Under the correctness override, a function that returns the
wrong result on its second invocation cannot win on brevity — and B is two characters shorter
precisely *because* it skips the guard that makes it correct. What would flip the call: nothing
short of B switching its default to `None` and allocating inside the body, i.e. becoming A.
