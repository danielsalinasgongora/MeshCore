# MeshChile observer profile

This fork adds a public Chile observer profile for Heltec V4 devices running the Gessaman MeshCore observer firmware.

## What the Chile profile sets on a clean flash

- LoRa radio: `927.875 MHz`, `62.5 kHz`, `SF8`, `CR5`.
- Transmit power preference: `22 dBm`.
- Default path hash mode: `1`, which MeshCore uses as 2-byte path hashes.
- MQTT slot 1: `meshchile`, using `wss://mqtt-msc.meshchile.cl:443/mqtt` with JWT device authentication, audience `mqtt-msc.meshchile.cl`, and the Let's Encrypt / ISRG Root X1 certificate chain used by the broker.
- MQTT slot 2: `analyzer-eu`.
- MQTT slot 3: `meshmapper`.
- MQTT status, packets, RX, and TX are enabled for map reporting and bot replies.
- Local adverts default to every 60 minutes and flood adverts default to every 12 hours for fresher map updates.
- Time zone: `America/Santiago`.
- NTP primary server: `ntp.shoa.cl`, with the upstream fallback servers still available.
- OLED status page shows node name, radio, TX power, path hash byte size, and Wi-Fi/IP state.
- Auto-start web panel: when Wi-Fi is connected, the LAN webconfig portal starts automatically.

## Build target

Use this PlatformIO environment:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile
```

For flashing a Heltec V4 over USB serial, use:

```bash
pio run -e heltec_v4_repeater_observer_mqtt_chile -t upload
```

## First boot and web configuration

The firmware keeps personal data out of the image. Do not commit Wi-Fi passwords, admin passwords, private keys, or exact home coordinates.

With the Chile build, the LAN web panel starts automatically after Wi-Fi connects. The serial command `start webconfig` is still available for manual startup, and `start webconfig ap` forces the setup AP when needed.

After flashing, configure each node through serial or webconfig:

1. Connect over serial.
2. Run `start webconfig` to open the web panel on the node's current Wi-Fi connection, or `start webconfig ap` to start the setup access point.
3. Set node name, admin password, Wi-Fi credentials, region/IATA, and coordinates in the web panel.
4. Save and reboot.

Serial fallback for a Chile observer:

```text
set name <node-name>
set password <strong-admin-password>
set wifi.ssid <wifi-name>
set wifi.pwd <wifi-password>
set radio 927.875,62.5,8,5
set tx 22
set path.hash.mode 1
set advert.interval 60
set flood.advert.interval 12
set mqtt.iata SCL
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

## Security notes

The public firmware image intentionally does not include operator credentials, node private keys, private Wi-Fi details, or private coordinates. Each operator should generate or import their own MeshCore keypair and use a unique admin password.

## Web terminal and messages

The web panel includes a CLI terminal that can run the same safe node commands as the serial console, including `advert`, `get wifi.status`, `get mqtt.ntp.diag`, and `alert test`.

Direct chat-style LoRa messages are companion/client behavior. The Chile observer profile also includes a command bot that can answer `#ping`, `#status`, `#wifi`, and `#hora`. It is disabled by default. When enabled, its own channel selector can target either Public or a private bot channel configured by hashtag/PSK from the web panel or CLI. Fault alerts still use the alert-channel guard and continue to reject Public as an alert destination. The observer firmware can transmit configured alert text with `alert test` when an alert channel is configured, and it can publish its own adverts/status to MQTT/maps. If `mqtt.tx` is left at `advert`, bot replies go out over LoRa but are not uploaded as message packets; set `mqtt.tx on` to see bot replies on MQTT-backed message maps.

## Map visibility

Map visibility depends on successful LoRa reception and successful MQTT publishing. After a clean flash and configuration, confirm:

```text
get public_key
get path.hash.mode
get mqtt.ntp
get mqtt.ntp.diag
get mqtt
```

Then send an advert and wait for the map indexers to process it:

```text
advert
advert flood
```
