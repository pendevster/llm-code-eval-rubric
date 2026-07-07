# 03 — Express `POST /users` handler

**Category:** API / security
**Prompt given to both models:**

> Write an Express route handler for `POST /users` that creates a user from a JSON body
> containing `email` and `password`, using a PostgreSQL database via a `db` client with a
> `db.query(text, params)` method. Return the created user.

---

## Model A

```javascript
const bcrypt = require("bcrypt");

async function createUser(req, res) {
  const { email, password } = req.body ?? {};

  if (typeof email !== "string" || typeof password !== "string") {
    return res.status(400).json({ error: "email and password are required" });
  }
  if (!/^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(email) || password.length < 8) {
    return res.status(400).json({ error: "invalid email or password too short" });
  }

  try {
    const hash = await bcrypt.hash(password, 12);
    const { rows } = await db.query(
      "INSERT INTO users (email, password_hash) VALUES ($1, $2) RETURNING id, email, created_at",
      [email.toLowerCase(), hash]
    );
    return res.status(201).json(rows[0]);
  } catch (err) {
    if (err.code === "23505") {
      return res.status(409).json({ error: "email already registered" });
    }
    return res.status(500).json({ error: "internal error" });
  }
}
```

## Model B

```javascript
async function createUser(req, res) {
  const { email, password } = req.body;

  const result = await db.query(
    `INSERT INTO users (email, password) VALUES ('${email}', '${password}') RETURNING *`
  );

  res.json(result.rows[0]);
}
```

---

## Scores

| Axis | Model A | Model B | Note |
|------|:------:|:------:|------|
| Correctness | 5 | 2 | B "works" only on trusted, well-formed input |
| Edge-case handling | 4 | 1 | B crashes on missing body; no duplicate handling |
| Idiomatic style & readability | 4 | 4 | B is short but that brevity hides the danger |
| Efficiency | 5 | 5 | Not a differentiator here |
| Security & robustness | 5 | 1 | B: SQL injection, plaintext password, leaks hash |
| Instruction-following | 5 | 3 | B returns the user but exposes the whole row |
| **Total (/30)** | **28** | **12** | |

## Comparison

This prompt implies **untrusted input** (a public-ish write endpoint), so the security axis
carries the verdict. Model B has three independent, severe defects:

- **SQL injection.** Values are interpolated straight into the query string. A `password`
  of `x'); DROP TABLE users; --` is executed. This alone disqualifies it.
- **Plaintext password storage.** `password` is inserted as-is; there's no hashing. A
  database read now exposes every credential.
- **Data leak on the way out.** `RETURNING *` plus `res.json(result.rows[0])` returns the
  stored password field to the client.

It also throws on a missing body (`req.body` undefined → destructuring error) and has no
handling for a duplicate email.

Model A parameterises the query (`$1, $2`), hashes with bcrypt at cost 12, validates types and
a basic email shape, normalises the email, returns only non-sensitive columns, and maps the
Postgres unique-violation code `23505` to a clean 409. Minor nits keep it from a perfect sheet:
the email regex is permissive (acceptable for a first-pass guard), and there's no rate limiting
— out of scope for the prompt.

## Verdict

**Winner: A** — decisively.

Model B is a textbook example of code that runs fine in a demo and is catastrophic in
production: it's injectable, stores secrets in plaintext, and leaks them back to the caller.
Under the rubric's security-override rule, a handler touching user data that is injectable
cannot win regardless of brevity. Nothing about B's concision offsets a `DROP TABLE` vector.
What would flip the call: nothing short of B parameterising the query, hashing the password,
and restricting the returned columns — i.e. becoming A.
