### Getting started with ESP32 and MicroPython
**Last update:** 2020-03-31 11:00:00 -0300.

[Send me a message](mailto:desconstruindo@furansa.me?subject=Comments%20on%20article%20Getting%20started%20with%20ESP32%20and%20MicroPython) and let me know your comments.

#### Introduction
In this article I'm going to share my first experience playing with ESP32
microcontroller and MicroPython language. The main goal here is to setup the
whole environment for development, sharing the steps taken to run my first 
program, because you know, sometimes even the most straight forward task can
be very tricky to acomplish.

I'm a electrical engineer post graduated in automotive embedded systems and I've
been working as software developer for 20 years, but never played with ESP32.
I remember at the first time I've heard about ESP32, I was very busy with a new
job, developing a computer vision embedded system that was Raspberry Pi based.

But last year I was mentoring some internals in my former company and they shown
me some ESP32 based boards they were using in their projects, and I was very
happy to see how fast they could prototype to build a proof of concept for these
projects.

I mean, I love bare metal and build everything from scratch, but sometimes you
just need to test an idea to see if it will work as expected, and I remember
when I was in the college and my first contact with the Arduino platform.

And why Python? I heard about MicroPython in 2016 and probably I was looking
for an oportunity to check this out. To be very honest, I don't think that
Python would be my first language of choice for an embedded project, surelly
my options would be C, C++ and Lua, in that order, for many reasons that are
out of the scope for this article. But let's see how the things goes, may be
I'll be surprised.

Enough talking. To get started, it's necessary to install all the tools to
communicate with the board, upload the MicroPython firmware and run the first
program to check if everything is working as expected.

There are two aspects that sometimes can be very annoying: since that there are
a lot of manufacturers around, there are also a lot of ESP32 based boards
around. Also, due to differences in some softwares versions, it's very common
to face some unexpected behaviour while trying to put everything up and
running together. It worth to mention also the lack of proper documentation.

The devil is in the details and in fact, that's why I wrote this article. The
steps here are basically the same from the official documentations, always read
the fucking manuals first (when available of course). My personal touch comes in
the specific details, the approach and reasonable order for the steps and
troubleshooting.

Here's the hardware and software specifications for this article:

* TTGO board with Espressif ESP-WROOM-32 microcontroller, ESP32D0WDQ6 revision
1 to be precise. This board comes with WiFi, Bluetooth, OLED display and 18650
battery holder;

* Ubuntu Linux 20.04 with Python 3.8;

* MicroPython firmware for ESP32 generic v1.12-310-g9418611c8;

* Visual Studio Code 1.43.2.

#### Set up the environment
Make sure that the current user is part of the ```dialout``` group:

```shell
$ id furansa | grep dialout
```

If is not, add it, then **log out and in** to make sure the changes will be
propagated across the whole system:

```shell
$ sudo adduser furansa dialout
```

Plug the board in some of the USB ports and check if it will be recognized,
and if the ```cp210x``` kernel module will be loaded. This also tells which port
the board will be accessible:

```shell
$ sudo dmesg
usbserial: USB Serial support registered for generic
usbcore: registered new interface driver cp210x
usbserial: USB Serial support registered for cp210x
cp210x 1-7:1.0: cp210x converter detected
usb 1-7: cp210x converter now attached to ttyUSB0
```

It's supposed to work out of the box for Ubuntu 20.04 and here, the board is
accessible at ```/dev/ttyUSB0```.

Some software are mandatory to be installed: ```esptool``` allows to flash the
board and upload the MicroPython firmware and ```screen```, the very famous
terminal multiplexer and emulator.

```shell
$ sudo apt-get install esptool screen
```

#### Flash and upload the MicroPython firmware
**Warning:** this procedure can do permanent damage to the board, so, you're at
your own. That said, let's continue >:-)

After download the MicroPython firmware for ESP32 boards from
[here](https://micropython.org/download#esp32){:target="_blank"} (I'm using the
generic v1.12-310-g9418611c8 as already mentioned), it's possible to proceed by
first erasing the current firmware:

```shell
$ esptool --chip esp32 --port /dev/ttyUSB0 erase_flash

esptool.py v2.8
Serial port /dev/ttyUSB0
Connecting........_____....._
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 24:62:ab:bb:d8:44
Enabling default SPI flash mode...
Erasing flash (this may take a while)...

A fatal error occurred: ESP32 ROM does not support function erase_flash.
```

Ouch! Barely started and the first critical error already showed up.

