## Acerca de MeshCore

MeshCore es una librería C++ liviana y portable para crear redes mesh de paquetes sobre LoRa y otros radios. Permite que nodos pequeños se comuniquen a larga distancia y que otros nodos repitan los mensajes cuando no hay internet o infraestructura tradicional disponible.

## 🔍 ¿Qué es MeshCore?

MeshCore permite crear redes descentralizadas parecidas a Meshtastic o Reticulum, pero con foco en ruteo multi-salto liviano para proyectos embebidos. Es útil para comunicaciones fuera de red, emergencias, zonas rurales, pruebas de radio, sensores e instalaciones donde se necesita resiliencia con bajo consumo.

Este fork parte del firmware Observer de Gessaman y agrega un perfil listo para Chile, pensado para levantar nodos observadores/repetidores con Heltec V4 y reportar a mapas comunitarios.

> **Firmware Observer MQTT**: la base original de Gessaman incluye webconfig, MQTT con autenticación JWT, NTP, estadísticas, SNMP y presets de mapas. Este fork mantiene esa base y suma un perfil chileno listo para compilar.

## 🇨🇱 Perfil MeshChile / Chile Observer

Este fork incluye un perfil público para nodos MeshCore en Chile. Está pensado para que cualquier persona pueda levantar un observer/repeater sin tener que repetir toda la configuración manual de radio, MQTT, mapas y reloj.

Lo que agrega este fork:

- Preset MQTT `meshchile` para `wss://mqtt-msc.meshchile.cl:443/mqtt`, con autenticación JWT por identidad del dispositivo y audience `mqtt-msc.meshchile.cl`.
- Target PlatformIO `heltec_v4_repeater_observer_mqtt_chile` para Heltec V4.
- Radio Chile por defecto: `927.875 MHz`, `62.5 kHz`, `SF8`, `CR5`.
- Potencia TX por defecto: `22 dBm`.
- `path.hash.mode = 1`, que en MeshCore equivale a hashes de ruta de 2 bytes.
- Reporte MQTT por defecto hacia MeshChile, LetsMesh Analyzer EU y MeshMapper.
- Zona horaria `America/Santiago`.
- NTP primario `ntp.shoa.cl`.
- Pantalla OLED mejorada con nombre del nodo, radio, potencia TX, tamaño de hash e IP/WiFi.
- Panel web LAN automático: cuando el nodo conecta al WiFi, el webconfig queda disponible por la IP del nodo.
- Terminal web integrada para ejecutar comandos CLI desde el navegador.
- Soporte documentado para canal privado de alertas del observer.

El firmware público no incluye credenciales personales. Cada operador debe configurar su WiFi, contraseña de administración, llaves MeshCore y ubicación propia después de flashear.

## ⚙️ Compilar

Instala PlatformIO y compila el perfil Chile:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile
```

## 🔌 Flashear un Heltec V4

Conecta el Heltec V4 por USB y ejecuta:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile -t upload
```

