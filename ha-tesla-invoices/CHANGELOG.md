<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 2026.07.08

- **Easier Tesla login**: the *Connect Tesla account* dialog now walks you
  through the sign-in with clear numbered steps, including a hint for desktop
  browsers where the `tesla://auth/callback` address never reaches the
  address bar. New fallback: **paste a refresh token directly** (e.g. from
  tesla_auth or the iOS "Auth app for Tesla") — the token is verified with
  Tesla before it is stored.
- **New option `enable_charging_invoice`** (default: `true`): disable it if
  you only want subscription invoices downloaded. Enable none, one, or both
  invoice types — with both disabled, downloads are paused and syncs only
  verify the Tesla connection (the dashboard shows a banner).
- **Correct month assignment across timezone boundaries**: charging sessions
  are now bucketed by the local date instead of UTC, so invoices around
  midnight at a month boundary are no longer missed or wrongly grouped.
- **Email export robustness**: one unreadable PDF no longer aborts the export
  loop, and overlapping send operations can no longer email an invoice twice.
- **Invoices readable on shared volumes**: downloaded PDFs and metadata are
  written world-readable (`0644`), so SMB shares of the invoice directory
  work; token files stay private.
- **Dashboard translations are now contributable**: languages live in plain
  JSON files — see the application repository's README for how to add one.

## 2026.07.07

Two fixes for the in-app Tesla login introduced in 2026.07.06:

- **"Open Tesla login" works again**: Tesla deregistered the redirect address
  the login relied on, so it failed with *"The 'redirect_uri' supplied is not
  registered for this 'client_id'"*. The login now uses the Tesla mobile
  app's `tesla://auth/callback` deep link instead. After signing in, the
  browser shows an error or an empty page (it cannot open the Tesla app) —
  copy the `tesla://auth/callback?code=…` address from the address bar and
  paste it into the app, as the updated dialog explains.
- **Setup banner no longer sticks around**: the "Welcome! Connect your Tesla
  account" banner was shown even when a token was already configured via the
  `refresh_token` option. It now disappears as soon as a Tesla account is
  connected.

## 2026.07.06

- **Sign in from the dashboard — no configuration needed**: just start the
  app, open the web UI, and click **Connect Tesla account**. You sign in on
  Tesla's own website (this app never sees your password), and the first sync
  starts automatically. The `refresh_token` option is now optional — set it
  only if you prefer to bring your own token; a token obtained via the in-app
  login takes precedence once present.
- A **Tesla account** section on the Maintenance tab shows the connection
  status and lets you reconnect (e.g. to switch accounts).
- **German translation**: the dashboard is now available in English and
  German, including all dialogs, statuses and number/month formatting.
- **Follows your Home Assistant language automatically** (read from the Core
  API at startup — the app now requests the `homeassistant_api` permission
  for this). The new **EN / DE** switch in the top-right corner, next to the
  theme toggle, overrides it (remembered per browser).

## 2026.07.05

Security- and robustness-focused release, following an external code review.

- **Multi-vehicle fix**: the charging history is now requested strictly per
  vehicle. Previously, accounts with several vehicles could download every
  invoice once per vehicle, filed under the wrong VIN.
- **Security hardening**: all state-changing API endpoints are protected
  against cross-site request forgery; email recipients no longer appear in
  URLs; SMTP certificate verification is explicit; the container drops root
  before starting the app; downloaded files are verified to really be PDFs.
- **In-page dialogs** replace browser popups, which could silently fail in
  the Home Assistant companion apps and make buttons (Sync, Email, Delete)
  appear dead.
- **No more duplicate emails after crashes**: metadata is written atomically
  and the PDF re-scan no longer runs concurrently with a sync.
- **ZIP export streams from disk** — no more out-of-memory risk on small
  boxes with a large invoice history.
- Deleting a PDF in the Files tab now removes its metadata entry too; the
  skipped-invoice counter ignores orphaned metadata; date-only invoices no
  longer show a made-up "00:00" time; background polling pauses in hidden
  tabs.
