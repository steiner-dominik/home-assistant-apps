# VMware Memory Tiering Report

Find out whether **NVMe memory tiering** (vSphere 8.0 U3 / VCF 9) can cut your
VMware memory bill, based on weeks of real data instead of a guess.

Memory tiering moves cold memory pages to NVMe, so a host can offer up to twice
its DRAM. It only works if the **active** memory fits into DRAM; Broadcom's
guidance is to keep it at or below 50% of DRAM. A single look at vCenter won't
tell you, because active memory swings with backups, batch jobs and month-end.
This app measures it every hour, for as long as you let it run.

Full documentation:
<https://github.com/steiner-dominik/vmware-memory-report>

## Setup

1. **Create a read-only account in vCenter**, for example
   `svc-memtier@vsphere.local` with the built-in *Read-only* role on the vCenter
   root object. The app never changes anything.
2. On the **Configuration** tab, set:
   - `servers` — the vCenter FQDN. Add one entry per vCenter.
   - `username` and `password` of that account.
3. Start the app and open **Memory Tiering** in the sidebar.

The first collection starts right away and takes a few seconds to a few minutes,
depending on the number of VMs. After that it runs every hour and the report is
rebuilt after every run. Let it run for **a few weeks** before you decide: one
day of data does not include your month-end or your backup windows.

Home Assistant must be able to reach vCenter on port 443.

## TLS certificates

Certificate verification is on by default. A vCenter with its own VMCA
certificate is not trusted by the system certificate store, so the first
collection fails with a certificate error. Either:

- **Trust the vCenter root CA (recommended).** Download it from the vCenter start
  page (*Download trusted root CA certificates*), copy the `.crt`/`.pem` file into
  the `ssl` folder of Home Assistant (for example with the Samba or File editor
  app) and set `ca_file` to `/ssl/vcenter-ca.pem`.
- **Or turn `verify_tls` off.** Acceptable for a lab; the password is then sent
  to whatever answers on that address.

## The panel

- **Report** — the trend report: memory tiering candidates per cluster, host
  charts against the guidance, cluster failover headroom (N+1 and stretched
  clusters), a weekday × hour heatmap and per-VM sizing. It is a single HTML file
  and can be downloaded from the **Data** tab to share it.
- **Status** — the last collection per vCenter, the next run, the busiest host
  and the app log. **Collect now** starts a run immediately (at most every
  5 minutes).
- **Runs** — every collection with its result. Gaps and failed runs are where the
  report has no data.
- **Data** — the monthly CSV files. They use the same format as the PowerShell
  and Python scripts, so data collected here can be reported by either edition.
- **Settings** — the running configuration. The password is only shown as set
  or not set.

## Options

| Option | Meaning |
|---|---|
| `servers` | vCenter FQDNs, one per entry |
| `username`, `password` | Read-only vCenter account |
| `verify_tls`, `ca_file` | Certificate verification, see above |
| `report_days` | Days of history shown in the report |
| `threshold_pct` | Tiering guidance: host active memory as % of DRAM |
| `stretched_cluster` | All clusters are stretched: after a site failure 50% of the cluster remains, instead of N+1 |
| `stretched_clusters` | Or name the stretched clusters individually |
| `cold_pct`, `hot_pct` | Per-VM sizing hints: worst-day P95 active memory as % of configured memory |
| `exclude_vm_pattern` | VMs whose name matches this regular expression are skipped. `^$` skips none |
| `retention_months` | CSV files older than this are deleted, `0` keeps everything |
| `title`, `support_contact` | Shown in the report header |
| `publish_entities`, `entity_prefix` | Home Assistant entities, see below |
| `language` | Default language of the panel; each browser can switch it |

Restart the app after changing an option. The report is rebuilt at startup, so
new thresholds and stretched clusters apply to the whole history.

## Entities

The app writes these straight to the Core API using the Supervisor token. No
MQTT broker and no template sensors are involved. `entity_prefix` renames them.
They are refreshed every 10 minutes, so they come back shortly after Home
Assistant restarts.

| Entity | Meaning |
|---|---|
| `sensor.memtier_status` | `ok`, `partial`, `failed`, `collecting`, `waiting` or `unconfigured` |
| `sensor.memtier_last_collection` | Time of the last collection |
| `sensor.memtier_hosts` | Hosts collected in the last run |
| `sensor.memtier_vms` | Powered-on VMs in the last run |
| `sensor.memtier_peak_host_active` | P95 active memory of the busiest host, as % of its DRAM |

An automation that tells you when collection stops working:

```yaml
automation:
  - alias: Memory tiering collection failing
    triggers:
      - trigger: state
        entity_id: sensor.memtier_status
        to: failed
        for: "02:00:00"
    actions:
      - action: notify.persistent_notification
        data:
          message: >-
            VMware memory collection has been failing for two hours:
            {{ state_attr('sensor.memtier_status', 'message') }}
```

## Data and backups

`/data` holds the CSV history, the report and the scheduler state. It is
included in Home Assistant backups. Plan for roughly 12 MB per month for every
100 powered-on VMs while the month is running; completed months are compressed
to about a tenth of that. `retention_months` limits how much is kept.

The collector reads the last hour of 20-second samples that each ESXi host keeps.
While the app is stopped, nothing is collected, and the report shows the gap.

## Notes

- Ingress only: the app's port is not published, and Home Assistant
  authenticates every request.
- Supported: vCenter 8.0 U1 or later. NVMe tier sizes are read on ESXi 8.0 U3 or
  later; older hosts are reported with DRAM only.
- Prefer a Windows jump host or Docker instead? The same collector is available
  as PowerShell and Python scripts and as a standalone container:
  <https://github.com/steiner-dominik/vmware-memory-report>
- This is an independent community project, not affiliated with, endorsed by or
  supported by VMware, Broadcom or the Home Assistant project. VMware, vSphere,
  vCenter and VCF are trademarks of Broadcom.
