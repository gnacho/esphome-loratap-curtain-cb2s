# LoraTap Curtain Relay — ESPHome (CB2S / BK7231N)

[🇬🇧 Read the English version](README.md)

Firmware de ESPHome para relés de persianas basados en el módulo **CB2S (Beken BK7231N)**, como los encontrados en dispositivos **LoraTap SC411WSC**, **LoraTap SC500W-CB2S** o similares.

> Este proyecto está pensado para ser **universal y modular**: el mismo firmware base sirve para distintas ubicaciones; solo tienes que elegir el archivo de botones físicos que corresponda a tu interruptor de pared.

## Características

- **Control estable con scripts** (sin problemas de interlock del `cover`).
- **Posición estimada** (0–100 %) y memoria del último estado.
- **Tiempos de subida/bajada ajustables** desde Home Assistant / web.
- **Inversión de sentido** configurable desde HA.
- **Timeout de seguridad** de 60 s para apagar los relés si algo falla.
- **Web server** integrado para control local sin depender de HA.
- **Portal cautivo** para recuperación fácil si falla la WiFi.
- Soporte para varios tipos de interruptor de pared vía archivos incluidos:
  - 1 pulsador (ciclo)
  - 2 pulsadores (subir/bajar)
  - 3 pulsadores (subir/bajar/parar)
  - Interruptor latching de 3 posiciones
  - Sin botones físicos

## Hardware confirmado

| Módulo | CB2S (BK7231N) |
|--------|----------------|
| MAC de ejemplo | `38:a5:c9:f0:44:e3` |
| Relé de subida | **P24** |
| Relé de bajada | **P26** |
| Botón subir | P23 |
| Botón bajar | P21 |
| Botón parar | P7 (candidato) |
| LED integrado | P10 |

> Los pines de botones pueden variar según el modelo exacto. Usa el modo que corresponda y ajusta las `substitutions` si es necesario.

## Nota importante sobre el mando RF

El mando RF original funciona a **868 MHz** y lo gestiona un **módulo receptor independiente** dentro del relé. **No pasa por el CB2S**, así que **no se puede leer con `remote_receiver` de ESPHome**. Si el mando deja de funcionar tras flashear, es porque el receptor está dañado o desemparejado, no por el software.

## Estructura de archivos

```text
.
├── curtain_relay_f044e3_full.yaml    # Firmware estable recomendado
├── curtain_relay_f044e3.yaml         # Firmware base original (con cover)
├── buttons_1way.yaml                 # 1 pulsador (ciclo)
├── buttons_2way.yaml                 # 2 pulsadores
├── buttons_3way.yaml                 # 3 pulsadores
├── buttons_latching.yaml             # Interruptor latching 3 posiciones
├── buttons_disabled.yaml             # Sin botones físicos
├── secrets.yaml.example              # Plantilla de secretos
├── PROJECT_MEMORY.md                 # Lecciones y notas del proyecto
├── README.md                         # Versión en inglés
└── README.es.md                      # Este archivo
```

## Cómo empezar

1. Copia `secrets.yaml.example` a `secrets.yaml` y rellena tus credenciales WiFi.
2. **Recomendado:** usa `curtain_relay_f044e3_full.yaml` como firmware.
3. Compila y flashea por UART:

```bash
esphome compile curtain_relay_f044e3_full.yaml
ltchiptool flash write -d /dev/ttyUSB0 \
  .esphome/build/curtain-relay-f044e3/.pioenvs/curtain-relay-f044e3/firmware.uf2
```

4. El módulo CB2S entra en modo bootloader automáticamente; no es necesario CEN a GND en la PCB original.
5. Verifica que responde:

```bash
ping curtain-relay-f044e3.local
```

## Conexión UART

| CB2S | Adaptador USB-TTL |
|------|-------------------|
| 3V3  | 3V3 (solo si el relé no tiene fuente propia) |
| GND  | GND |
| TX   | RX |
| RX   | TX |

> **Aviso:** nunca cortocircuites P26 a GND. Quemarás el CB2S y posiblemente el adaptador USB-TTL.

## Personalización

Edita las `substitutions` del YAML para cambiar pines o tiempos:

```yaml
substitutions:
  up_pin: "P24"
  down_pin: "P26"
  btn_up_pin: "P23"
  btn_down_pin: "P21"
  btn_stop_pin: "P7"
  safety_timeout: "60s"
```

## Calibración

Ajusta los números **Tiempo subida (s)** y **Tiempo bajada (s)** para que coincidan con el tiempo real de tu persiana. Con esos valores el porcentaje de posición será preciso.

## Referencias

- [ESPHome LibreTiny / bk72xx](https://esphome.io/components/libretiny.html)
- [LoraTap SC411WSC en devices.esphome.io](https://devices.esphome.io/devices/LoraTap-SC411WSC)
- [LoraTap SC500W en devices.esphome.io](https://devices.esphome.io/devices/LoraTap-SC500W)
- [YAML de jojo99 para SC500W-CB2S](https://community.home-assistant.io/t/loratap-sc500w-v1-shutter-switch-with-esphome/616375)

## Licencia

AGPL-3.0 — úsalo, modifícalo y compártelo bajo las mismas condiciones.