- Amount parsing: English-format totals without decimals (`1,234`) are no
  longer misread as `1.234`.
- The app now ships an icon, and the project is licensed under the
  **MIT License**.

## 2026.07.04

- **Aligned chart timelines**: the *Energy per month* and *Cost per month*
  charts now share one x-axis range, so the two timelines line up instead of
  each chart starting at its own first data point. Months without data show
  a thin faded **0 bar** instead of an invisible gap.
- **Bulk ZIP download**: a new **Download all (ZIP)** button next to
  *Export CSV* downloads every stored invoice PDF as one ZIP archive.
- **Sync all history asks about emails**: instead of a separate checkbox on
  the Maintenance tab, starting a full history sync now asks directly
  whether each new invoice should also be emailed (default: no — invoices
  are marked as *skipped* and can be sent later via *Email backlog*). The
  question only appears when the automatic email export is enabled.
- **Quieter add-on log**: HTTP requests are no longer logged one line each —
  the Supervisor watchdog polls `/health` constantly and drowned the log.
  Syncs, downloads and emails are still logged.
- **Dependency updates**: `pypdf` 5.1.0 → 6.14.2 and `uvicorn` 0.50.0 → 0.51.0.

## 2026.07.03

- **Dark mode**: the dashboard now follows the Home Assistant / system
  appearance automatically, with an **Auto / Light / Dark** switch in the
  top-right corner to override it (the choice is remembered).
- **Unified timestamps**: all timestamps in the dashboard are now shown as
  `YYYY-MM-DD HH:MM` (24-hour clock) including the time zone.
- **Email export no longer floods your inbox**:
  - Invoices synced while email export was disabled are marked as *skipped*
    and are never auto-sent later — enabling the export only emails invoices
    that are new from that point on.
  - **Sync all history** now skips email sending by default; a new checkbox
    on the Maintenance tab lets you opt in explicitly.
  - New **Maintenance → Email backlog** section: sends all skipped invoices
    as a **combined export**, batched into a few emails instead of one mail
    per invoice.
- **Nicer emails**: exported invoices now have a proper text body with an
  invoice summary (date, type, vehicle, location, energy, amount) and a
  meaningful subject line instead of a bare attachment.

## 2026.07.02

- **Uses your Home Assistant time zone**: log timestamps and the
  current/previous-month sync window now follow the time zone configured in
  Home Assistant (read from the Supervisor at startup) instead of UTC.
