# Memory del proyecto — Relé de persianas CB2S

## Reglas de configuración ESPHome aplicables

- Web server **SIEMPRE habilitado** en versiones estables (`web_server:`).
- **Sin contraseña** en portal cautivo ni web server.
- **Sin `api.encryption.key`**.
- **DHCP siempre**; reservar IP en el router si hace falta.
- Nombre único por dispositivo basado en los últimos 6 caracteres de la MAC.
- **`captive_portal` SIEMPRE activado** en versiones estables para recuperación sin cablear.

## Lecciones aprendidas

1. **CB2S entra en bootloader automáticamente** al flashear por UART; no hace falta CEN→GND ni power cycle (en la PCB original del relé).
2. **Las OTA con CB2S son inestables**. Una vez perdida la conexión, reflashear por UART es más fiable.
3. **NO usar variables de substitución dentro de `!include`**. `packages: !include ${button_config}` compila pero el dispositivo no arranca. Usar `!include buttons_2way.yaml` directamente.
4. **Pines de relés**: P24 = subida, P26 = bajada (confirmado con pruebas y con YAML de jojo99 para SC500W-CB2S).
5. **Pines de botones físicos**: P23 = subir, P21 = bajar, P7 = stop (candidato; en nuestro hardware P23/P21 confirmados).
6. **LED onboard**: P10 (según YAML de jojo99).
7. **Mando RF**: el relé usa un **módulo receptor RF independiente de 868 MHz**, no 433 MHz. El CB2S no lo lee directamente; el receptor controla los relés por sí solo. Por eso `remote_receiver` de ESPHome no detecta nada en ningún GPIO (probados P6, P7, P8, P10, P21, P23, P24, P26 con `dump: all` y `dump: raw`).
8. **Escaneo masivo de pines provoca bucles de reinicio** en CB2S. No usar `binary_sensor` en muchos pines a la vez ni `remote_receiver` con `dump: raw` de forma prolongada.
9. **Cortocircuitar P26 a GND quema el CB2S y el adaptador USB-TTL**. Confirmado: el primer CB2S y el adaptador CP2102 murieron así.
10. **El `secrets.yaml` con credenciales dummy rompe la conectividad tras OTA**. Siempre usar credenciales reales antes de flashear.
11. **Recuperación sin cablear**: si el relé tiene `captive_portal` con AP sin contraseña, se puede acceder a `192.168.4.1` y reconfigurar WiFi.
12. **El componente `cover` de ESPHome causa problemas de interlock tras un stop** en este hardware. Solución: usar **scripts + switches internos** (estilo jojo99) en vez de `cover`.

## Hardware confirmado (LoraTap SC500W / similar con CB2S)

| Función | Pin CB2S | Notas |
|---|---|---|
| Relé subida | P24 | Confirmado |
| Relé bajada | P26 | Confirmado |
| Botón subir | P23 | Confirmado en nuestro hardware |
| Botón bajar | P21 | Confirmado en nuestro hardware |
| Botón stop | P7 | Candidato |
| LED onboard | P10 | Según jojo99 |
| Receptor RF | 868 MHz, módulo independiente | No va al CB2S |

## Repo remoto

- **GitHub**: https://github.com/gnacho/esphome-loratap-curtain-cb2s
- Rama `main` sincronizada con el repo local.

## Estructura del repo

```text
.
├── curtain_relay_f044e3.yaml         # Firmware base con cover (histórico)
├── curtain_relay_f044e3_full.yaml    # Firmware estable con scripts (definitivo)
├── buttons_1way.yaml                 # 1 pulsador (ciclo)
├── buttons_2way.yaml                 # 2 pulsadores
├── buttons_3way.yaml                 # 3 pulsadores
├── buttons_latching.yaml             # Interruptor latching 3 posiciones
├── buttons_disabled.yaml             # Sin botones físicos
├── secrets.yaml.example              # Plantilla de secretos
├── README.md                         # Documentación en inglés
├── README.es.md                      # Documentación en castellano
└── PROJECT_MEMORY.md                 # Este archivo
```

## Próximos pasos

1. Verificar estabilidad del firmware `curtain_relay_f044e3_full.yaml` en uso diario.
2. Calibrar `Tiempo subida (s)` y `Tiempo bajada (s)` con la persiana real.
3. Añadir al repo fotos del pinout y del interruptor de pared usado.
4. **Mando RF**: documentar que el receptor de 868 MHz es independiente y no se puede leer con ESPHome. Si se quiere RF con ESPHome, hay que añadir un receptor de 433 MHz externo.
5. Preparar PR para devices.esphome.io cuando el pinout esté completo.

## Comandos útiles

```bash
# Compilar
esphome compile curtain_relay_f044e3_full.yaml

# Flashear por UART
ltchiptool flash write -d /dev/ttyUSB0 \
  /home/nacho/Descargas/curtain-relay-cb2s/.esphome/build/curtain-relay-f044e3/.pioenvs/curtain-relay-f044e3/firmware.uf2

# Flashear por OTA
esphome upload --device curtain-relay-f044e3.local curtain_relay_f044e3_full.yaml

# Logs
esphome logs --device curtain-relay-f044e3.local curtain_relay_f044e3_full.yaml

# Backup de flash original
ltchiptool flash read bk7231n /tmp/relay_backup.bin -d /dev/ttyUSB0
```

## Historial de versiones

- `curtain_relay_f044e3_full.yaml` (22-Jul-2026): firmware definitivo con scripts, sin cover, estable.
- `curtain_relay_f044e3.yaml` + `buttons_*.yaml`: firmware modular original con cover (tenía problemas de interlock).
- `curtain_relay_f044e3_jojo99.yaml`: adaptación directa del YAML de jojo99 (sin exponer controles a HA).
- `curtain_relay_f044e3_balanced.yaml`: versión con cover template, tiempos ajustables, inversión, timeout 60 s.
- `curtain_relay_f044e3_min.yaml`: firmware mínimo de recuperación.
- `curtain_relay_f044e3_diag.yaml`: diagnóstico con switches de prueba.
- `curtain_relay_f044e3_final.yaml`: versión avanzada con ejemplo comentado de botones.
- `curtain_relay_f044e3_universal.yaml`: intento fallido de configuración en runtime con `select`.
