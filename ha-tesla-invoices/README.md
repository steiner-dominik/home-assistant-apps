# ⚡ Home Assistant App: Tesla Invoices

**Automatically download all your Tesla charging & subscription invoices — and actually understand them.**

- 📥 Fetches every Supercharging and subscription invoice from your Tesla account
- 📊 Analytics dashboard via ingress: monthly kWh/cost charts, price per kWh,
  filtering, search, built-in PDF viewer
- 📤 CSV export for expense reports; optional automatic email export
- 🚗 Multi-vehicle and multi-currency aware
- 🔑 Only a refresh token is needed — access tokens are handled automatically

See [DOCS.md](DOCS.md) for installation and all configuration options.

This app runs the prebuilt image from
[steiner-dominik/tesla-invoices](https://github.com/steiner-dominik/tesla-invoices),
where the application is developed — it can also run standalone via Docker on
any server.
