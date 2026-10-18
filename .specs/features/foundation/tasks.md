# Foundation Tasks

## Execution Protocol (MANDATORY -- do not skip)

Implement these tasks with the `tlc-spec-driven` skill: **activate it by name and follow its Execute flow and Critical Rules.** Do not search for skill files by filesystem path. The skill is the source of truth for the full flow (per-task cycle, sub-agent delegation, adequacy review, Verifier, discrimination sensor).

**If the skill cannot be activated, STOP and tell the user - do not proceed without it.**

---

**Spec**: `.specs/features/foundation/spec.md`
**Context**: `.specs/features/foundation/context.md`
**Design**: skipped — Rails 8.1 conventions and the `authentication` generator resolve the architecture.
**Status**: Draft

Conventions (AD-007): code, routes, tables and I18n keys in English; every user-facing string comes from `config/locales/pt-BR.yml`.
Environment: `PGHOST=localhost PGUSER=postgres PGPASSWORD=root` exported in the shell, never committed.

---

## Test Coverage Matrix

> Generated from codebase, project guidelines, and spec - confirm before Execute. Guidelines found: none - strong defaults applied. Framework: Minitest (spec assumption); `config/ci.rb` defines the local gate.

| Code Layer | Required Test Type | Coverage Expectation | Location Pattern | Run Command |
| ---------- | ------------------ | -------------------- | ---------------- | ----------- |
| Model (`User`, `Session`) | unit | Every validation, normalization and token rule 1:1 to spec ACs; every listed edge case | `test/models/*_test.rb` | `bin/rails test test/models` |
| Controller + routes | integration | Every route in scope: happy path + every listed edge case + error paths (invalid input, invalid/expired token, rate limit, auth redirect) | `test/controllers/*_test.rb` | `bin/rails test test/controllers` |
| Mailer | unit | Recipient, subject (pt-BR) and link of each email | `test/mailers/*_test.rb` | `bin/rails test test/mailers` |
| Config / schema / locale files | none | - (build gate only) | - | build gate only |

## Gate Check Commands

> Generated from codebase - confirm before Execute.

| Gate Level | When to Use | Command |
| ---------- | ----------- | ------- |
| Quick | After tasks with unit tests only | `bin/rails test test/models test/mailers` |
| Full | After tasks with integration tests | `bin/rails test` |
| Build | After phase completion or config-only tasks | `bin/ci` |

---

## Execution Plan

Phases run sequentially; tasks within a phase run in order.

### Phase 1: Project setup

```
T1 → T2 → T3 → T4 → T5
```

### Phase 2: Authentication core

```
T6 → T7 → T8
```

### Phase 3: Account flows

```
T9 → T10 → T11 → T12
```

### Phase 4: Closing gate

```
T13
```

---

## Task Breakdown

### T1: Commit the generated Rails scaffold and planning docs

**What**: Commit the untouched `rails new` output plus `.gitignore`, `.tool-versions` and `.specs/` as the project baseline.
**Where**: repository root (existing untracked files)
**Depends on**: None
**Reuses**: output of `rails _8.1.4_ new .`
**Requirement**: none (baseline)

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [x] `git status --porcelain` is empty after the commit
- [x] `config/master.key`, `log/*`, `tmp/*`, `storage/*` are not tracked
- [x] Gate check passes: `bin/rails test` (0 tests, 0 failures)

**Tests**: none
**Gate**: quick

**Commit**: `chore: adicionar esqueleto rails 8.1 e specs do projeto`

---

### T2: Switch the database to PostgreSQL

**What**: Replace SQLite with PostgreSQL in development, test and the four production databases, with no credentials in the file.
**Where**: `config/database.yml`
**Depends on**: T1
**Reuses**: `bin/rails db:system:change --to=postgresql`
**Requirement**: DB-01, DB-02, DB-03, DB-04

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] `Gemfile` has `pg` and no `sqlite3`; `Dockerfile` no longer installs sqlite
- [ ] `config/database.yml` uses `adapter: postgresql` in every environment (DB-01)
- [ ] `bin/rails db:prepare` creates `vtes_development` and `vtes_test` (DB-02)
- [ ] `grep -nE "password|username" config/database.yml` returns only comments (DB-03)
- [ ] production defines `primary`, `cache`, `queue`, `cable` on PostgreSQL (DB-04)
- [ ] Gate check passes: `bin/rails test`

**Tests**: none
**Gate**: quick

**Commit**: `build: trocar sqlite por postgresql`

---

### T3: Remove the generated GitHub CI files

