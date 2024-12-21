I followed the instructions on the [ESPHome Getting Started
page](https://esphome.io/guides/getting_started_command_line.html)

The docker version works fine on Bellman (Linux) and on Mac until the
upload stage. Since the ESP32 is plugged into the Mac, I cannot do a
serial upload from Bellman, and Docker + USB fails on the Mac. So I am
using Conda instead.

```bash
 conda create --name=esphome python
 conda activate esphome
 pip install esphome
``` 

Create a YAML file

   esphome wizard wrover2.yaml
or
   docker run --rm -v "${PWD}":/config -it esphome/esphome wrover2.yaml wizard

Process the YAML file, will attempt OTA upgrade on the Docker version, because it can't find the serial port.

   esphome run wrover2.yaml
or
   docker run --rm -v "${PWD}":/config -it esphome/esphome wrover2.yaml run

Process the YAML and do upgrade over USB port on Linux; this step fails on the Mac because
it will run a virtual machine that cannot see the serial ports.

   docker run --rm -v "${PWD}":/config --device=/dev/tty.usbserial-1410 -it esphome/esphome wrover2.yaml run

## M5StickC notes

The M5StickC devices have an axp192 chip that's not supported by ESPHome, but
it controls the display backlight, so without it, you can't use the display.

I found a water level warning project on ESPHome that has the AXP192 support so I used it.

https://esphome.io/cookbook/leak-detector-m5stickc

Code: https://github.com/airy10/esphome-m5stickC
Better AXP192 code: https://gitlab.com/geiseri/esphome_extras
