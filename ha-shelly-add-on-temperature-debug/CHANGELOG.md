<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 2026.07.19.2

- New: the app icon follows the system's **dark/light theme** (favicon and
  PWA icons).
- Opening the page **no longer queries the devices** — it shows the newest
  cached result instead, so page views don't add samples to the history.
  Fresh readings still come from background polling, auto-refresh, or the
  query button.
- The summary bar now groups the current readings **per Shelly**, each
  sensor as a small round bubble (failing sensors highlighted).
- Compact toolbar: shorter button labels (full text in the tooltip), fits
  one row on desktop — noticeably better in German and on phones.
- Charts: **more x-axis timestamps** on even local-time boundaries, and
  axis text is no longer stretched on narrow screens.
- Installed PWAs now **reload automatically** when a new version is
  deployed (no more empty screen after an update).

## 2026.07.19.1

- New: **time-range selector** for the history charts (15 min … 7 days,
  default: last 24 h); the wiggle test automatically zooms to the last
  15 minutes.
- New: the dashboard is now **live** — it picks up new readings from
  background polling every 5 seconds, no clicking needed.
- New: the summary bar at the top shows the **current reading of every
  sensor** at a glance.
- The status legend in the footer is now an expandable, readable table
  covering all statuses.
- Slightly more verbose logging (status changes, provisioning steps) while
  staying quiet in steady state.

## 2026.07.18.4

- New: **sensor provisioning** — with the new `provision_passphrase` option
  set, the page can scan the Sensor Add-on's 1-Wire bus and attach + name
  newly connected DS18B20 probes (the Shelly reboots briefly to activate
  them). Lets a helper provision sensors without the Shelly web UI or admin
  password; disabled while the option is empty.
- Redesigned app icons (page favicon, PWA icons, and this app's store icon).
- Cleaner page footer.

## 2026.07.18.3

- Fix startup as a Home Assistant app: the options file mounted by the
  Supervisor is readable only by root, so the container no longer bakes in
  a non-root user ("configuration error: reading /data/options.json:
  permission denied" on start). 2026.07.18.2 never started as an app —
  this is the first working release.

## 2026.07.18.2

- Initial release as a Home Assistant app: live sensor status with failure
  guidance, per-sensor queries, wiggle test, history charts with CSV export,
  trend arrows, English/German, dark/light theme, optional Prometheus
  metrics.
- Devices are configured on the app's configuration page (`devices` list —
  any number of Shellys); via ingress no token or port setup is needed.
- Background polling (default: every 60 s) keeps the history filling so
  charts are already populated when the page is opened.
- History is bounded by a total memory budget (`history_max_mb`).
- Installable as a PWA with a mobile layout tuned for phones.
