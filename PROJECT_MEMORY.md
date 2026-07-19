# Memory del proyecto — Relé de persianas CB2S

## Reglas de configuración ESPHome aplicables

- Web server **SIEMPRE habilitado** en versiones estables (`web_server:`).
- **Sin contraseña** en portal cautivo ni web server.
- **Sin `api.encryption.key`**.
- **DHCP siempre**; reservar IP en el router si hace falta.
- Nombre único por dispositivo basado en los últimos 6 caracteres de la MAC.

## Lecciones aprendidas

1. **CB2S entra en bootloader automáticamente** al flashear por UART; no hace falta CEN→GND ni power cycle.
2. **Las OTA con CB2S son inestables**. Una vez perdida la conexión, reflashear por UART es más fiable.
3. **NO usar variables de substitución dentro de `!include`**. `packages: !include ${button_config}` compila pero el dispositivo no arranca. Usar `!include buttons_2way.yaml` directamente.
4. **Pines de relés**: P24 = subida, P26 = bajada (confirmado con pruebas).
5. **Estructura modular comunitaria**: YAML base + archivos de botones según tipo de interruptor.
6. **Mando RF**: se perdió al flashear. Recuperarlo requiere análisis del firmware stock o reprogramación del receptor.
7. **Backup stock** guardado en `/tmp/relay_backup.bin` (2 MiB). Particiones extraídas en `/tmp/relay_split/` y `/tmp/relay_extracted/`.

## Estructura del repo

```text
.
├── curtain_relay_f044e3.yaml   # Firmware base
├── buttons_1way.yaml           # 1 pulsador (ciclo)
├── buttons_2way.yaml           # 2 pulsadores
├── buttons_3way.yaml           # 3 pulsadores
├── buttons_latching.yaml       # Interruptor latching 3 posiciones
├── buttons_disabled.yaml       # Sin botones físicos
├── secrets.yaml.example        # Plantilla de secretos
├── README.md                   # Documentación en español
└── README.en.md                # Documentación en inglés
```

## Próximos pasos

1. Verificar estabilidad del firmware modular actual.
2. Calibrar `open_duration` y `close_duration` con una persiana real.
3. Identificar pines reales de los botones físicos de pared (P23, P21, P7 son candidatos).
4. Añadir al repo fotos del pinout y del interruptor de pared usado.
5. Investigar recuperación del mando RF.
6. Subir a GitHub y preparar PR para devices.esphome.io cuando el pinout esté completo.

## Comandos útiles

```bash
# Compilar
esphome compile curtain_relay_f044e3.yaml

# Flashear
ltchiptool flash write -d /dev/ttyUSB0 \
  /tmp/.esphome/build/curtain-relay-f044e3/.pioenvs/curtain-relay-f044e3/firmware.uf2

# Logs
esphome logs --device 192.168.1.37 curtain_relay_f044e3.yaml

# Backup de flash original
ltchiptool flash read -d /dev/ttyUSB0 -o relay_backup.bin
```

## Historial de versiones

- `curtain_relay_f044e3.yaml` + `buttons_*.yaml`: firmware modular actual (estilo comunidad).
- `curtain_relay_f044e3_balanced.yaml`: versión con cover template, tiempos ajustables, inversión, timeout 60 s.
- `curtain_relay_f044e3_min.yaml`: firmware mínimo de recuperación.
- `curtain_relay_f044e3_diag.yaml`: diagnóstico con switches de prueba.
- `curtain_relay_f044e3_final.yaml`: versión avanzada con ejemplo comentado de botones.
- `curtain_relay_f044e3_universal.yaml`: intento fallido de configuración en runtime con `select`.
