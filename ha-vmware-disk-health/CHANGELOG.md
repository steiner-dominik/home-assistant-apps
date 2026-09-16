<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 26.09.18

### Fixed

- **Identical drives could not be told apart.** Eight identical SSDs in one
  host became eight devices with the same name, so their entities ended up
  numbered `..._2` to `..._8`. A device is now named "<model> <serial>" and the
  entity ids follow the serial. The old entities are withdrawn automatically —
  Home Assistant removes them, and the roughly one poll of history they hold
  goes with them.
- **A pending sector no longer flips straight to critical.** Drives report one
  and clear it again minutes later once the sector reads cleanly, which made a
  disk go critical and then back to ok. It is now a warning on first sight, and
  critical only when the drive still reports it at the next poll.

## 26.09.17

### Fixed

- **The Configuration tab refused to save** with "Missing option
  'disk_overrides' in root". Home Assistant needs a default for every option
  that is not marked optional, and a list cannot be marked optional, so the
  per-disk settings now default to an empty list.

## 26.09.16

First release.

- Reads SMART data from ESXi hosts over SSH: health, temperature, power-on
  hours, data written, wear and error counters for SATA, SAS and NVMe disks.
- **Needs nothing installed on the hosts.** ESXi's own `esxcli` SMART data is
  enough; where the community smartctl VIB happens to be installed, the full
  ATA attribute table is used as well.
- Panel with an overview per host, a page per disk with history charts and a
  projection of when an SSD wears out, a log of status changes, and a setup
  page with the SSH key to authorize.
- One Home Assistant device per disk through MQTT discovery, with a status
  sensor, a problem binary sensor and sensors for every value the drive
  reports — ready for automations and notifications.
- Optional notifications from the app itself (ntfy, Gotify, email) for the
  standalone deployment.
- Prometheus metrics at `/metrics` and a JSON API.
- English and German; light, dark or following the system.
