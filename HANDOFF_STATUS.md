# Handoff status — October 11, 2026

Branch: `main`. No commits made. Repository baseline and implementation files
are untracked; git status does not provide an original tracked-file diff.

## Baseline / safety
- Read PROJECT_CONTEXT.md, local instructions, historical source, dependency,
  build and devcontainer configuration, and git status before implementation.
- Replaced historical top-level OAuth/billing/public-free app paths. No runtime
  Streamlit package mutation remains. Historical billing.py, patch script and
  Google verification notes stay as inactive artifacts, not imports/dependencies.
- Converted requirements.txt to pyproject.toml + uv.lock. No unrelated user
  changes discarded. Existing .env and credentials were not read or changed.
- No real emails, recipients, deployment, external database migration, SMS,
  payments or external service-state changes were used for testing.

## Completed
- Python 3.11–3.13 declaration, Python 3.12 development/CI; uv lock, pytest/Ruff
  configuration, names-only .env.example, GitHub Actions, locked devcontainer
  build and no CORS/XSRF disabling flags.
- app.py is composition only; outreach/ui.py composes Streamlit with pure
  recipients/templates/batches/outcomes/invitations and service adapters.
- Structured manual/CSV recipients, preview and mapping, mandatory append vs
  replace, Unicode/compound name preservation, explicit-address precedence,
  email-validator normalization (no MX checks), final edited deduplication.
- Exact rendered reviews and content hash including intended recipients in fake
  mode, From/Reply-To/BCC, attachment metadata/content hash and test mode.
- Validated job/URL/placeholders, BCC, attachment format/name/upload and encoded
  MIME size; spreadsheet-safe CSV exports; per-user suppression checks.
- Magic-link authentication, hashed expiring single-use invitations, verified
  UUID ownership, database admin role, revocation and per-operation authorization.
- Migration `supabase/migrations/202610110001_invite_outreach.sql`: private
  outreach schema, users/plans/settings/invitations/outreach_batches/
  recipient_attempts/quota_reservations/suppression_entries/audit_events,
  RLS/no direct table grants, four authenticated public RPCs. Legacy data untouched.
- Atomic globally serialized full-batch reservations; plan_1 default 300 UTC/day,
  batch/global UTC calendar-hour/day caps, required-config fail-closed behavior.
- Provider-independent fake and TLS-only SMTP adapters. Durable submitting
  checkpoint before SMTP, accepted/failed/unknown finalization, Message-ID and
  raw SMTP DATA response storage (including queue identifiers when supplied),
  primary-vs-BCC refusal details, SMTPUTF8 capability checking.
- Duplicate start protection, claim tokens and expected-ordinal click protection;
  one user-driven submission per continuation, cancellation between recipients,
  exact persisted-content recovery, identical attachment reupload, no unknown retry.
- Standalone local privacy/terms HTML build creates its output directory and
  no longer references absent assets or external font services. Shared legal
  drafts accurately label provider/retention/operator/legal placeholders.
- SETUP.md covers deployment commands, config, migration/admin bootstrap,
  Supabase magic-link email template, security and release blockers.

## Actual checks run
Final results on Windows / Python 3.12.5:
- `uv sync --locked` — passed; 51 resolved packages, 49 checked installations.
- `uv run ruff format --check .` — passed; 31 Python files formatted.
- `uv run ruff check .` — passed, no violations.
- `uv run pytest` — **81 passed in 3.91s**. Synthetic domain tests, threaded
  fake reservation/click tests, mocked HTTP/SMTP, Streamlit AppTest, static build.
- `uv run python -m compileall -q app.py outreach legal.py build_site.py` — passed.
- `uv run python build_site.py` — passed; wrote site/privacy.html and site/terms.html.
- SQL parser test: 37 PostgreSQL statements and four PLpgSQL function bodies
  parse. Private composite declarations are replaced with record only for body
  syntax parsing because pglast has no actual schema catalog.
- `git status --short` and `git branch --show-current` — ran; branch main, files
  untracked. Generated site/.venv/cache files are ignored.

Earlier findings corrected: Ruff import/variable/closure violations; AppTest's
three-second initial data-editor load timeout (raised test timeout to 30s).
Raw pglast PLpgSQL parsing cannot resolve the new private composite types; the
syntax-only test explicitly documents its limited substitution. These are not
claims of database integration success.

## Decisions / operational semantics
- Database `is_admin` role, not an email allowlist; bootstrap one verified Auth
  UUID explicitly with operator SQL. No app service-role credential.
- Plan allowance lives in outreach.plans. DB limits/sender settings must be set
  independently; app config can tighten, not expand limits. Real sending needs
  both app and database flags. Missing required settings block starts/sends.
- No automatic bulk loop/background thread: users explicitly continue each
  recipient. Closing the browser stops future actions, not an in-flight SMTP send.
- Count accepted real message submissions, including at most one optional BCC,
  once per message, not per envelope address. No delivery claim is made.
- Unknown outcomes keep conservative holds across time buckets. Never retry
  automatically. Definite failures and pending cancellations release holds.
- Five-minute stale submitting lease covers SMTP timeout (maximum 120 seconds)
  and HTTP persistence time. Recovery needs explicit action and exact reapproval.
- Attachments remain in session memory; persisted metadata/content hash permits
  identical reupload. Exact rendered messages/recipient addresses remain in DB.
- Identical owner/content hashes cannot start twice, even under new batch UUIDs.
- Supabase Auth signup alone grants no product access. Rate limits/auth abuse
  controls must be configured; expired sessions require a new magic link.

## Not production-verified / remaining decisions
1. No PostgreSQL/Docker executables were found; migrations were NOT applied.
   Fake concurrency and parser tests do NOT prove database locking, RLS,
   role grants, catalog resolution, Supabase JWT semantics or cross-user isolation.
2. Actual SMTP provider, domain SPF/DKIM/DMARC/provider verification, credentials,
   provider limits/acceptable-use policy and SMTPUTF8 behavior remain operator tasks.
3. Configure isolated Supabase, email-auth delivery, magic-link template/redirect,
   rate limits, admin bootstrap and quota settings; never enable real sends in tests.
4. Confirm hosting/build commands, log redaction, secrets and backups. No hosted
   browser/auth integration or deployment was performed. AppTest uses mocks.
5. Finalize operator/contact/jurisdiction/legal notices, retention/deletion of
   content/metadata/audit records/provider copies and evidence-based unknown-hold
   reconciliation. No automated deletion or unknown reconciliation UI exists.
6. Historical artifacts remain for reference only; do not follow their old setup.

## Next concrete task
Run the migration in an isolated Supabase project/local stack with fake SMTP;
verify two-session quota races, invitation races/revocation, cross-user denial,
stale recovery and cancellation release. Resolve provider, retention/legal and
hosting configuration before approving production activation.
