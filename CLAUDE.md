# CLAUDE.md

## Project overview

Spendly is a lightweight personal expense tracker built with Flask and SQLite.

---

## Architecture

```
expense-tracker 2/
├── app.py              # All routes — single file, no blueprints
├── main.py             # `uv init` leftover — NOT the entry point
├── spendly.db          # SQLite database — gitignored, created by init_db()
├── database/
│   ├── __init__.py     # Empty — marks `database/` as a Python package
│   └── db.py           # SQLite helpers: get_db(), init_db(), seed_db(), create_user(), get_user_by_email()
├── templates/
│   ├── base.html       # Shared layout — all pages extend this
│   ├── landing.html    # Marketing page — hero, features, CTA, "See how it works" video modal
│   ├── login.html      # Sign-in form (POST /login — implemented in Step 3)
│   ├── register.html   # Account creation form (POST /register — implemented in Step 2)
│   ├── terms.html      # Static Terms and Conditions page
│   └── privacy.html    # Static Privacy Policy page
├── static/
│   ├── css/
│   │   └── style.css   # Single global stylesheet (page-specific CSS inline in {% block head %})
│   └── js/
│       └── main.js     # Vanilla JS only — no frameworks
├── .claude/
│   ├── commands/
│   │   ├── create-spec.md   # /create-spec — scaffolds a spec + feature branch for the next step
│   │   ├── seed-expense.md  # /seed-expense — seeds realistic dummy expenses for a user
│   │   └── seed-user.md     # /seed-user — creates a single dummy user in the database
│   ├── specs/
│   │   ├── 01-database-setup.md  # Spec for Step 1 (database setup)
│   │   ├── 02-registration.md    # Spec for Step 2 (registration)
│   │   └── 03-login-and-logout.md  # Spec for Step 3 (login and logout)
│   └── plans/
│       ├── 01-database-setup.md  # Implementation plan for Step 1
│       ├── 02-registration.md    # Implementation plan for Step 2
│       └── 03-login-and-logout.md  # Implementation plan for Step 3
├── file.txt            # Assignment prompt log — spec/history reference, not code
├── pyproject.toml      # uv-managed dependencies
├── requirements.txt    # Pinned pip alternative for non-uv installs
└── uv.lock             # uv-resolved dependency lockfile — do not edit by hand
```

**Where things belong:**

- New routes → `app.py` only, no blueprints
- DB logic → `database/db.py` only, never inline in routes
- New pages → new `.html` file extending `base.html`
- Page-specific styles → new `.css` file, not inline `<style>` tags

---

## Code style

- Python: PEP 8, snake_case for all variables and functions
- Templates: Jinja2 with `url_for()` for every internal link — never hardcode URLs
- Route functions: one responsibility only — fetch data, render template, done
- DB queries: always use parameterized queries (`?` placeholders) — never f-strings in SQL
- Error handling: use `abort()` for HTTP errors, not bare `return "error string"`

---

## Tech constraints

- **Flask only** — no FastAPI, no Django, no other web frameworks
- **SQLite only** — no PostgreSQL, no SQLAlchemy ORM, no external DB
- **Vanilla JS only** — no React, no jQuery, no npm packages
- **No new uv packages** — work within `requirements.txt` as-is unless explicitly told otherwise
- Python 3.12+ assumed — f-strings and `match` statements are fine

---

## Commands

Dependencies are pinned in both `pyproject.toml` and `requirements.txt`. `uv` is the canonical tool (an `uv.lock` is checked in)

```bash
# install (uv)
uv sync

# install (uv)
uv add  -r requirements.txt

# run the dev server — Flask debug, port 5001 (NOT the default 5000)
uv run app.py

# tests (pytest + pytest-flask are already pinned; no tests exist yet)
pytest
pytest path/to/test_file.py::test_name   # single test
```

`main.py` is an unrelated `uv init` leftover ("Hello from expense-tracker-2!") — the real entry point is `app.py`.

On startup, `app.py` auto-calls `init_db()` and `seed_db()` inside an `app.app_context()` block — tables are created and sample data is seeded before the first request is handled.

---

## Implemented vs stub routes

| Route                         | Status                                   |
| ----------------------------- | ---------------------------------------- |
| `GET /`                     | Implemented — renders `landing.html`  |
| `GET /register`             | Implemented — renders `register.html` |
| `POST /register`            | Implemented — creates account, redirects to login |
| `GET /login`                | Implemented — renders `login.html`    |
| `POST /login`               | Implemented — authenticates user, sets session    |
| `GET /terms`                | Implemented — renders `terms.html`    |
| `GET /privacy`              | Implemented — renders `privacy.html`  |
| `GET /logout`               | Implemented — clears session, redirects to landing |
| `GET /profile`              | Stub — Step 4                           |
| `GET /expenses/add`         | Stub — Step 7                           |
| `GET /expenses/<id>/edit`   | Stub — Step 8                           |
| `GET /expenses/<id>/delete` | Stub — Step 9                           |

**Do not implement a stub route unless the active task explicitly targets that step.**

---

## Warnings and things to avoid

- **Never use raw string returns for stub routes** once a step is implemented — always render a template
- **Never hardcode URLs** in templates — always use `url_for()`
- **Never put DB logic in route functions** — it belongs in `database/db.py`
- **Never install new packages** mid-feature without flagging it — keep `requirements.txt` in sync
- **Never use JS frameworks** — the frontend is intentionally vanilla
- **database/db.py is implemented** — get_db(), init_db(), seed_db(), create_user(), get_user_by_email() are ready to use; import from `database.db`
- **FK enforcement is manual** — SQLite foreign keys are off by default; `get_db()` must run `PRAGMA foreign_keys = ON` on every connection
- The app runs on **port 5001**, not the Flask default 5000 — don't change this
