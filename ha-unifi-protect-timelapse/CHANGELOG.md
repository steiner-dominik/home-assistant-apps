<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 26.09.04

### Fixed

- **Skip TLS verification had no effect on the anonymous snapshot source.** It
  only ever applied to the Protect connection, so an `https://` snapshot URL
  with a self-signed certificate — the normal case for a camera on a local
  address — failed however the option was set. It now applies to every camera
  request. The option is renamed to **Skip TLS verification**
  (`camera_insecure_tls`); set it again if you had the old one on.
- **The app restarted every few minutes with nothing in its log.** Its health
  check was stricter than the status it reported: it had no startup grace and
  treated an archive it could not read as a failure, so Home Assistant restarted
  it while the panel showed everything as fine. There is now one shared health
  decision, and the app logs every change to it, so a restart always has a
  visible reason.
- **The staleness limit no longer assumes images arrive continuously.** It
  defaulted to three capture intervals even here, where nothing is captured, so
  an archive filled by a nightly bulk transfer looked broken all day. Leave it
  empty for the new default of 26 hours.
- **A full archive could be reported as empty.** The scan for the newest image
  gave up on the first unrelated directory, and names such as `@eaDir`,
  `#recycle` or `.snapshot` sort above a four digit year. It now looks only at
  date directories and steps past empty ones.

### Changed

- Defaults to the anonymous snapshot source with no fallback, so a fresh install
  works once you fill in the snapshot URL. Previously it defaulted to the
  Protect source and refused to start until every Protect field was set.

## 26.09.03

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

## 26.09.01 and 26.09.02

Published as images but never usable as an app: 26.09.01 ignored the configured
archive path and lost its state on restart, and 26.09.02 could not read its
configuration at all.
