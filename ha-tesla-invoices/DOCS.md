# Home Assistant App: Tesla Invoices

> ⚠️ **This is an independent community project. It is not affiliated with,
> endorsed by, or supported by Tesla, Inc. in any way.**

## Requirements

To use this app, you need a Tesla **refresh token**, generated with one of
these applications:

- Windows / macOS / Linux: [tesla_auth](https://github.com/adriankumpf/tesla_auth) (recommended)
- iOS: [Auth app for Tesla](https://apps.apple.com/us/app/auth-app-for-tesla/id1552058613)

> 🔒 **Treat the token like a password** — it grants full access to your
> Tesla account. That is all the app needs: access tokens are obtained,
> renewed and stored automatically.

## Installation

1. Add this app repository to Home Assistant:

   [![Open app repo on your Home Assistant instance][repo-btn]][repo-link]

   or add `https://github.com/steiner-dominik/home-assistant-apps` manually under
   **Settings → Apps → App Store → ⋮ → Repositories**.

2. Install the "Tesla Invoices" app.
3. Configure `refresh_token` (see [Requirements](#requirements)).
4. Start the app.

[repo-link]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsteiner-dominik%2Fhome-assistant-apps
[repo-btn]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg

## Configuration

```yaml
refresh_token: ""
polling_interval: 15
default_currency: ""
enable_email_export: false
enable_subscription_invoice: true
email:
  from: sender_mail@example.com
  to: receiver_mail@example.com
  mailserver: mailserver.example.com
  port: 587
  user: ""
  password: ""
```

### Option: `refresh_token` (required)

The refresh token retrieved from one of the authentication apps (see
[Requirements](#requirements)). This is the only credential the app needs —
access tokens are obtained from it automatically, renewed before they expire,
and stored in the app's private `/data` directory.

> ℹ️ Earlier versions also had an `access_token` option; it was never needed
> and has been removed. If the app reports an invalid `access_token` option
> after updating, remove that line from the configuration
> (*Configuration → three-dot menu → Edit in YAML*).
>
> The [standalone Docker deployment](https://github.com/steiner-dominik/tesla-invoices)
> still accepts an access token as an alternative for users who prefer not to
> store a long-lived credential.

### Option: `polling_interval`

How often (in minutes, 1-1440) the app checks for new invoices. Default: 15.

### Option: `default_currency`

Three-letter currency code (e.g. `EUR`, `USD`) that the dashboard prefers when
invoices exist in several currencies. Leave empty (default) to auto-detect the
currency carrying the largest share of the cost. Costs are **never converted**
between currencies — the dashboard shows per-currency totals instead.

### Option: `enable_email_export`

Set to `true` to send every newly downloaded invoice as an email attachment.
Each invoice is sent exactly once; the sent state is stored in the invoice's
metadata file.

### Option: `enable_subscription_invoice`

Set to `true` (default) to also download subscription invoices (e.g. Premium
Connectivity).

### Option: `email` (required if `enable_email_export` is set)

Dictionary of email server settings:

- `from`: sender address
- `to`: recipient address
- `mailserver`: mail server IP or FQDN
- `port`: SMTP port — 587 (default) uses STARTTLS, 465 uses implicit TLS;
  any other port is treated like 587 (STARTTLS)
- `user`: username to authenticate with the mail server (leave empty for no login)
- `password`: password to authenticate with the mail server

#### Example: Gmail

Gmail requires an **App Password** — your normal account password will not
work. Enable 2-step verification, then create one at
<https://myaccount.google.com/apppasswords>.

```yaml
enable_email_export: true
email:
  from: yourname@gmail.com
  to: recipient@example.com
  mailserver: smtp.gmail.com
  port: 587
  user: yourname@gmail.com
  password: "abcd efgh ijkl mnop"  # 16-character app password
```

With the SMTP settings configured you can also send individual invoices
manually via the **Email** button in the dashboard, even when
`enable_email_export` is `false`. The button asks for the recipient,
pre-filled with `email.to` — manual sends only require `mailserver` and
`from` to be configured.

## Usage

- The app checks for new invoices of the current and previous month on every
  polling interval.
- Timestamps in the log and the month boundaries of the sync window
  automatically use the **time zone configured in Home Assistant** — no
  setup needed.
- Open the app's **web UI** (ingress) to see the analytics dashboard with
  monthly energy/cost charts, filter and search invoices, view or download
  the PDFs, and export everything as CSV.
- To download your **complete invoice history**, click **"Sync all history"**
  on the **Maintenance** tab. A specific month can also be fetched via the
  API: `POST api/sync?month=2025-11` (relative to the ingress URL; also
  `all`, `cur`, `prev`).
- The refresh token (and the automatically obtained access token) are stored
  inside `/data/` in the container. The token from the app options is only
  used when it is newer than the stored one. To switch to a different Tesla
  account, simply configure a freshly generated refresh token — being newer,
  it wins over the stored one automatically.

## Support the project

If this app is useful to you, you can support its development via
[GitHub Sponsors](https://github.com/sponsors/steiner-dominik),
[Ko-fi](https://ko-fi.com/dominik_steiner), or
[Buy Me a Coffee](https://buymeacoffee.com/dominik.st).

## Disclaimer

**This project is not affiliated with, endorsed by, sponsored by, or in any
way officially connected to Tesla, Inc.** or any of its subsidiaries. All
product names, trademarks and registered trademarks are property of their
respective owners. This software is provided "as is" and without any
warranty; use at your own risk.