- **`access_token` option removed**: it was never needed — the refresh token
  is the only credential required; access tokens are obtained, renewed and
  stored automatically. ⚠️ If the app reports an invalid `access_token`
  option after updating, remove that line via
  *Configuration → three-dot menu → Edit in YAML*. (The
  [standalone Docker deployment](https://github.com/steiner-dominik/tesla-invoices)
  still accepts an access token as an alternative for users who prefer not
  to store a long-lived credential.)
- **Clearer logs, no sensitive data**: log messages have been rewritten in
  plain language ("Starting invoice sync…", "Invoice sync finished", …), and
  email addresses, full VINs and raw Tesla API responses are no longer
  logged — logs can be shared in bug reports safely. Tokens were never
  logged.
- **Working token generator links**: the documentation now points to
  [tesla_auth](https://github.com/adriankumpf/tesla_auth) and
  [Auth app for Tesla](https://apps.apple.com/us/app/auth-app-for-tesla/id1552058613)
  (iOS); the previously listed tools are outdated or gone.
- Clarified in the documentation and dashboard footer that this project is
  **not affiliated with Tesla, Inc.**

## 2026.07.01

First public release. Tesla Invoices started as an interactive CLI script,
grew into a Home Assistant add-on
([aSauerwein/tesla-invoices](https://github.com/aSauerwein/tesla-invoices)),
and has been completely refactored into a single application that runs
standalone via Docker or as a [Home Assistant app](https://github.com/steiner-dominik/home-assistant-apps).
Everything below ships in this first release.

### Invoice downloads

- Automatically downloads **all Supercharging and subscription invoices**
  (e.g. Premium Connectivity; subscription downloads can be disabled) on a
  configurable polling interval (1–1440 minutes, default 15).
- Every cycle covers the current **and** previous month, so invoices appearing
  right after a month boundary are never missed; the complete history can be
  fetched via **"Sync all history"** or `POST /api/sync?month=all|cur|prev|YYYY-MM`.
- Follows Tesla's paginated charging-history GraphQL API (which replaced the
  old REST endpoint) with the app-like headers it requires, aggregating
  energy, tier usage and cost from the per-fee-type records.
- One broken invoice never aborts a sync: download errors are logged per
  invoice and retried next cycle; non-PDF responses are rejected; two
  same-day invoices for one vehicle get unique file names.
- Credit notes are stored with negative amounts, so refunds correctly reduce
  all totals; subscription costs are extracted from the localized grand-total
  line of the PDF.

### Resilient Tesla authentication & API access

- **A refresh token alone is enough** — access tokens are bootstrapped,
  rotated and persisted automatically; rotated refresh tokens are saved, and
  freshly pasted tokens win over stored ones only when newer.
- Works around Tesla's bot mitigation: all requests are pinned to **TLS 1.3**,
  and the token refresh is sent with a **browser TLS fingerprint**
  (`curl_cffi`) — Tesla silently issues down-scoped tokens to refreshes made
  from vanilla Python TLS stacks (same root cause as
  [teslamate#5399](https://github.com/teslamate-org/teslamate/issues/5399)).
- A 401/403 from any endpoint forces one token refresh and a retry instead of
  trusting a poisoned access token until expiry; connection resets and
  HTTP 429 are retried with backoff (honoring `Retry-After`); the polling
  schedule adds random jitter so its cadence is not metronomic.

### Analytics dashboard

- Web dashboard (Home Assistant ingress or standalone): summary cards,
  monthly **energy and cost charts** with running totals, per-session
  **price per kWh**, filtering by year/vehicle/type, free-text search and
  sortable columns — following the active filters.
- **Multi-currency aware**: totals are grouped per currency, never blindly
  converted; the preferred display currency is configurable and auto-detected
  by default.
- **Built-in PDF viewer** (no bounce out of the Home Assistant mobile app),
  **CSV export** for expense reports (`GET /api/export.csv`, escaped against
  spreadsheet formula injection), and a **Files tab** with view, download and
  delete actions.
- **Maintenance tab** with detailed sync status, a failure banner when the
  last sync failed, "Sync all history" and "Re-scan PDFs" (re-extracts
  cost/currency from stored PDFs with the current parser).

### Email export

- Optionally sends every new invoice as an email attachment, **exactly once**
  (tracked in the metadata sidecar files).
- Individual invoices can be mailed on demand to any recipient from the
  dashboard.
- SMTP with STARTTLS (587) or implicit TLS (465); Gmail app passwords
  documented.

### Security & robustness

- Token files are stored with mode 600; the download endpoint is hardened
  against path traversal.
- HTTP timeouts everywhere, plus a watchdog health check that reports
  unhealthy if the download loop ever dies (the Supervisor restarts the app).
- PDF parsing and file scanning run off the event loop, so long re-scans
  never block the health check or the dashboard.
- Metadata sidecar files carry a `meta_version`; values produced by older
  extraction logic are re-derived automatically on the next sync or re-scan.

### Packaging & deployment

- One `python:3.14-alpine` image serves both deployments — standalone Docker
  (environment variables / `docker.env`) and the Home Assistant app
  (`/data/options.json`, auto-detected). No s6/bashio, direct Python
  execution.
- Published as a prebuilt multi-arch image to
  `ghcr.io/steiner-dominik/tesla-invoices` (amd64, aarch64) — the Home
  Assistant app pulls it instead of building locally.
- CI runs lint (ruff) and the test suite on every push and pull request.
- Calendar versioning (`YYYY.MM.patch`, zero-padded so versions sort lexically), matching Home Assistant conventions.
