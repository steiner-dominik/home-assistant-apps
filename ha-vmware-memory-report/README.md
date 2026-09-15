# VMware Memory Tiering Report (Home Assistant app)

Find out whether NVMe memory tiering can cut your VMware memory bill, based on
weeks of real data instead of a guess. The app reads active and consumed memory
from vCenter every hour with a read-only account, and shows a self-contained
trend report with tiering candidates, failover headroom and oversized VMs
inside Home Assistant.

Source, documentation, the standalone PowerShell and Python scripts and the
Docker deployment:
<https://github.com/steiner-dominik/vmware-memory-report>
