<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
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