After look some posts at [esptool GitHub](https://github.com/espressif/esptool){:target="_blank"}
and [MicroPython forum](https://forum.micropython.org){:target="_blank"}, my
best guess is that this error can be related with some incompatibility between
the ```esptool``` and the current firmware. This is obvious by the error
message, but I found few people reporting the same problem with older versions
of ```esptool```, and at the time of this writing I'm using the latest version
(2.8). So, if you have a reasonable explanation for this error, I'll appreciate
if you [contact](http://desconstruindo.furansa.me/about){:target="_blank"} me
and let me know.

After try different options without luck, with different baud rates for example,
I decided to ignore this error and proceed with the upload of the MicroPython
firmware:

```shell
$ esptool --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x1000 esp32-idf3-20200327-v1.12-310-g9418611c8.bin

esptool.py v2.8
Serial port /dev/ttyUSB0
Connecting........_____....
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 24:62:ab:bb:d8:44
Changing baud rate to 460800
Changed.
Enabling default SPI flash mode...
Configuring flash size...
Auto-detected Flash size: 4MB
Erasing flash...
Compressed 1442640 bytes to 917380...
Took 3.83s to erase flash block
Wrote 1442640 bytes (917380 compressed) at 0x00001000 in 28.5 seconds (effective 405.3 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

Hm, interesting! Looks like the upload was successfully completed and honestly
I'm not sure if this is good, let's see.

#### Connect to the board and access the REPL prompt
The REPL (Read Evaluate Print Loop) prompt is the interactive environment where
you can type some commands and see its output, as the same way we do in a
regular Python implementation. This is very convenient and one of the advantages
of MicroPython, so, let's connect using ```screen``` and try it out:

```shell
$ screen /dev/ttyUSB0 115200

FAT filesystem appears to be corrupted. If you had important data there, you
may want to make a flash snapshot to try to recover it. Otherwise, perform
factory reprogramming of MicroPython firmware (completely erase flash, followed
by firmware programming).
```

Oh, really man? I was not expecting a walk in the park but I was almost at the
point of considering return back to the chicken farm. Again, I was not able to
find out something more conclusive, but this
[post](https://github.com/micropython/micropython/issues/4747){:target="_blank"}
gave me some insight.

How I solved the problem: despite this error message, it was possible to access
the REPL prompt after hitting the ```CTRL + C``` to stop the error printing.

```shell
FAT filesystem appears to be corrupted. If you had important data there, you
may want to make a flash snapshot to try to recover it. Otherwise, perform
factory reprogramming of MicroPython firmware (completely erase flash, followed
by firmware programming).

Traceback (most recent call last):
  File "_boot.py", line 11, in <module>
  File "inisetup.py", line 34, in setup
  File "inisetup.py", line 15, in check_bootsec
  File "inisetup.py", line 30, in fs_corrupted
KeyboardInterrupt: 
MicroPython v1.12-310-g9418611c8 on 2020-03-27; ESP32 module with ESP32
Type "help()" for more information.
```

And now we are able to format the board filesystem:

```python
>>> import os
>>> os.VfsFat.mkfs(bdev)
```

So far so good (hope so), let's test the system.

#### Test the system
Let's play around and check if the system (hardware and software) is responding
as expected. Still from the REPL, let's blink the LED connected at GPIO pin 16.

Alternatively, it's also possible to use the ```miniterm``` emulator that comes
with PySerial module, after installed, the use is similar to ```screen```: 

```shell
$ sudo apt-get install python3-serial
$ python3 -m serial.tools.miniterm /dev/ttyUSB0 115200
```

And once connected:

```python
>>> import machine
>>> pin16 = machine.Pin(16, machine.Pin.OUT)
>>> print(pin16.value())
0
>>> pin16.value(1)
```

Following the help message from the REPL terminal, it's possible to connect to
the wireless network:

```python
>>> import network
>>> sta_if = network.WLAN(network.STA_IF); sta_if.active(True)
>>> sta_if.connect("NETWORK_NAME", "NETWORK_PASSWD")
>>> sta_if.isconnected()
True
```

Looks like everything is going well. And now for something completely different,
or at least more interesting.

#### Upload files to the board
By uploading files to the board we'll be able to do more interesting things.
There are at least two tools to help with this, ```ampy``` and ```rshell```.

When using Python's pip to install packages, in general is not a good ideia to
do this as root and install the packages system-wide. This can cause some
incompatibilities in the future or even with already installed packages. The
most indicated is to install as regular user or even better, to create a Python
[virtual environment](https://docs.python.org/3/tutorial/venv.html){:target="_blank"}
for these.

Here I'm going to install both system-wide because it's a dedicated system:

```shell
root@antares:~# pip3 install adafruit-ampy
...
Installing collected packages: python-dotenv, adafruit-ampy
Successfully installed adafruit-ampy-1.0.7 python-dotenv-0.12.0
```

```shell
root@antares:~# pip3 install rshell
...
Installing collected packages: pyudev, rshell
Successfully installed pyudev-0.22.0 rshell-0.0.27
```

Both ```ampy``` and ```rshell``` looks quite the same and at the very first
moment, I've only missed a more detailed help documentation. Let's perform some
simple operations to put and list files inside the board.

This will allow us to think about a more structured organization for the future
projects, for example, by separing the code blocks by functionality and/or
responsability in Python modules inside different directories and files. That's
a good practice for any software architecture.

Initially, let's create two simple files: ```boot.py``` that will hold the code
that the system will execute after booting and ```main.py```, as a main entry
program.

The code is pretty straight forward, for the ```boot.py```:

```python
import esp

esp.osdebug(None)  # Disable debugging messages
```

And for the ```main.py```:

```python
from time import sleep
from machine import Pin

# Configure GPIO pin 16 as output
pin16 = Pin(16, Pin.OUT)

# Blink the board's LED at pin16 5 times
for i in range(5):
    pin16.value(1)
    sleep(0.5)
    pin16.value(0)
    sleep(0.5)
```

This is OK for a hardware Hello World! Now to upload these files with both 
```ampy``` and/or ```rshell```, notice that when using ```rshell``` the file is
copied into the ```/pyboard``` directory inside the board:

```shell
$ ampy --port /dev/ttyUSB0 put boot.py
$ rshell --port /dev/ttyUSB0 cp main.py /pyboard/
```

To list the copied files:

```shell
$ ampy --port /dev/ttyUSB0 ls
$ rshell --port /dev/ttyUSB0 ls /pyboard
```

Now with the files already in place it's possible to turn-off and turn-on the
board and see the magic happening.

#### Play with the OLED display
Let's finish with style by playing with the SSD1306 OLED display. Download the
driver created by Adafruit from 
[here](https://raw.githubusercontent.com/RuiSantosdotme/ESP-MicroPython/master/code/Others/OLED/ssd1306.py){:target="_blank"}
and copy to the board:

```shell
$ rshell --port /dev/ttyUSB0 cp ssd1306.py /pyboard/
```

Modify and upload our ```main.py``` that now will import the driver and write
to the display:

```python
from time import sleep
from machine import I2C, Pin

import ssd1306

# Configure GPIO pin 16 as output
pin16 = Pin(16, Pin.OUT)

# Blink the board's LED at pin16 5 times
for i in range(5):
    pin16.value(1)
    sleep(0.5)
    pin16.value(0)
    sleep(0.5)

# I2C interface pins configuration
i2c = I2C(-1, scl=Pin(4), sda=Pin(5))

# Configure the display
oled_width = 128
oled_height = 64
oled = ssd1306.SSD1306_I2C(oled_width, oled_height, i2c)

# Setup the text and show the message
oled.text("Hello", 0, 0)
oled.text("MicroPython", 0, 10)
oled.text("at ESP32", 0, 20)
oled.show()
```

After restart the board you should be able to see the LED blinking five times
and the message will be displayed.

#### Visual Studio Code as IDE
There are some extensions to work with MicroPython using Visual Studio Code and
I choose the one recently released by Seeed.

This extension allow to connect to the board by clicking at the "Device
Connection/Disconnection" icon in the status bar, and selecting the serial port.

After successfully connected, select the "Open MicroPython Terminal" icon at the
status bar and voila, you'll be in the REPL prompt.

#### Conclusion
In this article it was possible to walk through the first steps to set up and
run MicroPython in the ESP32 board. Now it's possible to use this environment to
start to structure and develop a more complex embedded system project.

And that's what I'm going to do in the next articles, by developing a monitoring
system that will perform data acquisition, processing and visualization, also
integrating with AWS.

#### References
* [ESP32 WROOM Series](https://www.espressif.com/en/products/hardware/esp-wroom-32/overview){:target="_blank"}

* [ESP32-WROOM-32 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf){:target="_blank"}

* [ESP8266 and ESP32 serial bootloader utility](https://github.com/espressif/esptool){:target="_blank"}

* [MicroPython Quick reference for the ESP32](http://docs.micropython.org/en/latest/esp32/quickref.html){:target="_blank"}

* [MicroPython Basics: Load Files & Run Code](https://learn.adafruit.com/micropython-basics-load-files-and-run-code){:target="_blank"}

* [Getting Started with MicroPython on ESP32 and ESP8266](https://randomnerdtutorials.com/getting-started-micropython-esp32-esp8266){:target="_blank"}

* [Flash/Upload MicroPython Firmware to ESP32 and ESP8266](https://randomnerdtutorials.com/flash-upload-micropython-firmware-esp32-esp8266){:target="_blank"}

* [MicroPython: OLED Display with ESP32 and ESP8266](https://randomnerdtutorials.com/micropython-oled-display-esp32-esp8266){:target="_blank"}

* [ESP32 TTGO dev board with OLED Display Tutorial](https://electricnoodlebox.wordpress.com/tutorials/esp32-ttgo-dev-board-with-oled-display-tutorial){:target="_blank"}
