# Thread and Matter Examples

This repository contains examples for presentations of Thread and Matter network protocols.

## Requirements

Raspberry Pi 4 (or 4) with following software installed:
- OpenThread Border Router ([codelab](https://openthread.io/codelabs/openthread-border-router#1))
- Home Assistant ([installation](https://www.home-assistant.io/installation/))
- Python Matter Server ([installation](https://github.com/home-assistant-libs/python-matter-server))
- Mosquitto Broker ([Docker](https://hub.docker.com/_/eclipse-mosquitto))
- MQTT-SN Gateway ([Docker](https://hub.docker.com/r/kyberpunk/paho))

Docker images (except OTBR) can be installed using this Docker Compose file: [docker-compose.yaml](compose/docker-compose.yaml).

Development board with OpenThread examnple CLI:
- [nRF52840 example](https://github.com/openthread/ot-nrf528xx/blob/main/src/nrf52840/README.md)
- [ESP32-C6 example](https://github.com/espressif/esp-idf/tree/master/examples/openthread/ot_cli)
- For MQTT examples this OpenThread fork must be used: [kyberpunk/openthread](https://github.com/kyberpunk/openthread)

Development board with Mater Lighting example:
- [nRF52840 example](https://github.com/project-chip/connectedhomeip/tree/master/examples/lighting-app/nrfconnect)
- [ESP32-C6 example](https://github.com/project-chip/connectedhomeip/tree/master/examples/lighting-app/esp32)

## Example setup

![Example setup](files/overview.drawio.png)

## Examples

### Thread commissioning

https://openthread.io/guides/border-router/external-commissioning

Based on OpenThread CLI example. OTBR web GUI or [Android commissioning App example](https://thread-thread-group.en.aptoide.com/app) can be used for external commissioning.

QR Code format:
```
v=1&&eui=f4ce36d7ad8edb94&&cc=J01NU5
```

OpenThread CLI command:
```
ifconfig up
panid 0xffff
joiner start J01NU5
```

Start network after successfull joining:
```
thread start
```

![Thread commissioning recording](files/commissioning.gif)

### Thread internetworking

Based on OpenThread CLI example.

Ping command to OTBR gateway in OpenThread CLI:
```
ping fdf1:eba9:89c5:1:ca04:d50a:9bb6:444d
```

Opposite ping request from Raspberry Pi:
```
ping6 fdae:5a06:af86:5322:82af:21fe:6700:b51c
```

![Ping recording](files/ping.gif)

Communication to the Internet (with NAT64):
```
ping 8.8.8.8
dns resolve google.com 8.8.8.8
curl http://httpbin.org/get
```

![Ping recording](files/internet.gif)

### Matter in Home Assistant

Setup Home Assistant integrations:
- [Thread](https://www.home-assistant.io/integrations/thread/)
- [Matter](https://www.home-assistant.io/integrations/matter/)

Build and commission Lighting App sample following instructions:
- [nRF52840 example](https://github.com/project-chip/connectedhomeip/tree/master/examples/lighting-app/nrfconnect)
- [ESP32-C6 example](https://github.com/project-chip/connectedhomeip/tree/master/examples/lighting-app/esp32)

Alternatively [HA SkyConnect](https://www.home-assistant.io/skyconnect/) dongle can be used for Matter integration.

![Home Assistant](files/homeassistant-153.jpeg)

[Home Assistant recording](files/homeassistant.mp4)

### Home Assistant Matter Commissioning

[![Home Assistant Matter Commissioning](https://img.youtube.com/vi/Fk0n0r0eKcE/0.jpg)](https://www.youtube.com/watch?v=Fk0n0r0eKcE)