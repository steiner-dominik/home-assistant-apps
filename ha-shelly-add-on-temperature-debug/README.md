# 🌡 Home Assistant App: Shelly Add-on Temperature Debug

**A safe, instant troubleshooting view of the DS18B20 temperature and DHT22
humidity sensors attached to your Shelly Sensor Add-ons.**

> ⚠️ **This is an independent community project. It is not affiliated with,
> endorsed by, or supported by Shelly Group / Allterco Robotics in any way.**

- 🩺 One click shows every sensor on every configured Shelly: value, status
  (OK · 85 °C reset · no reading · missing · unreachable) and plain-language
  guidance on what to check
- 📈 In-memory history charts make intermittent wiring problems visible;
  wiggle test polls every 2 s while you re-seat cables
- 🕙 Background polling keeps history filling even while nobody watches
- 📊 CSV export, optional Prometheus metrics, English/German, dark/light
- 🔒 The Shelly admin password stays on the server — viewers never see it

See [DOCS.md](DOCS.md) for installation and all configuration options.

This app runs the prebuilt image from
[steiner-dominik/shelly-add-on-temperature-debug](https://github.com/steiner-dominik/shelly-add-on-temperature-debug),
where the application is developed — it can also run standalone via Docker on
any server.

❤️ If this app is useful to you, you can support its development via
[GitHub Sponsors](https://github.com/sponsors/steiner-dominik),
[Ko-fi](https://ko-fi.com/dominik_steiner), or
[Buy Me a Coffee](https://buymeacoffee.com/dominik.st).
