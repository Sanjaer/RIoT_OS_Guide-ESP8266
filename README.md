# RIoT_OS_Guide-ESP8266
This is a guide to the set-up necessary for using RIoT OS with the ESP8266. The original guide (to which a link can be found under Resources) is not very self-explanatory, so I had to reseach by my own. If anythin is wronw please feel free to point it out.

## Preview
This guide will use the RIOT Docker  Toolchain to compile the projects.

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
However, if you try to use it, an error will prompt for lack of privileges of the user, so the best way I found is to execute as root, and also change the $UID to 0 (root user). This way you will log as root in the image and have all the permissions.
```
$ sudo docker run -i -t --privileged -v /dev:/dev -u 0 -v $(pwd):/data/riotbuild riotbuild
```
The main folder of the Docker image is the main folder of the git repository, .../riotdocker-Xtensa-ESP, so anyting we create here will appear in the RIOT image and the other way around.

What I did was create a "Projects" folder where I copy the examples and tests of RIOT. In this case we are going to use the "default" test. So from inside the RIOT image, type:
```
# mkdir /data/riotbuild/Projects
# cp -r /data/riotbuild/RIOT/examples/default /data/riotbuild/Projects
```
## Project configuration

## Connecting to the board

## Resources
