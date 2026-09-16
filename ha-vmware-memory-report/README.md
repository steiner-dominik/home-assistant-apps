# VMware Memory Tiering Report (Home Assistant app)

Find out whether NVMe memory tiering can cut your VMware memory bill, based on
weeks of real data instead of a guess. The app reads active and consumed memory
from vCenter every 15, 30 or 60 minutes with a read-only account, and shows a
self-contained trend report inside Home Assistant: tiering candidates ranked by
active vs. consumed memory, the DRAM you would save, failover headroom and
oversized VMs. English and German.

Source, documentation, the standalone PowerShell and Python scripts and the
Docker deployment:
<https://github.com/steiner-dominik/vmware-memory-report>
