<!-- https://developers.home-assistant.io/docs/apps/presentation#keeping-a-changelog -->
## 26.09.07

### Fixed

- **A panel left open across an update kept running the old code.** The panel
  lives in an iframe you can leave open for days, and an old one fails in ways
  the app's own log knows nothing about — a panel from before 26.09.05 reports
  "camera unreachable" from a bug fixed back then, while the server is answering
  perfectly. The panel now reloads itself once when it notices the app has been
  updated.

  **If the live preview is still failing, reload the panel** (or your browser
  tab) once after updating. From this version on it does that by itself.

### Added

- **Request logging.** Failed requests are recorded with the path, status and
  timing, so a browser that asked and got an error can be told apart from one
  that never asked at all.
- The live preview shows the actual error on screen instead of only a red badge.

## 26.09.06

### Fixed

- **The panel was blocked from rendering inside Home Assistant.** The app
  refused to be embedded in any iframe, and an app panel is exactly that. It now
  allows Home Assistant to embed it, while still refusing any other site.

### Added

- **A `diagnose` command** for when the camera is unreachable from the app but
  fine from everywhere else. It reports the container's own network, whether the
  camera's address collides with it, the state of the archive, and the result of
  a real snapshot request.

  Docker hands its networks addresses from the same private range many home
  networks use. If your camera's address lands inside one of those, containers
  treat it as being on their own bridge and never send the traffic to your LAN —
  so it works from the Home Assistant host and times out from inside the app.
  This is now reported on the Status tab and in the log, with the fix.

## 26.09.05

### Fixed

- **Live preview always said "camera unreachable"**, whatever the camera was
  doing. A naming collision introduced with ingress support meant the request
  was never actually made. The server had been answering correctly all along,
  which is why nothing showed in the log.
- **A missing image showed a broken image icon.** The Live tab now explains what
  is wrong instead: whether the archive directory is missing, cannot be read, or
  simply has no images yet.

### Added

- **The archive is diagnosed instead of shown as a dash.** Its state is checked
  at startup and on every check, and reported with the path and what to look at,
  in the log, on the Live tab and on the Status tab. An empty panel used to be
  indistinguishable from a wrong path.
- **Errors in the panel are written to the app log.** A browser console is no
  use to someone reading add-on logs, so the page now reports its own failures
  back to the app.

### Note

A wrong archive path is reported loudly but does **not** make the app unhealthy,
so Home Assistant will not restart it over a configuration mistake.

**Where do the images come from?** This app reads an archive that something else
writes; it does not capture or move files unless you turn that on. Mount your
share under Settings → System → Storage and point `archive_path` at it. The Live
tab works from the camera directly and needs no archive at all. See the
Documentation tab.

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
