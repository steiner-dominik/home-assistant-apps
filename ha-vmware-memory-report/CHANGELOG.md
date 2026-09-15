<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
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
