Arduino ESP-NOW
===============

I found the Arduino IDE experience with ESP32 underwhelming. Previously, I've worked directly with the underlying [ESP-IDF](https://idf.espressif.com/) from Espressif. With ESP-IDF, you have to configure things explicilty, e.g. you have to specify that the chip type is `esp32c3`, whereas the Arduino IDE tries to guess things - if this worked all the time, it'd be great - however, it doesn't and when it gets things wrong the illusion that it's handling things for you crumbles and you have to understand what's going on in the guts of the thing in order to resolve the issue.

However, many people use the Arduino IDE and are familiar with it so, I decided to use it for this project. Below, there's a trouble shooting section on some of the issues that I came across.

Download the Arduino IDE [here](https://www.arduino.cc/en/software).

When it starts up, it'll download various packages that it needs and you'll see output like this in its console:

```
Downloading packages
arduino:arduinoOTA@1.3.0
arduino:avr-gcc@7.3.0-atmel3.6.1-arduino7
...
Downloading Stepper@1.1.3
Stepper@1.1.3
Installing Stepper@1.1.3
Installed Stepper@1.1.3
```

If your ESP32 dev board is plugged in, the IDE will detect it and you can select it from the _Select Board_ dropdown.

A little popup appears telling you that the _esp32 [v ...] core has to be installed_, just click _Yes_ (rather than _Install Manually_).

The console displays something like this:

```
Downloading packages
arduino:dfu-util@0.11.0-arduino5
esp32:esptool_py@4.5.1
esp32:mklittlefs@3.0.0-gnu12-dc7f933
...
esp32:xtensa-esp32s2-elf-gcc@esp-2021r2-patch5-8.4.0 installed
Installing esp32:xtensa-esp32s3-elf-gcc@esp-2021r2-patch5-8.4.0
Configuring tool.
esp32:xtensa-esp32s3-elf-gcc@esp-2021r2-patch5-8.4.0 installed
Installing platform esp32:esp32@2.0.16
Configuring platform.
Platform esp32:esp32@2.0.16 installed
```

Once ready, I pasted in the following short sketch slightly adapted from one on [Random Nerd Tutorials](https://randomnerdtutorials.com/get-change-esp32-esp8266-mac-address-arduino/):

```
#include <WiFi.h>

void setup(){
  Serial.begin(115200);
  WiFi.mode(WIFI_MODE_STA);
}

void loop(){
  Serial.print("ESP Board MAC Address:  ");
  Serial.println(WiFi.macAddress());
  Serial.println();
  delay(500);
}
```

And pressed the _Upload_ button (the right pointing arrow). After a lot of trouble shooting (see _Trouble shooting_ section below), this finally worked - hopefully, it works first time for you.

Once it worked, I saw this output in the console:

```
ketch uses 235962 bytes (18%) of program storage space. Maximum is 1310720 bytes.
Global variables use 15884 bytes (4%) of dynamic memory, leaving 311796 bytes for local variables. Maximum is 327680 bytes.
esptool.py v4.5.1
...
Wrote 248064 bytes (138604 compressed) at 0x00010000 in 2.0 seconds (effective 996.8 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

Then I opened the serial monitor tab via the menu _Tools / Serial Monitor_ and changed the baud rate to 115200 and **pressed the tiny RESET button on my board**.

After the board reset, the serial monitor displayed:

```
ESP Board MAC Address:  64:E8:33:00:89:D4

ESP Board MAC Address:  64:E8:33:00:89:D4

...
```

### No serial data received

Often, on trying to upload a new sketch, I got the error:

```
A fatal error occurred: No serial data received.
Failed uploading: uploading error: exit status 2
```

I _believe_ this happens when the existing sketch (like the one above) ties up the serial port in some way.

The way to resolve this was to hold down the tiny BOOT button on my board, then press and release the similarly tiny RESET button and finally release the BOOT button.

This puts the board into a mode where it won't run the existing sketch and a new sketch can now successfully be uploaded.

### Discovering other ESP32s via broadcast

HERE HERE

Trouble shooting
----------------

This section covers various issues I encountered while using the Arduino IDE.

### No module named 'serial'

On trying to upload my first sketch, I got this error:

```
Traceback (most recent call last):
  File "/home/ghawkins/.arduino15/packages/esp32/tools/esptool_py/4.5.1/esptool.py", line 31, in <module>
    import esptool
  File "/home/ghawkins/.arduino15/packages/esp32/tools/esptool_py/4.5.1/esptool/__init__.py", line 41, in <module>
    from esptool.cmds import (
  File "/home/ghawkins/.arduino15/packages/esp32/tools/esptool_py/4.5.1/esptool/cmds.py", line 14, in <module>
    from .bin_image import ELFFile, ImageSegment, LoadFirmwareImage
  File "/home/ghawkins/.arduino15/packages/esp32/tools/esptool_py/4.5.1/esptool/bin_image.py", line 14, in <module>
    from .loader import ESPLoader
  File "/home/ghawkins/.arduino15/packages/esp32/tools/esptool_py/4.5.1/esptool/loader.py", line 30, in <module>
    import serial
ModuleNotFoundError: No module named 'serial'
exit status 1

Compilation error: exit status 1
```

This seems to be a common, but unresolved problem, on Debian/Ubuntu Linux systems. To resolve it:

```
$ sudo apt install python3-serial
```

However, that still doesn't resolve the issue if you're using a Python version manager like `pyenv`.

### Pyenv managed Python doesn't use system serial package

The `python3-serial` package is installed to `/usr/lib/python3/dist-packages/serial`.

However, if you're using `pyenv` then it's managed version of Python will not look in this location.

To resolve this, you can tell `pyenv` to use the system Python version in the terminal from which you're going to launch the Arduino IDE:

```
$ pyenv shell system
$ python3
Python 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import serial
>>>
```

If entering `import serial` works without problem then everything is working and you can start the Arduino IDE:

```
$ cd .../arduino-ide_2.3.2_Linux_64bit
$ ./arduino-ide
```

### IDE auto-detects the wrong board type

The Arduino IDE found the port that my board was connected to but misidentified it as an ESP32-S3-USB-OTG board when in fact it was a Seeed Xiao C3 board.

This only became obvious when I tried to upload my first sketch (after having resolved the issues above) and got the error:

```
A fatal error occurred: This chip is ESP32-C3 not ESP32-S3. Wrong --chip argument?
Failed uploading: uploading error: exit status 2
```

This was resolved by going to the boards dropdown and selecting _Select other board and port..._ I entered "C3" in the search field and found _XIAO_ESP32C3_ there and selected that.

Note: you can also select the board via the _esp32_ submenu under _Tools / Board_ but there the list of boards is huge, unsorted and unsearchable. Another option is the _Board Manager_ but this is usually useful when the IDE does not already have a definition for the relevant board installed.

### Protocol error on uploading sketch

On uploading my first sketch, after resolving the issues above, I got the following `OSError: [Errno 71] Protocol error` when it was "Hard resetting via RTS pin...":

```
Leaving...
Hard resetting via RTS pin...
Traceback (most recent call last):
  File "/home/ghawkins/.arduino15/packages/esp32/tools/esptool_py/4.5.1/esptool.py", line 34, in <module>
    esptool._main()
...
  File "/usr/lib/python3/dist-packages/serial/serialutil.py", line 463, in rts
    self._update_rts_state()
  File "/usr/lib/python3/dist-packages/serial/serialposix.py", line 708, in _update_rts_state
    fcntl.ioctl(self.fd, TIOCMBIC, TIOCM_RTS_str)
OSError: [Errno 71] Protocol error
Failed uploading: uploading error: exit status 1
```

Many people seem to have seen this problem but I found few clear solutions. What worked for me was to burn the latest Arduino bootloader to the board.

I first tried selecting the menu item _Tools / Burn Bootloader_ but this just resulted in the very short error:

```
No programmer
```

To resolve this you have to go to the _Tools / Programmer_ menu and select _Esptool_ (it's the only item in the submenu but unless you select it, it remains unticked and the IDE doesn't know what programmer to use).

Once, this is done, I tried _Tools / Burn Bootloader_ and got the error:

```
esptool.py v4.5.1
Serial port /dev/ttyACM0
Connecting...
Chip is ESP32-C3 (revision v0.4)
Features: WiFi, BLE
Crystal is 40MHz
MAC: 64:e8:33:00:89:d4
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 921600

A fatal error occurred: No serial data received.
Failed chip erase: uploading error: exit status 2
```

As noted up above, when this happens you have to hold down the board's BOOT button, press and release its RESET button and then release the BOOT button. Once this is done, retry _Tools / Burn Bootloader_.

### Using esptool.py to upload a new bootloader

If all else fails you can write the bootloader to the board using Espressif's own `esptool.py`.

First, find the bootloader file like so:

```
$ ls ~/.arduino15/packages/esp32/hardware/esp32/2.0.16/tools/sdk
esp32/  esp32c3/  esp32s2/  esp32s3/  versions.txt
$ ls ~/.arduino15/packages/esp32/hardware/esp32/2.0.16/tools/sdk/esp32c3/bin
bootloader_dio_40m.elf  bootloader_dio_80m.elf  bootloader_dout_40m.elf  bootloader_dout_80m.elf  bootloader_qio_40m.elf  bootloader_qio_80m.elf  bootloader_qout_40m.elf  bootloader_qout_80m.elf
```

I.e. first you find the subdirectory that corresponds to your chip type, in my case as `esp32c3`, then in the `esp32c3/bin` subdirectory, you find various bootloader files with things like `_qio_` and `_80m_` in their names.

If you look under the _Tools_ menu in the Arduino IDE, you'll see:

* Flash Frequency: "80 MHz"
* Flash Mode "QIO"

So, you want the bootloader file with `_80m_` (for the frequency) and `_qio_` (for flash mode) in its name.

Then erase the board and write the bootloader to the board using the ESP-IDF `esptool.py`:

```
$ esptool.py --chip esp32c3 --port /dev/ttyACM0 erase_flash
esptool.py v4.7.0
...
Chip erase completed successfully in 14.3s
Hard resetting via RTS pin...
$ esptool.py --chip esp32c3 --port /dev/ttyACM0 --baud 460800 write_flash -z 0x0 ~/.arduino15/packages/esp32/hardware/esp32/2.0.16/tools/sdk/esp32c3/bin/bootloader_qio_80m.elf
esptool.py v4.7.0
...
Wrote 657376 bytes (257964 compressed) at 0x00000000 in 4.6 seconds (effective 1153.7 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

The important thing is to know the address for the `write_flash` command, i.e. `-z 0x0` above. `0x0` is the right value for ESP32-S3 and C3 boards, but for classic ESP32 boards and S2 boards, the value is `0x1000`.

Installing `esptool.py` is described [here](https://docs.espressif.com/projects/esptool/en/latest/esp32/installation.html).

An easier alternative that _may_ work for you, is to use the Espressif [online flasher tool](https://espressif.github.io/esptool-js/). In _Program_ section, set the baudrate to 460800 and click _Connect_ and select the appropriate port from the list that pops up.

It should work with recent versions of Google Chrome and Microsoft Edge, but I just got the error:

```
esptool.js
Serial port WebSerial VendorID 0x303a ProductID 0x1001
Connecting...Error: Failed to execute 'open' on 'SerialPort': Failed to open serial port.
```

If it works for you then you first need to press the _Erase Flash_ button, then the _Choose File_ button and select the appropriate `bootloader_xyz.elf` file (see above), set the _Flash Address_ to 0x1000 for classic ESP32 and S2 boards and to 0x0 for S3 and C3 boards and finally click _Program_.
