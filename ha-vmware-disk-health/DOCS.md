# VMware Disk Health

> ⚠️ **This is an independent community project. It is not affiliated with,
> endorsed by, or supported by Broadcom Inc. or VMware.**

Watches the disks in your ESXi hosts and tells you before one of them fails:
temperature, remaining SSD endurance, data written, and the error counters that
actually predict failures — reallocated, pending and uncorrectable sectors.

## Installation

1. Add this app repository to Home Assistant:

   [![Open app repo on your Home Assistant instance][repo-btn]][repo-link]

   or add `https://github.com/steiner-dominik/home-assistant-apps` manually under
   **Settings → Apps → App Store → ⋮ → Repositories**.

2. Install the "VMware Disk Health" app.
3. Enter your hosts on the **Configuration** tab (see below) and start the app.
4. Open the panel and follow the **Setup** page: it shows the SSH key to
   authorize on each host, and tests the connection.

[repo-link]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsteiner-dominik%2Fhome-assistant-apps
[repo-btn]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg

## Giving the app access to a host

ESXi has no read-only SMART interface, so the app logs in over SSH as `root`
and runs read-only commands (`esxcli storage core device …`, `esxcli nvme …`).
It never writes to the hosts.

1. **Enable SSH** on the host: in the ESXi web interface, **Host → Actions →
   Services → Enable Secure Shell (SSH)**.
2. **Authorize the app's key.** The Setup page shows a ready-made command; run
   it once on the host (ESXi Shell, or an SSH session as root):

   ```sh
   echo 'ssh-ed25519 AAAA… vmware-disk-health' >> /etc/ssh/keys-root/authorized_keys
   ```

   A password can be used instead, but the key is safer and survives password
   changes.
3. Press **Test connection** on the Setup page.

The host's key is remembered the first time the app connects, and a changed key
is refused afterwards — so a host that is impersonated later is noticed. After
legitimately reinstalling a host, use **Forget host key** on the Setup page.

## Configuration

```yaml
hosts:
  - name: esxi-01           # shown in the UI and in entity names
    address: esxi-01.lan    # hostname or IP
    # port: 22
    # username: root
    # password: ""          # leave empty to use the app's SSH key
    # smartctl_path: auto   # "none" never uses smartctl
    # exclude: "*USB*, t10.ATA_____SomeDisk*"
poll_interval_minutes: 60
retention_raw_days: 90      # then one row per day is kept, forever
```

SMART values change slowly, so hourly polling is plenty; every poll is a handful
of SSH commands per host.

**Thresholds** decide when a disk is a warning or critical:

| Option | Default |
|---|---|
| `life_remaining_warn_pct` / `life_remaining_crit_pct` | 20 % / 10 % |
| `temp_hdd_warn_c` / `temp_hdd_crit_c` | 50 / 60 °C |
| `temp_ssd_warn_c` / `temp_ssd_crit_c` | 60 / 70 °C |
| `temp_nvme_warn_c` / `temp_nvme_crit_c` | 70 / 80 °C |
| `use_drive_temp_limit` | `true` — never warn later than the drive's own limit |

Sector and error counters are not configurable. A pending sector is a warning
when first seen and critical when the drive still reports it at the next poll
(drives clear these again by themselves); an uncorrectable sector is always
critical, a reallocated sector a warning, and a counter that grows within a
week is critical.

**Per disk** you can rename a drive, ignore it, or change its thresholds:

```yaml
disk_overrides:
  - match: "S3F2NWBHB56997P"   # serial, device id or model; * and ? allowed
    name: "Boot SSD"
  - match: "*PiKVM*"
    ignore: true
```

## Entities

With the Mosquitto broker installed (or any broker configured under `mqtt`),
every disk becomes a **device** under the MQTT integration, linked to its host:

| Entity | |
|---|---|
| `sensor.<disk>_status` | ok / warning / critical / unknown, with the findings as attributes |
| `binary_sensor.<disk>_problem` | on for warning and critical — the one to automate on |
| `sensor.<disk>_temperature` | °C |
| `sensor.<disk>_life_remaining`, `…_endurance_used` | % (SSD and NVMe) |
| `sensor.<disk>_data_written`, `…_data_read` | bytes, as a total |
| `sensor.<disk>_power_on_time`, `…_power_cycles`, `…_unsafe_shutdowns` | |
| `sensor.<disk>_reallocated_sectors`, `…_pending_sectors`, `…_crc_errors`, `…_media_errors`, … | error counters the drive reports |

Each host also gets a device with **Reachable**, **Last successful poll**,
**Disks** and **Disks with problems**.

Only values a drive actually reports become entities, so there are no entities
that stay unknown forever.

A simple automation:

```yaml
automation:
  - alias: Disk problem
    triggers:
      - trigger: state
        entity_id: binary_sensor.samsung_ssd_750_evo_120gb_problem
        to: "on"
    actions:
      - action: notify.persistent_notification
        data:
          title: "Disk problem"
          message: >-
            {{ state_attr('sensor.samsung_ssd_750_evo_120gb_status', 'findings') | join(', ') }}
```

If you would rather have the app notify you directly (ntfy, Gotify or email),
turn on `alerts` in the configuration.

## What is collected

| Source | On the host | Used for |
|---|---|---|
| `esxcli storage core device list` / `… capacity list` | always | which disks exist, size, sector format |
| `esxcli storage core device smart get` | always | health, temperature, power-on hours, data written, sector counters |
| `esxcli nvme device log smart get` | always | NVMe health log: wear, spare, media errors, unsafe shutdowns |
| `smartctl` ([community VIB][vib]) | only if installed | the full ATA attribute table, CRC errors, vendor wear attributes |

[vib]: https://github.com/bsv9/smartctl-esxi-vib

smartctl is **optional**; it is used when present and never installed by the
app. It cannot read NVMe drives on ESXi, so those always come from `esxcli`.

Some drives report less than others. A Crucial MX500, for example, tells ESXi
nothing about its wear, so remaining life only shows up there with smartctl
installed.

## Troubleshooting

- **"Connection failed"** on the Setup page: SSH is disabled on the host, or the
  key is not authorized yet. The message is the SSH error verbatim.
- **A disk shows "Unknown":** the host answered, but the drive returned no SMART
  data — common for disks behind a RAID controller, which hides them from ESXi.
- **A disk shows "Missing":** it was there before but not in the latest
  collection; the values shown are the last known ones.
- **No entities in Home Assistant:** check the Setup page under
  "Home Assistant & notifications". Without a broker (install the Mosquitto
  broker app), the panel still works, but there are no entities.
- The app log shows every status change and every failed host.
