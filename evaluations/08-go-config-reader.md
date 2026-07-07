# 08 — Read a `key=value` config file (Go)

**Category:** Error handling / Go
**Prompt given to both models:**

> Write a Go function `readConfig(path string) (map[string]string, error)` that reads a file
> line by line and parses `key=value` pairs into a map. Skip blank lines and `#` comments,
> and handle errors appropriately.

---

## Model A

```go
func readConfig(path string) (map[string]string, error) {
	f, err := os.Open(path)
	if err != nil {
		return nil, err
	}
	defer f.Close()

	config := make(map[string]string)
	scanner := bufio.NewScanner(f)
	for scanner.Scan() {
		line := strings.TrimSpace(scanner.Text())
		if line == "" || strings.HasPrefix(line, "#") {
			continue
		}
		key, value, ok := strings.Cut(line, "=")
		if !ok {
			return nil, fmt.Errorf("invalid line: %q", line)
		}
		config[strings.TrimSpace(key)] = strings.TrimSpace(value)
	}
	if err := scanner.Err(); err != nil {
		return nil, err
	}
	return config, nil
}
```

## Model B

```go
func readConfig(path string) (map[string]string, error) {
	data, _ := os.ReadFile(path)
	config := make(map[string]string)
	lines := strings.Split(string(data), "\n")
	for _, line := range lines {
		parts := strings.Split(line, "=")
		config[parts[0]] = parts[1]
	}
	return config, nil
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 2 | B ignores the read error and never skips blanks/comments |
| Edge-case handling | 4 | 1 | B panics on a missing file, blank line, or line without `=`. A nit: dup keys silently overwrite |
| Idiomatic style & readability | 5 | 2 | `data, _ :=` is the canonical Go anti-pattern |
| Efficiency | 5 | 4 | A streams; B reads the whole file and re-splits each line |
| Security & robustness | 5 | 1 | B has an ignored error plus several panic vectors |
| Instruction-following | 5 | 2 | B skips neither blanks nor comments and never populates `error` |
| **Total (/30)** | **29** | **12** | |

## Comparison

The prompt has three explicit asks — parse `key=value`, **skip blanks and `#` comments**, and
**handle errors appropriately** — and Model B ignores all three while introducing runtime panics.

- **The discarded error.** `data, _ := os.ReadFile(path)` throws the error away. On a missing
  or unreadable file, `data` is `nil`, `string(data)` is `""`, and the loop reaches
  `parts[1]` on a one-element slice — an **index-out-of-range panic**. Discarding an `error`
  with `_` is the single most-flagged Go anti-pattern (`go vet` / `errcheck` catch it), and
  here it converts a recoverable failure into a crash. The function's `error` return is a lie:
  it is *always* `nil`.
- **Panics on ordinary input.** `strings.Split(line, "=")` followed by `parts[1]` panics on any
  line without an `=` — including the empty string produced by a trailing newline. A perfectly
  valid file ending in `\n` crashes B. Blank lines and `#` comments (which the prompt says to
  skip) also hit `parts[1]` and panic.
- **Silent truncation.** For a legitimate `token=abc=def` value, `strings.Split` yields three
  parts and `parts[1]` keeps only `abc` — the same first-`=` truncation the JS query-string
  case (02) exhibited.

Model A opens the file and `defer`s the close, streams with `bufio.Scanner` (bounded memory),
trims each line, explicitly `continue`s past blanks and `#` comments, splits on the first `=`
with `strings.Cut`, returns a descriptive `error` on a malformed line, and — importantly —
checks `scanner.Err()` so a mid-read I/O failure isn't swallowed. It never panics on hostile
input.

The one honest nit against A: duplicate keys silently overwrite (last-wins) with no warning.
That's a defensible default, but unflagged — which is why edge-case handling is a 4, not a 5.

## Verdict

**Winner: A**

A checks every error path and cannot panic on bad input; B discards the read error outright and
then panics on a missing file, a blank line, a comment, or merely a trailing newline — while
returning an `error` value that is always `nil`. Under the security/robustness override, a file
parser that crashes on untrusted or absent input and silently reports success is disqualified
regardless of its brevity. B also fails the two explicit "skip blanks and comments" instructions.
What would flip the call: nothing short of B checking the read error, guarding the split, and
skipping blanks/comments — i.e. converging on A.
