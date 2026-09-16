# VMware Disk Health (Home Assistant app)

SMART and SSD health monitoring for VMware ESXi hosts: wear, data written,
temperatures and error counters of every local SATA and NVMe disk, with
history, Home Assistant entities and notifications.

The app reads the data over SSH with read-only commands and installs nothing on
the hosts.

Source, documentation and standalone Docker deployment:
<https://github.com/steiner-dominik/vmware-disk-health>
