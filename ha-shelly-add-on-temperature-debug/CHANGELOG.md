<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
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
