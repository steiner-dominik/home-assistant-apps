# Home Assistant App: Tesla Invoices

## Requirements

To use this app, you need a Tesla `access_token` and `refresh_token`,
generated with one of these applications:

- Android: [Tesla Tokens](https://play.google.com/store/apps/details?id=net.leveugle.teslatokens)
- iOS: [Auth App for Tesla](https://apps.apple.com/us/app/auth-app-for-tesla/id1552058613)
- TeslaFi: [Tesla v3 API Tokens](https://support.teslafi.com/en/communities/1/topics/16979-tesla-v3-api-tokens)
- Chromium/Edge: [Chromium Tesla Token Generator](https://github.com/DoctorMcKay/chromium-tesla-token-generator)

## Installation

1. Add this app repository to Home Assistant:

   [![Open app repo on your Home Assistant instance][repo-btn]][repo-link]

   or add `https://github.com/steiner-dominik/home-assistant-apps` manually under
   **Settings → Apps → App Store → ⋮ → Repositories**.

2. Install the "Tesla Invoices" app.
3. Configure at least `refresh_token` (see [Requirements](#requirements)).
4. Start the app.

[repo-link]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsteiner-dominik%2Fhome-assistant-apps
[repo-btn]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg

## Configuration

```yaml
access_token: ""
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

### Option: `access_token` (optional)

The access token retrieved from one of the authentication apps (see
[Requirements](#requirements)). It can be left empty: with only a
`refresh_token` configured, the app obtains a fresh access token on its
own. Either way it refreshes and stores its own tokens after the first start.

### Option: `refresh_token` (required)

The refresh token retrieved from one of the authentication apps (see
[Requirements](#requirements)).

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
- Open the app's **web UI** (ingress) to see the analytics dashboard with
  monthly energy/cost charts, filter and search invoices, view or download
  the PDFs, and export everything as CSV.
- To download your **complete invoice history**, click **"Sync all history"**
  on the **Maintenance** tab. A specific month can also be fetched via the
  API: `POST api/sync?month=2025-11` (relative to the ingress URL; also
  `all`, `cur`, `prev`).
- `refresh_token` and `access_token` are stored inside `/data/` in the
  container. Tokens from the app options are only used if they are newer.
  To switch to a different Tesla account, simply configure freshly generated
  tokens — being newer, they win over the stored ones automatically.
