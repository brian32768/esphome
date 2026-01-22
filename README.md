I followed the instructions on the [ESPHome Getting Started
page](https://esphome.io/guides/getting_started_command_line.html)

```bash
 conda create --name=esphome python pip
 conda activate esphome
 pip install esphome
``` 

Create a YAML file

```bash
esphome wizard wrover2.yaml
```

Process the YAML file, will attempt OTA upgrade on the Docker version using a unique name based on the board and a suffix, 
the suffix is normally the last six of the MAC. The suffix
is specified on the command line to avoid having to have a 
unique YAML for each group of devices. The right name then
ends up in the DHCP server too, so the hostname can be used
instead of the IP address. Relieving me from having to nail
down every switch and dimmer in the DHCP server, yay!

```bash
esphome -s suffix 12AB34 run wrover2.yaml
```

Process the YAML and do upgrade over USB port on Linux.

```bash
docker run --rm -v "${PWD}":/config --device=/dev/tty.usbserial-1410 -it esphome/esphome wrover2.yaml run
```

Do the upgrade over the air.

Here's an example of updating a Martin Jerry switch. 

```bash
esphome run --device 192.168.2.47 --client-id mj-s01-7ac70c mj-s01.yaml
```

## ESP-C3-12F notes

This is a device that looks like an ESP8266 based ESP-12F
but has a C3 processor so it's different. But ESPhome supports it.
I am going to solder one into a Martin Jerry switch to test the concept.

```bash
esphome wizard mj-c3.yaml
```

1. I named my node mj1. I am sure this is a duplicate but all the nodes are in a box on the shelf right now so I will risk it.

2. I set CPU to ESP32 but there was no option for this board so I took a stab at "esp32-c3-mli-kit" which is "Ai-Thinker ESP-C3-M1-I-Kit". 
This is a NodeMCU look-alike.
I think ESPHome should support any board that PlatformIO supports.
You can override settings. See https://docs.platformio.org/en/latest/boards/espressif32/esp32-c3-m1i-kit.html.
I suspect as long as I leave it plugged into the programmer it will
work just like the NodeMCU board.

3. I set it to use wildsong2 wifi

4. I did not set any OTA password.

I should be able to build now, but this failed with a missing arduino file.
I don't even want to use Arduino, and I found that I could edit the YAML
and change framework: type: arduino to esp-idf then I was off to the races.

```bash
esphome config mj-c3.yaml
esphome compile mj-c3.yaml
```

esptool.py --before default_reset --after hard_reset --baud 115200 --port /dev/ttyUSB0 --chip esp32c3 write_flash -z --flash_size detect 0x10000 .esphome/build/mj1/.pioenvs/mj1/firmware.bin 0x0 .esphome/build/mj1/.pioenvs/mj1/bootloader.bin 0x8000 .esphome/build/mj1/.pioenvs/mj1/partitions.bin 0x9000 .esphome/build/mj1/.pioenvs/mj1/ota_data_initial.bin

## M5StickC notes

### Water heater

I am using an RS485 hat.
https://wiki.wildsong.biz/index.php?title=M5StickC

I set the RS485 pins in econet_base.yaml

### Display

For a year or so the display would not work because the AXP192 power management unit was not supported.

I found a water level warning project on ESPHome that has the AXP192 support so I used it.

https://esphome.io/cookbook/leak-detector-m5stickc

Code: https://github.com/airy10/esphome-m5stickC
Better AXP192 code: https://gitlab.com/geiseri/esphome_extras