Si necesitas indicar puerto manualmente:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile -t upload --upload-port COM3
```

## 🌐 Panel web

En el perfil Chile, el panel web se levanta automáticamente cuando el nodo está conectado al WiFi. En la pantalla OLED deberías ver la IP del nodo.

También puedes iniciarlo manualmente por serial:

```text
start webconfig
```

Para levantar un AP de configuración:

```text
start webconfig ap
```

El panel usa la contraseña de administración del nodo. Cámbiala siempre en cada instalación.

## 🧭 Configuración rápida por serial

Plantilla segura para un nodo observer/repeater chileno:

```text
set name <nombre-del-nodo>
password <clave-admin-segura>
set wifi.ssid <nombre-wifi>
set wifi.pwd <clave-wifi>
set radio 927.875,62.5,8,5
set tx 22
set path.hash.mode 1
set lat <latitud>
set lon <longitud>
set mqtt.iata SCL
set mqtt1.preset meshchile
set mqtt2.preset analyzer-eu
set mqtt2.filter all
set mqtt3.preset meshmapper
set mqtt.rx on
set mqtt.packets on
set mqtt.status on
set mqtt.tx advert
set mqtt.ntp ntp.shoa.cl
set timezone America/Santiago
advert
```

Notas:

- `path.hash.mode 1` significa 2-byte en MeshCore.
- Usa `SCL`, `VAP` u otro código según la zona/comunidad donde quieras reportar.
- No publiques coordenadas exactas si el nodo está en una ubicación privada.

## 🚨 Canal de alertas del observer

El firmware Observer puede enviar alertas LoRa por un canal configurado. Esto sirve para que el nodo mande avisos o pruebas desde la terminal web o serial.

Configura un canal privado así:

```text
set alert.hashtag <#canal-privado>
set alert on
alert test
```

Si tienes una clave de canal explícita de 16 bytes en hexadecimal, úsala así:

```text
set alert.psk <clave-hex-de-32-caracteres>
set alert on
alert test
```

No publiques la clave del canal en GitHub. Si usas `set alert.psk`, el firmware guarda la clave y puede dejar `alert.hashtag` como `(unset)`, lo cual es normal: la PSK pasa a ser la fuente real del canal.

## 🗺️ Mapas y visibilidad

Este perfil está preparado para reportar a:

- MeshChile MQTT: `mqtt-msc.meshchile.cl`
- LetsMesh Analyzer EU
- MeshMapper

Para forzar presencia después de configurar:

```text
advert
advert flood
```

La aparición en mapas depende de que el nodo tenga hora válida, WiFi/MQTT conectado, radio correcto y que algún gateway/mapa procese el advert.

Comandos útiles de diagnóstico:

```text
get wifi.status
get mqtt.ntp
get mqtt.ntp.diag
get mqtt1.preset
get mqtt2.preset
get mqtt3.preset
get path.hash.mode
get tx
get public.key
clock
```

## 📟 Pantalla OLED

El perfil Chile muestra información útil para operación en terreno:

- Nombre del nodo.
- Frecuencia y SF.
- Ancho de banda, CR y TX dBm.
- Tamaño de hash de ruta.
- Estado WiFi o IP asignada.

## 📱 Clientes MeshCore

Para companion/client:

- Web: https://app.meshcore.nz
- Android: https://play.google.com/store/apps/details?id=com.liamcottle.meshcore.android
- iOS: https://apps.apple.com/us/app/meshcore/id6742354151?platform=iphone
- NodeJS: https://github.com/liamcottle/meshcore.js
- Python: https://github.com/fdlamotte/meshcore-cli

## 🛠 Hardware

MeshCore soporta varios dispositivos LoRa. Este perfil está enfocado y probado para Heltec V4 con OLED.

## 🔐 Seguridad

No subas a GitHub:

- Claves WiFi.
- Contraseñas admin.
- Llaves privadas MeshCore.
- Coordenadas privadas exactas.
- PSK de canales privados.

El firmware debe compartirse sin secretos. Cada usuario configura sus datos desde webconfig o serial después del flash.

## 📚 Más documentación

- Guía detallada del perfil Chile: [docs/meshchile-observer.md](./docs/meshchile-observer.md)
- Documentación oficial MeshCore: https://docs.meshcore.io
- Flasher oficial MeshCore: https://meshcore.io/flasher
- Observer firmware base: https://observer.gessaman.com/

## 📜 Licencia

MeshCore es software open-source bajo licencia MIT. Puedes usarlo, modificarlo y distribuirlo para proyectos personales, comunitarios o comerciales respetando la licencia original.

## Contribuir

Para cambios generales de MeshCore, usa `dev` como rama base. Para cambios del perfil Chile, abre un issue o PR explicando qué hardware usaste, qué mapa/MQTT probaste y qué comandos de verificación pasaron.