**What**: Delete `.github/workflows/ci.yml` and `.github/dependabot.yml`.
**Where**: `.github/`
**Depends on**: T2
**Reuses**: -
**Requirement**: QA-03

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] `.github/` no longer exists (QA-03)
- [ ] Gate check passes: `bin/rails test`

**Tests**: none
**Gate**: quick

**Commit**: `chore: remover ci do github`

---

### T4: Set pt-BR as the default locale

**What**: Add `rails-i18n`, set `default_locale = :"pt-BR"` and create the base `config/locales/pt-BR.yml`.
**Where**: `config/application.rb`
**Depends on**: T3
**Reuses**: `rails-i18n` translations for Active Record messages
**Requirement**: QA-04

**Tools**:

- MCP: `context7` (rails-i18n setup)
- Skill: NONE

**Done when**:

- [ ] `bin/rails runner 'puts I18n.default_locale'` prints `pt-BR` (QA-04)
- [ ] `config/locales/pt-BR.yml` exists with English keys
- [ ] Gate check passes: `bin/rails test`

**Tests**: none
**Gate**: quick

**Commit**: `feat(i18n): definir pt-BR como idioma padrão`

---

### T5: Open development emails in the browser

**What**: Add `letter_opener` to the development group and set it as the development delivery method.
**Where**: `config/environments/development.rb`
**Depends on**: T4
**Reuses**: existing `default_url_options` (localhost:3000)
**Requirement**: supports AUTH-01, AUTH-16 (spec assumption)

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] `config.action_mailer.delivery_method = :letter_opener` in development only
- [ ] Gate check passes: `bin/ci` (closes Phase 1)

**Tests**: none
**Gate**: build

**Commit**: `build: usar letter_opener para e-mails em desenvolvimento`

---

### T6: Generate the Rails authentication base

**What**: Run `bin/rails generate authentication` and migrate, keeping the generated sessions, passwords, mailer and tests.
**Where**: `app/controllers/concerns/authentication.rb`
**Depends on**: None (requires Phase 1 complete)
**Reuses**: Rails 8.1 authentication generator
**Requirement**: base for AUTH-09, AUTH-13, AUTH-16

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] `users` and `sessions` tables exist in `db/schema.rb`
- [ ] Generated tests pass unchanged
- [ ] Gate check passes: `bin/rails test`
- [ ] Test count: all generated tests pass (no deletions)

**Tests**: integration
**Gate**: full

**Commit**: `feat(auth): gerar base de autenticação do rails`

---

### T7: Add username and email confirmation to User

**What**: Add `username` and `confirmed_at` columns, unique indexes on `lower(email_address)` and `username`, validations, normalization and the 24-hour single-use confirmation token.
**Where**: `app/models/user.rb`
**Depends on**: T6
**Reuses**: `normalizes`, `has_secure_password`, `generates_token_for`
**Requirement**: AUTH-02, AUTH-03, AUTH-04, AUTH-05

**Tools**:

- MCP: `context7` (`generates_token_for`)
- Skill: NONE

**Done when**:

- [ ] Email: required, valid format, stripped and downcased, unique case-insensitively ("E-mail já está em uso") (AUTH-03)
- [ ] Username: required, 5–30 chars, `[a-z0-9_-]`, downcased, unique ("Username já está em uso") (AUTH-04)
- [ ] Password: 8 chars minimum, over 72 bytes rejected (AUTH-02, edge case)
- [ ] Duplicate insert bypassing validations raises `ActiveRecord::RecordNotUnique` (AUTH-05)
- [ ] Confirmation token resolves within 24 h, fails after 24 h and after `confirmed_at` is set
- [ ] Gate check passes: `bin/rails test test/models`
- [ ] Test count: ≥ 14 model tests pass

**Tests**: unit
**Gate**: quick

**Commit**: `feat(auth): adicionar username e confirmação de e-mail ao usuário`

---

### T8: Public home page and protected account page

**What**: Add `HomeController#show` as public root and `AccountsController#show` at `/account`, showing email and username.
**Where**: `app/controllers/accounts_controller.rb`
**Depends on**: T7
**Reuses**: `allow_unauthenticated_access` from the `Authentication` concern
**Requirement**: AUTH-14, AUTH-15

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] `GET /` responds 200 without login
- [ ] `GET /account` without login redirects to `new_session_path` (AUTH-14)
- [ ] `GET /account` logged in shows the user's email and username (AUTH-15)
- [ ] Gate check passes: `bin/ci` (closes Phase 2)
- [ ] Test count: ≥ 3 new integration tests pass

