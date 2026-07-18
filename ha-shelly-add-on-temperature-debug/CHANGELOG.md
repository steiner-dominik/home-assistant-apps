<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
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
