# Spec: Login and Logout

## Overview
This feature implements user authentication for Spendly. It converts the `/login` stub into a functional POST handler that verifies credentials against the database, stores the authenticated user's ID in the session, and redirects to the dashboard (or a suitable landing page). It also implements the `/logout` stub, which clears the session and redirects to the landing page. After this step, the app can distinguish logged-in users from guests, which is a prerequisite for all expense features.

## Depends on
- Step 01 — Database Setup (`users` table must exist)
- Step 02 — Registration (`create_user` and password hashing must be in place; a user must exist to log in against)

## Routes
- `GET /login` — render login form — public
- `POST /login` — validate credentials, set session, redirect — public
- `GET /logout` — clear session, redirect to `/` — public (no login required to log out)

## Database changes
No database changes. The `users` table created in Step 01 already stores `email` and `password_hash`.

## Templates
- **Modify:** `templates/login.html` — add a POST form with `email` and `password` fields, flash message display, and a link to `/register`
- **Modify:** `templates/base.html` — the navbar currently always shows "Sign in" / "Get started", with no way to log out once logged in. Add a `session.user_id` check in `.nav-links`: when a session is active, show a "Log out" link (`url_for('logout')`) instead of "Sign in" / "Get started"; when there is no session, keep the existing two links unchanged. This must NOT link to `/profile` or any other not-yet-built page — `/profile` is out of scope until Step 04, and login no longer redirects there (it redirects to `landing`). The only new nav element in scope is the "Log out" link itself.

## Files to change
- `app.py` — implement `login()` as GET+POST handler and implement `logout()`
- `database/db.py` — add `get_user_by_email(email)` helper that returns a user row or `None`
- `templates/login.html` — add POST form and flash display
- `templates/base.html` — add the `session.user_id`-based "Log out" nav link described above

## Files to create
No new files.

## New dependencies
No new dependencies. `werkzeug.security.check_password_hash` is already available via the existing `werkzeug` install.

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` via `get_db()`
- Parameterised queries only — never use f-strings in SQL
- Passwords verified with `werkzeug.security.check_password_hash`
- Session key for the logged-in user must be `session["user_id"]` (integer)
- Use `flask.session` — do not roll a custom session mechanism
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Use `url_for()` for every internal link — never hardcode paths
- On failed login show a generic flash error ("Invalid email or password.") — do not reveal which field was wrong
- After successful login redirect to `url_for("landing")` until a dashboard route exists
- Do not flash a message on successful login — redirect to the landing page silently, with no banner
- `logout()` must call `session.clear()` then redirect to `url_for("landing")`
- `get_user_by_email` belongs in `database/db.py`, not inline in the route
- The navbar's logged-in state must be driven by `session.user_id` directly in the Jinja template (Flask injects `session` into the template context automatically) — do not pass a separate `logged_in` variable from every route
- The "Log out" nav link must point at `url_for('logout')`, never a hardcoded `/logout` path, and must not introduce a link to `/profile` or any other unbuilt route
- A user with an active session (`session.user_id` set) must not be able to reach `/login` or `/register` — both routes must redirect to `url_for("landing")` immediately, before handling `GET` or `POST`, if a session is already active

## Definition of done
- [ ] Visiting `GET /login` renders the login form with email and password fields
- [ ] Submitting the form with valid credentials (e.g. demo@spendly.com / demo123) sets `session["user_id"]` and redirects to `/`
- [ ] Submitting with a wrong password shows "Invalid email or password." flash and stays on the login page
- [ ] Submitting with an unregistered email shows the same generic error flash
- [ ] Visiting `GET /logout` clears the session and redirects to `/`
- [ ] After logout, `session["user_id"]` is no longer present
- [ ] The `/logout` route no longer returns the raw stub string
- [ ] After a successful login, the navbar shows a "Log out" link and no longer shows "Sign in" / "Get started"
- [ ] The navbar does not link to `/profile` anywhere as part of this step
- [ ] After visiting `/logout`, the navbar reverts to showing "Sign in" / "Get started"
- [ ] While logged in, visiting `GET /login` or `GET /register` redirects to `/` instead of rendering the form