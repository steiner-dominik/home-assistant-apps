# UniFi Protect Timelapse

Timelapse capture and archive watchdog for a **UniFi Protect** camera.

By default this app captures nothing. It watches: it probes the camera to prove
it is reachable, and checks that new images keep appearing in your archive. That
makes it safe to run alongside whatever is already capturing — it will never add
frames or disturb the interval.

Full documentation:
<https://github.com/steiner-dominik/unifi-protect-timelapse>

## Where do the images come from?

This app does not capture and does not move files. It **reads an archive that
something else writes** — your Raspberry Pi today, or the standalone Docker
deployment once you switch over — and shows you the live camera alongside it.

So there are two independent things on this page:

- **Live** always works from the camera directly. It needs only the camera
  settings, not the archive.
- **Timelapse** and the archive health checks read `archive_path`. If nothing
  fills that directory, there is nothing to show, and the panel will say so.

That is why `capture_enabled` and `sync_mode` are off by default: running a
second capturer against the same archive would give you uneven spacing. If you
want *this* app to be the thing that captures and moves files, see
[Capturing from Home Assistant](#capturing-from-home-assistant).

## Setup

1. **Mount your archive.** Go to **Settings → System → Storage → Add network
   storage** and mount the NFS share your images live on. Pick usage type
   **Media** and give it a name; Home Assistant then makes it available to this
   app at `/media/<name>`. (Usage type *Share* appears at `/share/<name>`.)

   Without this step the app has no archive at all: the Live tab still works,
   but the Timelapse tab is empty and the panel reports the directory as
   missing.
2. Set `archive_path` to that location. If you named the mount `timelapse`,
   that is `/media/timelapse`. The app expects the images in the usual
   `YYYY/YYYY-MM/YYYY-MM-DD/` layout underneath it, so point it at the
   directory that *contains* the year folders.

   Check it worked: the **Status** tab shows *Archive: readable*, and the app
   log says `archive is readable`. If the path is wrong you get an explicit
   error naming it, both in the log and on the Live tab.
3. Point the app at your camera:
   - `camera_source: protect` — set `protect_host`, `protect_api_key` and
     `protect_camera_id`. Create the key under **Settings → Control Plane →
     Integrations** on the UniFi OS console.
   - `camera_source: snapshot` — set `camera_snapshot_url`, and enable
     *Anonymous Snapshot* for the camera in UniFi Protect.
   - Optionally set `camera_fallback_source` to the other one.
4. Set `archive_max_age` comfortably above the interval your images are actually
   captured at, so a single late image does not raise an alarm.
5. Open the panel from the sidebar. Home Assistant handles authentication.

## What it checks

| Check | What it catches |
|---|---|
| Camera probe | The camera does not answer at all |
| Frozen frames | The camera answers, with the same bytes, forever |
| Archive freshness | Images are not landing in the archive any more |
| Gap detection | Stretches of a past day where images are missing |

The frozen-frame check exists because a camera in that state looks perfectly
healthy by every other measure: it responds, the response is a valid JPEG, and
files keep being written. Only comparing consecutive frames reveals it.

The app is reported unhealthy — and the Home Assistant watchdog will restart
it — when the camera is unreachable or the archive has gone stale, but only
inside the configured `active_hours`.

## Entities

The app writes these straight to the Core API using the Supervisor token. No
MQTT broker and no template sensors are involved. `entity_prefix` renames them.

| Entity | Meaning |
|---|---|
| `binary_sensor.timelapse_camera_online` | The camera answered the last probe |
| `binary_sensor.timelapse_archive_available` | The archive's sentinel file is present |
| `binary_sensor.timelapse_frame_frozen` | Identical frames are being returned |
| `sensor.timelapse_archive_newest` | Timestamp of the newest archived image |
| `sensor.timelapse_spool_files` | Images currently buffered locally |
| `sensor.timelapse_status` | `ok`, `failing`, `camera_offline`, `frozen` or `buffering` |

An automation that tells you when the timelapse quietly stops:

```yaml
automation:
  - alias: Timelapse stopped
    triggers:
      - trigger: state
        entity_id: sensor.timelapse_status
        to: camera_offline
        for: "00:15:00"
    actions:
      - action: notify.persistent_notification
        data:
          message: "The timelapse camera has been unreachable for 15 minutes."
```

## The panel

- **Live** — the newest archived image, plus a *live preview* button that fetches
  a fresh frame and **never writes it to disk**, so the archive keeps exactly one
  image per interval.
- **Timelapse** — browse by year, month and day, play a day back with a scrubber,
  see where the gaps are, and export the day as MP4 or ZIP. Days that contain
  images from several capture series can be filtered by series, so playback does
  not jump between camera framings.
- **Status** — camera reachability, archive freshness, buffered images, and a
  read-only view of the running configuration. Secrets are never shown; the API
  key appears only as a yes/no.

## Capturing from Home Assistant

Set `capture_enabled: true` if you want this app to take the images rather than
a separate deployment. Then also set `sync_mode` to `opportunistic` so the
images are moved to `archive_path`, and check `capture_interval`,
`active_hours` and `filename_prefix`.

The archive must contain a sentinel file named `.nas` before anything is
written to it:

```bash
touch /media/timelapse/.nas
```

This is not optional. An unmounted network share looks exactly like an empty
local directory, and without the check the app would fill the add-on's own
storage while deleting the originals. If the file is missing, the app refuses to
move anything and buffers locally instead.

Only run one capturer. Two of them writing to the same archive would produce
uneven spacing.

## Data

`/data` holds the app's state (last capture, camera and archive health) and, if
capturing is enabled, the local buffer. It is included in Home Assistant backups
automatically.

## Notes

- Ingress only: the app's port is not published, and Home Assistant
  authenticates every request. `auth_mode` exists for the case where you expose
  the port yourself.
- `protect_insecure_tls` disables certificate verification for the console
  connection. Enable it only for a self-signed console certificate.
- This is an independent community project, not affiliated with, endorsed by or
  sponsored by Ubiquiti Inc. or the Home Assistant project. UniFi and UniFi
  Protect are trademarks of Ubiquiti Inc.
