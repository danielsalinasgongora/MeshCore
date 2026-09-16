# Firmware Observer MeshCore para Chile

Este fork prepara el firmware Observer MQTT de MeshCore para levantar un nodo observer/repeater en Chile en hardware Heltec V4. La documentación general de MeshCore, la arquitectura base y el uso avanzado del firmware original están en el repositorio upstream: [agessaman/MeshCore](https://github.com/agessaman/MeshCore).

Este README se enfoca solo en lo necesario para flashear, configurar y verificar un observer chileno con este firmware. **Nota: los binarios publicados por este fork son solo para HELTEC V4. No los uses en otro hardware.**

## Qué deja listo este firmware

- Radio Chile: `927.875 MHz`, `62.5 kHz`, `SF8`, `CR5`.
- Potencia TX: `22 dBm`.
- `path.hash.mode 1`, que en MeshCore equivale a advert/path hash de 2 bytes.
- MQTT preconfigurado para:
  - MeshChile: `mqtt-msc.meshchile.cl`.
  - LetsMesh Analyzer EU.
  - MeshMapper.
- Certificado correcto para el broker actual de MeshChile, usando Let’s Encrypt / ISRG Root X1.
- Zona horaria `America/Santiago`.
- NTP primario `ntp.shoa.cl`.
- Panel web automático cuando el nodo conecta a WiFi.
- Pantalla OLED con estado útil del nodo: nombre, radio, TX, hash e IP/WiFi.
- Bot opcional para responder `#ping`, `#status`, `#wifi` y `#hora` en Public o en un canal privado.

El firmware público no incluye tus claves, WiFi, coordenadas privadas ni llaves de canales. Cada operador debe configurar esos datos después del flash.

## Hardware recomendado

Esta guía está pensada solo para:

- Heltec V4 / ESP32-S3 con LoRa.
- Cable USB-C de datos.
- WiFi con salida a internet.
- Antena adecuada para la banda usada por MeshCore Chile.

## Opción rápida: flashear con PlatformIO

Instala PlatformIO y conecta el Heltec V4 por USB.

Compila el firmware:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile
```

Flashea el equipo:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile -t upload
```

Si necesitas indicar el puerto manualmente en Windows:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile -t upload --upload-port COM3
```

Después del flash, abre una consola serial a `115200` baudios.

## Configuración inicial paso a paso

Con el nodo recién flasheado, configura estos valores por serial. Cambia los campos entre `<...>` por tus datos.

```text
set name <nombre-del-nodo>
set password <clave-admin-segura>
set wifi.ssid <nombre-wifi>
set wifi.pwd <clave-wifi>
set radio 927.875,62.5,8,5
set tx 22
set path.hash.mode 1
set advert.interval 60
set flood.advert.interval 12
set lat <latitud>
set lon <longitud>
set mqtt.iata <codigo-zona>
set mqtt1.preset meshchile
set mqtt2.preset analyzer-eu
set mqtt2.filter all
set mqtt3.preset meshmapper
set mqtt.rx on
set mqtt.packets on
set mqtt.status on
set mqtt.tx on
set mqtt.ntp ntp.shoa.cl
set timezone America/Santiago
reboot
```

Notas rápidas:

- Usa `SCL`, `VAP` u otro código acordado por tu zona/comunidad en `mqtt.iata`.
- `path.hash.mode 1` significa 2-byte en MeshCore.
- `mqtt.tx on` permite que los mensajes generados por el nodo, como respuestas del bot, también suban a mapas MQTT. Si queda en `advert`, solo suben los adverts propios.
- No publiques coordenadas exactas si el nodo está en una ubicación privada.

## Configurar desde el panel web

Cuando el WiFi conecta, el panel web se levanta automáticamente en la IP del nodo. La OLED debería mostrar esa IP.

También puedes iniciarlo manualmente por serial:

```text
start webconfig
```

O forzar un punto de acceso de configuración:

```text
start webconfig ap
```

Entra desde el navegador con la contraseña admin del nodo. Desde la web puedes ajustar WiFi, radio, ubicación, MQTT, bot y otros parámetros sin repetir todo por serial.

## Activar presencia en mapas

Después de configurar y reiniciar, fuerza un advert:

```text
advert
advert flood
```

El nodo debería reportar a:

- [Mapa MeshChile](https://mapa-msc.meshchile.cl/#nodos)
- [Mensajes MeshChile](https://mapa-msc.meshchile.cl/#mensajes)
- [LetsMesh Analyzer](https://analyzer.letsmesh.net/)
- [MeshMapper](https://vap.meshmapper.net/)

La aparición puede tardar unos minutos según el mapa, los filtros activos y el último advert recibido.

## Verificar que todo quedó funcionando

Estos comandos son los más útiles después del reboot:

```text
get wifi.status
get mqtt.status
get mqtt.tx
get mqtt.ntp
get mqtt.ntp.diag
get path.hash.mode
get tx
get public.key
clock
```

Estado esperado para MQTT:

```text
msgs: on, 1: meshchile (ok), 2: analyzer-eu (ok), 3: meshmapper (ok), q:0
```

Estado recomendado para que los mensajes del bot aparezcan en mapas:

```text
get mqtt.tx
> on
```

## Bot de comandos LoRa

El bot viene apagado por defecto. Se puede activar desde el panel web en la sección MQTT → LoRa command bot, o por serial.

Para responder en el canal Public:

```text
set bot on
set bot.channel public
```

Comandos disponibles desde el canal elegido:

```text
#ping
#status
#wifi
#hora
```

Ejemplos de respuesta:

```text
🏓 pong | SNR -1.50 dB | RSSI -114 dBm | uptime 35min | 2 hops | QTA_OBSERVER
📡 status | uptime 35min | RX 120 | TX 8 | Hash:2-byte | QTA_OBSERVER
📶 wifi | IP 192.168.1.50 | RSSI -41 dBm | QTA_OBSERVER
🕒 hora UTC 2026-09-15 18:30:00 | QTA_OBSERVER
```

El bot responde como máximo una vez por minuto para no saturar LoRa. Si mandas varios comandos seguidos, espera 60 segundos entre pruebas.

Para usar un canal privado del bot:

```text
set bot on
set bot.channel private
set bot.hashtag <#canal-privado>
```

O con una PSK explícita de 16 bytes en hexadecimal:

```text
set bot on
set bot.channel private
set bot.psk <clave-hex-de-32-caracteres>
```

No publiques PSK privadas ni nombres de canales privados si identifican tu instalación local.

## Canal de alertas del observer

El canal de alertas es independiente del bot. Sirve para que el observer envíe avisos operativos o pruebas.

Configura un canal privado con hashtag:

```text
set alert.hashtag <#canal-privado>
set alert on
alert test
```

O con PSK explícita:

```text
set alert.psk <clave-hex-de-32-caracteres>
set alert on
alert test
```

Por seguridad, el firmware rechaza Public como canal de alertas para evitar spam accidental. Public sí puede usarse para el bot si lo habilitas explícitamente.

## Diagnóstico rápido

Si no apareces en MeshChile:

1. Revisa WiFi:

   ```text
   get wifi.status
   ```

2. Revisa MQTT:

   ```text
   get mqtt.status
   ```

   MeshChile debe salir como `ok`.

3. Revisa hora/NTP:

   ```text
   get mqtt.ntp.diag
   clock
   ```

4. Revisa radio y hash:

   ```text
   get radio
   get path.hash.mode
   ```

5. Fuerza advert:

   ```text
   advert
   advert flood
   ```

Si el bot responde por LoRa pero no ves la respuesta en el mapa de mensajes, revisa:

```text
get mqtt.tx
```

Debe estar en `on`.


## Documentación adicional

- Firmware base original: [agessaman/MeshCore](https://github.com/agessaman/MeshCore)
- Guía específica del perfil Chile: [docs/meshchile-observer.md](./docs/meshchile-observer.md)
- Documentación MeshCore: [docs.meshcore.io](https://docs.meshcore.io)
- Flasher oficial MeshCore: [meshcore.io/flasher](https://meshcore.io/flasher)
- MeshChile: [meshchile.cl](https://meshchile.cl/)

## Licencia

Este fork mantiene la licencia MIT del proyecto MeshCore original. Revisa el repositorio upstream para los detalles completos de licencia y atribución.

