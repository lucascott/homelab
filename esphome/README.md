# ESPHome configurations

Based on the [ESPHome project](https://esphome.io)

## Quickstart

The execution of the ESPHome configuration requires the `esphome` CLI (install it locally with `pipx install esphome`).

1. Run `mv secrets.example.yaml secrets.yaml` and modify the secret variables' values in the file with the desired configuration.
2. Run `esphome run esp32_bedroom.yaml` with the ESP32 connected via USB or, if preconfigured with over-the-air updates, connected to the same wifi.

