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

