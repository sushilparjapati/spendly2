
# Spec: Login and Logout

## Overview

Implement session-based login and logout so registered users can authenticate with Spendly. This step upgrades the existing stub `GET /login` route into a fully functional form handler that verifies credentials against the `users` table, writes the authenticated user's id and name into Flask's signed cookie session, and redirects to the dashboard (or landing page until the dashboard exists). It also implements the `GET /logout` stub to clear the session and redirect back to the landing page. Together these two routes form the authentication boundary that all future protected pages depend on.

## Depends on

- Step 01 — Database setup (`users` table, `get_db()`)
- Step 02 — Registration (`create_user()`, password hashing convention)

## Routes

- `GET /login` — render login form — public (already exists as stub, upgrade to GET+POST)
- `POST /login` — validate credentials, write session, redirect — public
- `GET /logout` — clear session, redirect to `/` — public (safe to call when not logged in)

## Database changes

No new tables or columns. The existing `users` table covers all requirements.

A new DB helper must be added to `database/db.py`:

- `get_user_by_email(email)` — queries `users` by email, returns a `sqlite3.Row` (id, name, email, password_hash, created_at) or `None` if not found.

## Templates

- **Modify**: `templates/login.html`
  - Change the form `action` to `url_for('login')` with `method="post"`
  - Add `name` attributes to all inputs: `email`, `password`
  - Add a block to display flash error messages (e.g. "Invalid email or password")
  - Keep all existing visual design

- **Modify**: `templates/base.html`
  - In the nav, show a "Logout" link (`url_for('logout')`) when `session.get('user_id')` is set
  - Hide the "Login" and "Register" nav links when the user is already logged in

## Files to change

- `app.py` — upgrade `login()` to handle `GET` and `POST`; implement `logout()`; import `session` and `check_password_hash`
- `database/db.py` — add `get_user_by_email(email)` helper
- `templates/login.html` — wire up form action/method and flash message display
- `templates/base.html` — conditionally render nav links based on session state

## Files to create

None.

## New dependencies

No new dependencies. Uses `werkzeug.security.check_password_hash` (already installed) and Flask's built-in `session`, `flash`, `redirect`, `url_for`.

## Rules for implementation

- No SQLAlchemy or ORMs
- Parameterised queries only — never use f-strings in SQL
- Verify passwords with `werkzeug.security.check_password_hash` — never compare plaintext
- On successful login, store only `session['user_id']` (int) and `session['user_name']` (str) — never store the password hash in the session
- On failed login, flash a single generic message "Invalid email or password" — do not distinguish between unknown email and wrong password (prevents user enumeration)
- On success, redirect to `url_for('landing')` until a dashboard route exists
- `logout()` must call `session.clear()` then redirect to `url_for('landing')` — it must not error if the user is already logged out
- Use `abort(405)` if an unsupported HTTP method reaches the route
- All templates extend `base.html`
- Use CSS variables — never hardcode hex values
- Use `url_for()` for every internal link — never hardcode URLs

## Definition of done

- [ ] `GET /login` renders the login form without errors
- [ ] Submitting valid credentials writes `user_id` and `user_name` to the session and redirects to `/`
- [ ] Submitting an unknown email re-renders the form with "Invalid email or password" — no session written
- [ ] Submitting a wrong password re-renders the form with "Invalid email or password" — no session written
- [ ] Submitting with any empty field re-renders the form with a validation error
- [ ] `GET /logout` clears the session and redirects to `/` — works whether or not the user is logged in
- [ ] After login the nav shows a "Logout" link and hides "Login" / "Register"
- [ ] After logout the nav shows "Login" and "Register" again
- [ ] Visiting `/logout` when not logged in does not raise an error