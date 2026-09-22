<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 26.09.08

- **Three views: Summary, Simple, Expert.** *Summary* is the page for the customer: the
  verdict, the two buying decisions and where the memory goes. *Expert* replaces
  "Everything" and adds a data-quality panel.
- **One verdict per buying decision.** *New servers* (less DRAM plus an NVMe tier) and
  *Existing hosts* (add an NVMe tier) each get their own verdict and headline number.
- **Half-DRAM test:** active P95 ÷ assigned at or below 25% (with a 1:1 tier) means DRAM can
  shrink to half of what is assigned.
- **Conservative DRAM saving**, sized on assigned memory; the measured figure is shown next to
  it. Hosts that already run a tier are left out of the saving.
- **Extra memory is gated by CPU**: only hosts whose memory is full while their sustained CPU
  idles count, capped at what that CPU can run. `cpu_idle_pct` now compares against sustained
  CPU (P95 of the interval averages) instead of the P95 of the peaks.
- New sections: where the memory goes, the three questions per cluster, a host map of memory
  used against CPU, and data quality.
- Fixes: empty "Support:" in the report header, raw `noTiering` in the host table, vSphere
  Pods filling the "Hot" VM list.

## 26.09.07

- **Memory tier usage is read where vSphere publishes it.** vCenter 9.x offers
  `mem.tier.consumed.latest` per tier; vCenter 8.0 U3 offers no tiering counter at all.
  Where it exists, "cold in DRAM" and "on NVMe" are measured rather than derived from
  consumed memory and DRAM size. Older vCenters keep the estimate, which is close.
- **The NVMe tier switch starts from your hardware.** Hosts that already run a tier know
  their own DRAM:NVMe ratio, so the switch defaults to it instead of to `tier_ratio` -
  a host with 128 GB DRAM and 512 GB NVMe now sizes at 400%, not 100%.
- New host CSV columns `TierDramMB` and `TierNvmeMB`, empty before vSphere 9.

Tested against vCenter 8.0.3 and 9.1.1 with both the Python and PowerShell editions.

## 26.09.06

- **Host CPU is collected**, and with it the case where a tier replaces a purchase: a host
  whose memory is full while its CPUs idle gains capacity from an NVMe tier instead of from
  another socket. New figure "Tier instead of a new host", a badge in the host table, and
  CPU P95 and core columns. Two new options set the thresholds: `ram_bound_pct` (70) and
  `cpu_idle_pct` (50).
- A host that is memory bound *and* CPU bound is correctly not flagged - that one needs a
  host, not a tier.
- The CPU counter is optional: a vCenter that does not publish it still produces the full
  memory report, with the CPU columns left empty.

Monthly host CSV files gain six columns and are widened in place on the first run.

## 26.09.05

- **Fixes the blank report in 26.09.04.** Removing the failover section left two
  references to elements that went with it; in a browser those are `null`, and the
  resulting error happened before the first render, so every card stayed empty.
- Tier counter discovery now scans every counter group, not only `mem.*`, and the log
  reports how many counters the vCenter offered when none match.

## 26.09.04

- **The "cluster failover headroom" section is gone.** It compared consumed memory with
  the capacity surviving a failure - ordinary HA admission control, which vCenter answers
  better and which says nothing about tiering.
- **What replaced it is the tiering-specific question**: after a host or site is lost, the
  same hot working set lands on fewer hosts, so it has to fit less DRAM. A cluster sized
  right at the 50% limit crosses it the moment a host dies. Now a badge and a figure.
- **Stretched clusters are a sizing input**, not a section. The new "failure to survive"
  switch decides whether sizing reserves capacity for a failure; it defaults to none.
- **The heatmap now shows active over consumed memory** - when the working set is hottest,
  which is when a tier is under the most pressure.
- **DRAM is sized in DIMMs**: rounded up to a population you can order (16 to 256 GB
  modules, up to 48 per host) and named, e.g. `20 x 32 GB`.
- **The candidate bars show a "DRAM after tiering" line** - the active bar has to sit under it.
- **Memory tier counters are discovered** and listed on `sensor.memtier_status` as
  `tier_counters`, so a later release can report how much each host currently keeps on NVMe.

Monthly CSV files gain a `TierCounters` column. Existing files are widened in place on the
first run after the update; nothing has to be moved away.

## 26.09.03

- **The report opens in Simple view**: a one-line verdict ("Strong candidate - 90% of
  the memory your hosts back is cold"), four figures and two tables. Everything else -
  host and cluster charts, failover headroom, the heatmap, the per-host and per-VM
  tables - moves behind an **Everything** switch, remembered per browser.
- **Simulate the NVMe tier size** in the report: 50%, 100% (the 1:1 default), 200% or
  400% of DRAM, the way it is configured on the host. It moves the sizing numbers only;
  whether a cluster is a candidate does not depend on the ratio. The `tier_ratio` option
  sets where the switch starts.

## 26.09.02

- **Active vs. consumed memory is now the headline of the report** - the metric the
  tiering decision is actually made on. At or below 40% (configurable) a cluster is a
  candidate: the rest of the memory its hosts back is cold and an NVMe tier can absorb it.
- **Active as a share of DRAM is shown as what it is**: a feasibility check that limits
  tiering on hardware you already own, not a measure of the benefit.
- **New sizing section**: how much DRAM new hosts would need, how much you would save,
  and how much more memory today's DRAM could back once a tier is added.
- **Hosts that already have an NVMe tier** no longer have their cold memory counted
  twice: it is capped at DRAM, and what already sits on NVMe is shown separately.
- **The report is translatable**, like this panel: English and German, switchable in the
  report and remembered per browser.
- **Collection interval** is now an option: 60, 30 or 15 minutes. A shorter interval
  gives a finer time resolution and loses less data when a run fails; it does not find
  peaks an hourly run misses, because every run already reads all 20-second samples.
- Two new entities: `sensor.memtier_active_of_consumed` and `sensor.memtier_cold_in_dram`.

## 26.09.01

First release.

- **Hourly collection** of active and consumed memory for every host and VM,
  read from vCenter with a read-only account. Nothing is changed in vCenter.
  Collection starts as soon as the connection is configured.
- **Trend report in the sidebar**: memory tiering candidates, host charts against
  the 50% active memory guidance, N+1 and stretched cluster failover headroom,
  weekday × hour heatmap and per-VM sizing. Rebuilt after every run.
- **Status, run history and CSV downloads.** Failed and partial runs show up with
  the vCenter's own error message.
- **Health entities** written straight to the Core API: collector status, last
  collection, hosts, powered-on VMs and the busiest host's active memory.
- English and German interface.
