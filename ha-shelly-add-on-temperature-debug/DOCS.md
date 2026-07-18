# Home Assistant App: Shelly Add-on Temperature Debug

> ⚠️ **This is an independent community project. It is not affiliated with,
> endorsed by, or supported by Shelly Group / Allterco Robotics in any way.**
> "Shelly" is used here only to describe compatibility.

## Installation

1. Add this app repository to Home Assistant:

   [![Open app repo on your Home Assistant instance][repo-btn]][repo-link]

   or add `https://github.com/steiner-dominik/home-assistant-apps` manually under
   **Settings → Apps → App Store → ⋮ → Repositories**.

2. Install the "Shelly Add-on Temperature Debug" app.
3. Open the **Configuration** tab and add your Shelly device(s) — see below.
4. **Start the app** and open its web UI (**Open Web UI** / sidebar entry).

[repo-link]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsteiner-dominik%2Fhome-assistant-apps
[repo-btn]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg

## Configuration

Only `devices` is required — everything else has sensible defaults.

```yaml
devices:
  - host: 192.168.1.50
    name: Pool
  - host: 192.168.1.51
    name: Garden
    password: individual-admin-password
shelly_password: fallback-admin-password
background_poll_seconds: 60
```

### Option: `devices` (required)

The list of Shelly devices (Gen2/Gen3/Gen4 with a Sensor Add-on) to query.
Add one entry per device; each entry has:

- `host` (required): IP address or hostname of the Shelly, e.g.
  `192.168.1.50` (`http://` is assumed if no scheme is given)
- `name` (optional): display name on the page — defaults to the host
- `password` (optional): the device's admin password, if this device's
  password differs from `shelly_password`
- `user` (optional): auth user, defaults to `admin` (Gen2+ is always `admin`)

### Option: `shelly_password`

Fallback admin password used for every device that has no `password` of its
own. Leave empty if your Shellys have authentication disabled.

> 🔒 The Shelly password never leaves the server side: the browser only ever
> talks to the app, all device queries are read-only, and they are
> rate-limited.

### Option: `background_poll_seconds`

Default `60`: the app itself polls all devices on this interval, so the
history charts are already populated whenever you open the page. Set `0` to
disable — queries then only happen while somebody is using the page.

### Option: `history_max_mb`

Total memory budget (in MB, default 16) for the in-memory history, shared by
all sensors. When it is full, the oldest samples are dropped. History lives
in RAM only and is lost when the app restarts.

### Options: `auto_refresh_seconds`, `auto_refresh_default`

Interval of the page's auto-refresh (default 30 s) and whether the toggle
starts enabled for browsers that never touched it (default off).

### Options: `query_timeout_seconds`, `query_min_interval_seconds`

Per-device query timeout (default 5 s) and the rate limit between real
device queries (default 2 s; faster requests share a cached result), so the
page can never hammer your Shellys.

### Option: `metrics_enabled`

Set to `true` to expose Prometheus metrics (temperatures, humidity, sensor
health, Wi-Fi RSSI, uptime) at the app's `/metrics` endpoint. To scrape it
from outside, also expose the port (below).

### Option: `provision_passphrase`

Off by default. When set, the page gets an **Add sensors** panel that can
provision brand-new DS18B20 probes without anyone touching the Shelly web
UI: connect the probe to the Sensor Add-on's terminals, scan the 1-Wire bus
from the page, enter the desired sensor name, and the app attaches the probe
(`SensorAddon.AddPeripheral`), reboots the Shelly to activate it, and sets
the name.

The panel asks for this passphrase **in addition** to the normal page
access — so you can hand the passphrase to whoever wires up new sensors
without giving them the Shelly admin password. Anyone with the passphrase
can add sensors and thereby briefly restart the device (its outputs may
switch during the restart), so pick a long random value. Leave empty to
disable provisioning entirely (the API then does not exist).

### Option: `debug_token` and the optional port

Via ingress (the normal way) Home Assistant authenticates every request and
no token is needed. If you additionally map the app's port in the app's
**Network** section (e.g. to let a Prometheus server scrape metrics, or to
share the page without Home Assistant accounts), set a long random
`debug_token` — that port would otherwise be open to everyone on your
network. The token is entered once in the browser (or sent as
`Authorization: Bearer` header) and never appears in URLs.

## Usage

- Open the web UI and press **Query sensors now** — every configured Shelly
  is queried live, with per-sensor status and guidance for anything that is
  not OK (85 °C power-on reset, missing sensor, no reading, …).
- The **wiggle test** polls every 2 seconds for 60 seconds while you
  physically re-seat cables and connectors — contact problems show up live
  in the chart.
- The **history chart** shows this app-session's readings; export everything
  as **CSV** for analysis elsewhere.
- With a `provision_passphrase` configured, **Add sensors** lets a helper
  attach newly wired DS18B20 probes (name asked before adding) without
  access to the Shelly web UI.
- The page is available in **English and German**, follows your light/dark
  preference, and can be **installed as a PWA** on your phone.

## Support the project

If this app is useful to you, you can support its development via
[GitHub Sponsors](https://github.com/sponsors/steiner-dominik),
[Ko-fi](https://ko-fi.com/dominik_steiner), or
[Buy Me a Coffee](https://buymeacoffee.com/dominik.st).

## Disclaimer

**This project is not affiliated with, endorsed by, sponsored by, or in any
way officially connected to Shelly Group / Allterco Robotics** or any of its
subsidiaries. All product names, trademarks and registered trademarks are
property of their respective owners. This software is provided "as is" and
without any warranty; use at your own risk.
