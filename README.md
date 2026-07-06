# Home Assistant Apps

[![Open app repo on your Home Assistant instance][repo-badge]][repo-link]

A [Home Assistant app repository](https://www.home-assistant.io/common-tasks/os#installing-a-third-party-app-repository)
maintained by Dominik Steiner.

## 🏠 Installation

1. Click the badge above, **or** add the repository manually:
   **Settings → Apps → App Store → ⋮ → Repositories** and enter

   ```text
   https://github.com/steiner-dominik/home-assistant-apps
   ```

2. Install an app from the store and follow its documentation tab.

## 📦 Apps

| App | Description |
| ------ | ----------- |
| [**⚡ Tesla Invoices**](ha-tesla-invoices/) | Downloads your Tesla charging & subscription invoices, with an analytics dashboard (monthly kWh/cost charts, price per kWh, CSV export) and optional email export. |

## 🧩 How this repository works

This repository only contains the app **manifests and documentation**. The
application itself lives in
[steiner-dominik/tesla-invoices](https://github.com/steiner-dominik/tesla-invoices),
which publishes a prebuilt multi-arch Docker image to GHCR. The Supervisor
pulls that image directly (`image:` key in `config.yaml`) — installs are fast
and nothing is compiled on your Home Assistant machine.

Found a bug or have a feature request?

- **App behavior, dashboard, downloads** → [tesla-invoices issues](https://github.com/steiner-dominik/tesla-invoices/issues)
- **Installation, app options, ingress** → [this repository's issues](https://github.com/steiner-dominik/home-assistant-apps/issues)

## ⚖️ Disclaimer

This software is provided “as is” and without any warranty. Use at your own
risk. The apps here are not affiliated with or endorsed by any third party
whose services they interact with.

[repo-badge]: https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg
[repo-link]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsteiner-dominik%2Fhome-assistant-apps
