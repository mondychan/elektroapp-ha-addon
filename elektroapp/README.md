# Elektroapp Home Assistant Add-on

This add-on exposes Elektroapp through Home Assistant Ingress and reads its
configuration from the add-on options (Supervisor).

## Installation (overview)

1. Add your GitHub add-on repository in Home Assistant (Settings > Add-ons).
2. Install the "Elektroapp" add-on.
3. Open the add-on configuration and fill in the InfluxDB connection and tariff
   settings.
4. Start the add-on and open it via the Ingress panel.

## Notes

- The app reads add-on options from `/data/options.json`.
- The `tarif.vt_periods` option accepts a string like `6-7,9-10,13-14,16-17`.
