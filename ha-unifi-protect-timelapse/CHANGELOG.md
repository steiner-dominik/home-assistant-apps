<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 26.09.03

Initial working release.

Fixes the app failing to start with
`reading /data/options.json: permission denied`. The image dropped privileges,
but Home Assistant writes an app's configuration as root and does not change the
container's user, so the app could not read its own settings.

- Timelapse capture and archive watchdog for a UniFi Protect camera, packaged
  for Home Assistant with ingress support.
- Watchdog by default: probes the camera and checks how fresh the archive is
  without writing anything, so it can run alongside a capturing deployment
  without disturbing its interval.
- Optional capturing, if you would rather Home Assistant took the images
  itself — same schedule, buffering and archive layout as the standalone
  deployment.
- Two snapshot sources — the camera's anonymous endpoint and the UniFi Protect
  integration API — with automatic fallback between them.
- Native Home Assistant entities written straight to the Core API with the
  Supervisor token: camera reachability, archive availability, a frozen-camera
  problem sensor, newest archived image, buffered images and an overall status.
  No MQTT broker and no template sensors.
- Archive browser with playback, gap detection, MP4 export and ZIP download.
- Live view with an on-demand preview that is never written to the archive.
- Dark/light/auto theme, English and German, installable as a PWA.

Versions 26.09.01 and 26.09.02 of the image exist but were never usable as an
app: 26.09.01 ignored the configured archive path and lost its state on restart,
and 26.09.02 could not read its configuration at all.
