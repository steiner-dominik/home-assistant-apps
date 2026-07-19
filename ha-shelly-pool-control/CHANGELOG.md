<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 2026.07.19.1

Initial release.

- Supervisory panel for a solar pool heating controlled by a Shelly 2PM/1PM
  Gen2+ — the safety-critical ΔT control loop runs on the Shelly itself
  (`pool-control` script), so the pool keeps running without network, server
  or Home Assistant.
- Dashboard with live temperatures, ΔT, pump power, mode switcher and a
  decision feed ("why is it (not) running?").
- Every control parameter configurable with units, ranges, defaults and
  inline help; changes are confirmed by the device (revision handshake).
- Fault policies (sensor failure, dry run, overload, no power, mat
  overtemperature), history charts, fault/audit journals, notifications
  (SMTP / Telegram / webhook), backups with schedule + restore.
- Ingress support (HA authenticates, role configurable), optional MQTT
  Discovery entities, optional InfluxDB v2 mirror.
- Dark/light/auto theme, English & German, installable as a PWA.
