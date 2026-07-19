# LoraTap Curtain Relay — ESPHome (CB2S / BK7231N)

ESPHome firmware for curtain relay modules based on **CB2S (Beken BK7231N)**, such as those found in **LoraTap SC411WSC** or similar devices.

> This project is designed to be **universal and modular**: the same base firmware works for different installations; you only need to pick the physical button configuration file that matches your wall switch.

## Features

- `time_based` cover with **estimated position** (0–100 %) and memory of the last state.
- **Adjustable up/down travel times** from Home Assistant / web UI.
- **Direction inversion** configurable from HA.
- **60 s safety timeout** to turn off relays if something goes wrong.
- **Built-in web server** for local control without relying on HA.
- Support for several wall switch types via included files:
  - 1 button (cycle)
  - 2 buttons (up/down)
  - 3 buttons (up/down/stop)
  - 3-position latching switch
  - No physical buttons

## Confirmed hardware

| Module | CB2S (BK7231N) |
|--------|----------------|
| Example MAC | `38:a5:c9:f0:44:e3` |
| Up relay | **P24** |
| Down relay | **P26** |
| Up button (candidate) | P23 |
| Down button (candidate) | P21 |
| Stop button (candidate) | P7 |

> Button pins may vary depending on the exact model. Pick the right mode and adjust the `substitutions` if needed.

## File structure

```text
.
├── curtain_relay_f044e3.yaml   # Base firmware
├── buttons_1way.yaml           # 1-button cycle
├── buttons_2way.yaml           # 2 buttons
├── buttons_3way.yaml           # 3 buttons
├── buttons_latching.yaml       # 3-position latching switch
├── buttons_disabled.yaml       # No physical buttons
├── secrets.yaml.example        # Secrets template
└── README.md
```

## Getting started

1. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your WiFi credentials.
2. Open `curtain_relay_f044e3.yaml` and choose the button type by editing the last line:

```yaml
packages:
  buttons: !include buttons_2way.yaml
```

3. Compile and flash via UART:

```bash
esphome compile curtain_relay_f044e3.yaml
ltchiptool flash write -d /dev/ttyUSB0 \
  /tmp/.esphome/build/curtain-relay-f044e3/.pioenvs/curtain-relay-f044e3/firmware.uf2
```

4. The CB2S module enters bootloader mode automatically; no need to bridge CEN to GND.
5. Verify it responds:

```bash
ping curtain-relay-f044e3.local
```

## UART wiring

| CB2S | USB-TTL adapter |
|------|-----------------|
| 3V3  | 3V3 |
| GND  | GND |
| TX   | RX |
| RX   | TX |

## Customization

Edit the `substitutions` in the base YAML to change pins or timeouts:

```yaml
substitutions:
  up_pin: "P24"
  down_pin: "P26"
  btn_up_pin: "P23"
  btn_down_pin: "P21"
  btn_stop_pin: "P7"
  safety_timeout: "60s"
```

## Calibration

Adjust the **Up time (s)** and **Down time (s)** numbers to match your curtain's real travel time. With correct values the position percentage will be accurate.

## References

- [ESPHome LibreTiny / bk72xx](https://esphome.io/components/libretiny.html)
- [LoraTap SC411WSC on devices.esphome.io](https://devices.esphome.io/devices/LoraTap-SC411WSC)
- [LoraTap SC500W on devices.esphome.io](https://devices.esphome.io/devices/LoraTap-SC500W)

## License

MIT — use, modify and share.
