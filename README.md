# RIoT_OS_Guide-ESP8266
This is a guide to the set-up necessary for using RIoT OS with the ESP8266. The original guide (to which a link can be found under Resources) is not very self-explanatory, so I had to reseach by my own. If anythin is wronw please feel free to point it out.

## Preview
This guide will use the RIOT Docker Toolchain to compile, upload and monitor projects to the ESP8266 development board.

## Dependencies
Follow the official guide to install the dependencies, install docker and other tools.
```
$ sudo apt-get install docker.io
$ pip install pyserial
$ pip install esptool
```

## RIoT OS Docker Image
To install this, simply follow this commands
```
$ git clone https://github.com/gschorcht/riotdocker-Xtensa-ESP.git
$ cd riotdocker-Xtensa-ESP
$ git checkout esp8266_only
$ docker build -t riotbuild .
```
To start it, the official guide uses this command
```
$ cd /path/to/RIOT
$ docker run -i -t --privileged -v /dev:/dev -u $UID -v $(pwd):/data/riotbuild riotbuild
```
However, if you try to use it, an error will prompt because of lack of privileges of the user, so the best way I found is to execute as root, and also change the $UID to 0 (root user). This way you will log as root in the image and have all the permissions.
```
$ sudo docker run -i -t --privileged -v /dev:/dev -u 0 -v $(pwd):/data/riotbuild riotbuild
```
The main folder of the Docker image is the main folder of the git repository, .../riotdocker-Xtensa-ESP, so anything we create here will appear in the RIOT image and the other way around.

What I did was create a "Projects" folder where I copy the examples and tests of RIOT. In this case we are going to use the "default" test. So from inside the RIOT image, type:
```
# mkdir /data/riotbuild/Projects
# cp -r /data/riotbuild/RIOT/examples/default /data/riotbuild/Projects
```
This way we copy the tutorial to a new location and we can modify it as needed.

Now we have to modify a couple of things to get to do all the compile flash monitor process from inside the Docker image in one step. First, there is a typo in this file so we have to modify it from OUTSIDE the Docker image:
```
$ sudo vi ./RIOT/cpu/esp8266/Makefile.include
```
Then search for FLASH_SIZE and replace the value with:
```
FLASH_SIZE ?= -fs 1MB
```
This way we are capable of flashing the ESP8266 with this script. Otherwise it will prompt something like 4m not recognized flash size... duh. Also by using ?= we make it optional, so we can override this from the project Makefile.

Tips for vi:
[ESC] + /FLASH_SIZE --> to find it
[ESC] + dd --> to remove the line
[ESC] + i + paste the new line

Finally, there are a couple of tools we have to install in our Docker image, so type the following lines INSIDE the Docker image:
```
# apt update
# apt install python3-pip
# pip3 install pyserial
```

## Project configuration
Now, there are some things that we need to modify in our project in order to make it compile, flash and run in one line.

### Optional
The way I find best to work is:
(1) Create a "Projects" folder
(2) Copy the example (or template) from the examples folder to Projects
(3) Change owner of all files to your user
```
$ mkdir Projects
# cp -r ./RIOT/examples/[example_or_template]
# chown -R [your_user]:[your_group] Projects/
```
### Configure Makefile
In the Makefile of the project you are working with, we have to enable the ESP8266 with the characteristics needed.

```
BOARD = esp8266-olimex-mod
FLASH_SIZE = -fs 4m # this is the size of the flash memory in MB. See the "Tips and tricks" section for more info
[...]
USEMODULE += esp_wifi # Only if needed, probably needed 'though
[...]
```

## Connecting to the board
Once the project is configured, all we have to do is go to the project's directory in the Docker image and execute the following command:
```
make all flash term
```

## Resources

http://doc.riot-os.org/group__cpu__esp8266.html
https://github.com/RIOT-OS/RIOT/wiki/Use-Docker-to-build-RIOT
https://hub.docker.com/r/schorcht/riotbuild_esp8266?????????????????????
https://github.com/espressif/esptool
https://nodemcu.readthedocs.io/en/master/flash/
https://www.espressif.com/sites/default/files/documentation/2a-esp8266-sdk_getting_started_guide_en.pdf
https://github.com/espressif/esptoolhttps://github.com/RIOT-OS/RIOT/tree/master/examples/default
https://github.com/RIOT-OS/RIOT/tree/master/examples/default
https://github.com/RIOT-OS/RIOT/wiki/Tutorial:-How-to-port-a-radio-module-driver-to-RIOT-OS
https://github.com/RIOT-OS/RIOT/issues/11844
https://github.com/gschorcht?tab=repositories

## Tips and Tricks
### Get ESP8266 flash memory size
With the ESP8266 connected via USB
```
$ esptool.py flash_id
```
Prompt should be something like this:
```
esptool.py v2.7
Found 1 serial ports
Serial port /dev/ttyUSB0
Connecting....
Detecting chip type... ESP8266
Chip is ESP8266EX
Features: WiFi
Crystal is 26MHz
MAC: 84:f3:eb:8e:5d:53
Uploading stub...
Running stub...
Stub running...
Manufacturer: c8
Device: 4016
Detected flash size: 4MB
Hard resetting via RTS pin...
```

### Print IP Address
Add this lines in your project
```
[...]
extern int _gnrc_netif_config(int argc, char **argv);
[...]
/* print network addresses */
puts("Configured network interfaces:");
_gnrc_netif_config(0, NULL);
[...]
```
