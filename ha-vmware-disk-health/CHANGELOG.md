<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 26.09.22.4

### Added

- **vSAN tier per disk.** Disks claimed by vSAN (OSA) show a *vSAN cache* or
  *vSAN capacity* badge and their disk group, from `esxcli vsan storage list`
  (read-only; hosts without vSAN simply skip it). Also in the entity
  attributes and as `vmware_disk_health_disk_vsan_info` in Prometheus.
- **Estimated wear for drives that report none**, from data written against
  the vendor's rated endurance (TBW): Intel DC S3500/S3510/S3520/S3700/S3710
  and Crucial MX500. Clearly marked as an estimate (hatched bar, "est.", its own
  *Estimated life remaining* entity), with a projected wear-out date. It can
  raise a warning, never a critical status.
- **A disk that disappears from its host is now an event and a notification**,
  and its entities turn *unknown* with the problem sensor on. Its return is
  reported too.

### Changed

- **SSDs no longer warn forever about a few reallocated sectors.** Retiring
  worn blocks is normal for an SSD: a stable count below
  `ssd_reallocated_warn_count` (new option, default 10) is OK, and any
  increase within a week still warns. Hard disks are unchanged.

### Fixed

- **"Data written per day" showed one huge spike after updating to 26.09.22.1**
  on Intel/Solidigm SSDs, and would have turned into a wildly wrong write rate:
  history recorded in the old unit is now converted on startup, and steps no
  drive could physically write are ignored.
- Two instances on the same MQTT broker (for example home and lab) kept
  disconnecting each other: both used the same client id inside Docker.
- An unreachable host reported 0 disks and 0 problem disks and lost its
  "last successful poll"; these are now unknown / kept.
- Disks excluded after they were first seen no longer show up as missing.
- SATA SSDs behind a SAS controller were labelled as SAS drives.
- History charts of error counters showed repeated axis labels ("1, 1, 1").
- The Gotify token is sent as a header instead of in the URL; SMTP on port
  465 uses implicit TLS.
- The database is safe against concurrent access from the scheduler and the
  web UI, and the SSH key file is never briefly readable by other users.

## 26.09.22.3

### Added

- **The disk details pane now also shows esxcli's raw device properties**
  (`esxcli storage core device list`), alongside the existing raw SMART data —
  useful for checking exactly what ESXi reports for a drive without SSH'ing
  into the host.

## 26.09.22.2

### Fixed

- **Life remaining showed 100% on Intel/Solidigm SSDs no matter how worn
  they actually were.** The same attribute aliasing behind the write/read
  fix also stuck the wear-level value at 100 on those drives — confirmed
  against a live host where three drives with petabytes written all showed
  100%, while Dell's own out-of-band monitoring (iDRAC) reported real wear
  (68-88% remaining) for the same disks. There is no way to recover the true
  percentage from esxcli alone, so it now shows as unknown instead of a
  misleading 0% used.

## 26.09.22.1

### Fixed

- **Data written/read was far too low on Intel/Solidigm SATA SSDs.**
  `esxcli storage core device smart get` uses one fixed attribute name for
  every vendor, but Intel/Solidigm data-center SATA SSDs (model codes
  starting `SSDSC`, e.g. the D3-S4610) count that attribute in 32 MiB units
  instead of sectors, undercounting written/read data by a factor of 65536.
  A drive that actually wrote ~401 TB over 9 years showed 6.1 GB.

## 26.09.22

### Fixed

- **Disks without a serial number could not be told apart either.** The
  previous release named devices "<model> <serial>", but drives that report
  their own WWN get `naa.*`/`eui.*` device ids from ESXi instead of one that
  encodes a serial — which esxcli alone cannot recover without the smartctl
  VIB. Those devices fell back to the bare model name and collided again. They
  now get a short id suffix instead, both in Home Assistant and the web UI.

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