**Tests**: integration
**Gate**: build

**Commit**: `feat(account): adicionar página inicial e página da conta`

---

### T9: Sign-up

**What**: Add `RegistrationsController` (`GET/POST /registration`) that creates an unconfirmed user and sends the confirmation email.
**Where**: `app/controllers/registrations_controller.rb`
**Depends on**: None (requires Phase 2 complete)
**Reuses**: `User` validations from T7; `rate_limit` as in `SessionsController`
**Requirement**: AUTH-01, AUTH-02, AUTH-03, AUTH-04, AUTH-12, AUTH-19

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] Valid sign-up creates user with `confirmed_at: nil`, enqueues 1 confirmation email, starts no session (AUTH-01)
- [ ] Invalid sign-up returns 422 with pt-BR errors, creates no user, sends no email (AUTH-02)
- [ ] Duplicate email (case/whitespace variants) and duplicate username are rejected with their messages (AUTH-03, AUTH-04)
- [ ] 11th attempt from the same IP within 3 minutes shows "Muitas tentativas. Tente novamente mais tarde." (AUTH-12)
- [ ] Logged-in user visiting `/registration/new` is redirected to `/` (AUTH-19)
- [ ] `UserMailer#confirmation` test checks recipient, pt-BR subject and confirmation link
- [ ] Gate check passes: `bin/rails test`
- [ ] Test count: ≥ 7 new tests pass

**Tests**: integration
**Gate**: full

**Commit**: `feat(auth): adicionar cadastro de usuário`

---

### T10: Email confirmation and resend

**What**: Add `ConfirmationsController` with `GET /confirmations/:token` (confirm) and `GET/POST /confirmations` (resend).
**Where**: `app/controllers/confirmations_controller.rb`
**Depends on**: T9
**Reuses**: confirmation token from T7; mailer from T9
**Requirement**: AUTH-06, AUTH-07, AUTH-08, AUTH-12

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] Valid token sets `confirmed_at`, starts session, redirects to `/` with "E-mail confirmado" (AUTH-06)
- [ ] Expired (25 h, via `travel`), tampered and already-used tokens leave the user unchanged and show "Link inválido ou expirado" with a resend link (AUTH-07)
- [ ] Resend shows the same neutral message for unknown, confirmed and unconfirmed emails; only unconfirmed sends an email (AUTH-08)
- [ ] Resend rate limit returns the AUTH-12 message on the 11th attempt
- [ ] Gate check passes: `bin/rails test`
- [ ] Test count: ≥ 8 new tests pass

**Tests**: integration
**Gate**: full

**Commit**: `feat(auth): adicionar confirmação e reenvio de e-mail`

---

### T11: Sign-in and sign-out rules

**What**: Adapt `SessionsController` to block unconfirmed users, use pt-BR messages, redirect after sign-out to the sign-in page and redirect logged-in users away from sign-in.
**Where**: `app/controllers/sessions_controller.rb`
**Depends on**: T10
**Reuses**: generated `SessionsController` and its tests
**Requirement**: AUTH-09, AUTH-10, AUTH-11, AUTH-12, AUTH-13, AUTH-19

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] Confirmed user with correct credentials gets a session and lands on `/` (AUTH-09)
- [ ] Unknown email and wrong password both show "E-mail ou senha inválidos", no session (AUTH-10)
- [ ] Unconfirmed user with correct credentials gets no session and sees "Confirme seu e-mail para entrar" with resend link (AUTH-11)
- [ ] 11th attempt in 3 minutes shows the AUTH-12 message
- [ ] Sign-out destroys the current session and redirects to `new_session_path` (AUTH-13)
- [ ] Logged-in user visiting `/session/new` is redirected to `/` (AUTH-19)
- [ ] Gate check passes: `bin/rails test`
- [ ] Test count: ≥ 7 sessions tests pass

**Tests**: integration
**Gate**: full

**Commit**: `feat(auth): ajustar login e logout às regras da conta`

---

### T12: Password reset rules

**What**: Adapt `PasswordsController` to show the neutral message, end every session of the user after reset and show pt-BR invalid-link messages.
**Where**: `app/controllers/passwords_controller.rb`
**Depends on**: T11
**Reuses**: generated `PasswordsController`, `PasswordsMailer` and their tests
**Requirement**: AUTH-12, AUTH-16, AUTH-17, AUTH-18

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] Reset request shows the same message for known and unknown emails; email sent only for known (AUTH-16)
- [ ] Valid token + valid password updates the password, deletes all of the user's sessions, redirects to sign-in (AUTH-17)
- [ ] Expired (16 min) and tampered tokens leave the password unchanged and show "Link inválido ou expirado" (AUTH-18)
- [ ] 11th reset request in 3 minutes shows the AUTH-12 message
- [ ] `PasswordsMailer` test checks recipient, pt-BR subject and reset link
- [ ] Gate check passes: `bin/rails test`
- [ ] Test count: ≥ 6 passwords tests pass

