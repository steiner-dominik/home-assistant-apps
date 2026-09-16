<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
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