**Tests**: integration
**Gate**: full

**Commit**: `feat(auth): ajustar redefinição de senha`

---

### T13: Green local gate and setup docs

**What**: Make `bin/ci` pass end to end and document the PostgreSQL environment variables and `bin/ci` in the README.
**Where**: `README.md`
**Depends on**: None (requires Phase 3 complete)
**Reuses**: `config/ci.rb`
**Requirement**: QA-01, QA-02

**Tools**:

- MCP: NONE
- Skill: NONE

**Done when**:

- [ ] `bin/ci` exits 0 (QA-01)
- [ ] A temporary rubocop offense makes `bin/ci` exit non-zero, then is reverted (QA-02)
- [ ] README lists Ruby/Rails versions, `PGHOST`/`PGUSER`/`PGPASSWORD` and `bin/setup`, `bin/dev`, `bin/ci`
- [ ] Gate check passes: `bin/ci`

**Tests**: none
**Gate**: build

**Commit**: `docs: documentar setup local e gate de qualidade`

---

## Phase Execution Map

```
Phase 1 → Phase 2 → Phase 3 → Phase 4

Phase 1:  T1 → T2 → T3 → T4 → T5
Phase 2:  T6 → T7 → T8
Phase 3:  T9 → T10 → T11 → T12
Phase 4:  T13
```

---

## Task Granularity Check

| Task | Scope | Status |
| ---- | ----- | ------ |
| T1 | 1 baseline commit | ✅ Granular |
| T2 | database config (generator touches Gemfile/Dockerfile) | ✅ Cohesive |
| T3 | 1 directory removal | ✅ Granular |
| T4 | locale config | ✅ Granular |
| T5 | 1 env config | ✅ Granular |
| T6 | generator output | ⚠️ Many files, one generator command |
| T7 | 1 model + its migration | ✅ Cohesive |
| T8 | 2 tiny pages (root + account) | ⚠️ Cohesive: one checks the other's redirect |
| T9 | 1 controller + its mailer | ✅ Cohesive |
| T10 | 1 controller | ✅ Granular |
| T11 | 1 controller | ✅ Granular |
| T12 | 1 controller | ✅ Granular |
| T13 | gate + README | ✅ Granular |

## Diagram-Definition Cross-Check

| Task | Depends On (task body) | Diagram Shows | Status |
| ---- | ---------------------- | ------------- | ------ |
| T1 | None | start | ✅ Match |
| T2 | T1 | T1 → T2 | ✅ Match |
| T3 | T2 | T2 → T3 | ✅ Match |
| T4 | T3 | T3 → T4 | ✅ Match |
| T5 | T4 | T4 → T5 | ✅ Match |
| T6 | None (Phase 1 complete) | phase start | ✅ Match |
| T7 | T6 | T6 → T7 | ✅ Match |
| T8 | T7 | T7 → T8 | ✅ Match |
| T9 | None (Phase 2 complete) | phase start | ✅ Match |
| T10 | T9 | T9 → T10 | ✅ Match |
| T11 | T10 | T10 → T11 | ✅ Match |
| T12 | T11 | T11 → T12 | ✅ Match |
| T13 | None (Phase 3 complete) | phase start | ✅ Match |

## Test Co-location Validation

| Task | Code Layer Created/Modified | Matrix Requires | Task Says | Status |
| ---- | --------------------------- | --------------- | --------- | ------ |
| T1 | none | none | none | ✅ OK |
| T2 | config | none | none | ✅ OK |
| T3 | config | none | none | ✅ OK |
| T4 | config / locale | none | none | ✅ OK |
| T5 | config | none | none | ✅ OK |
| T6 | model + controller (generated) | integration | integration | ✅ OK |
| T7 | model | unit | unit | ✅ OK |
| T8 | controller | integration | integration | ✅ OK |
| T9 | controller + mailer | integration | integration | ✅ OK |
| T10 | controller | integration | integration | ✅ OK |
| T11 | controller | integration | integration | ✅ OK |
| T12 | controller + mailer | integration | integration | ✅ OK |
| T13 | docs | none | none | ✅ OK |
