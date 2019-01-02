---
layout: post
title: BLE_CTF - Setup, compilation and flashing
categories: infosec
---

I wanted to get into Bluetooth Low Energy for quite some time now, but due to my OSCP studies I had other priorities. Thankfully, last month I passed, so now I can focus a bit on IoT/hardware hacking and my first step was the [BLE_CTF](https://github.com/hackgnar/ble_ctf) project by [@hackgnar](https://twitter.com/hackgnar).

To do the **BLE_CTF** I had two options: [buy the pre-flashed ESP32](https://www.ebay.com/itm/173370426012?ssPageName=STRK:MESELX:IT&_trksid=p3984.m1558.l2649) (this also supports [@hackgnar](https://twitter.com/hackgnar)) or build the whole thing [from source](https://github.com/hackgnar/ble_ctf). As I have too many microcontrollers lying around I choose the latter and documented the steps as well.

### Install dependencies
First we need to install a list of dependencies that will be used later.
 
    sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python python-pip python-setuptools python-serial

### Download and extract toolchain
Next we will create the directories for the toolchain, wget the archive and extract it. The toolchain contains the compiler, linker and other required tools.

    mkdir -p ~/esp
    cd ~/esp
    wget https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz
    tar -xzf ~/Downloads/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz

### Install Espressif IoT Development Framework
We need this as well for the ble_ctf build to work.

    git clone --recursive https://github.com/espressif/esp-idf.git
    export IDF_PATH=~/esp/esp-idf
    python -m pip install --user -r $IDF_PATH/requirements.txt

### Export toolchain location
We need this so make knows where to find our toolchain.

    export PATH="$HOME/esp/xtensa-esp32-elf/bin:$PATH"

### Cloning the ble_ctf git repository into our home folder

    cd ~
    git clone https://github.com/hackgnar/ble_ctf
    cd ble_ctf

### Adding your user to the dialout group
In most cases you probably don't have the permissions to read/write /dev/ttyUSB0. Adding yourself to the dialout group fixes it.

    sudo usermod -a -G dialout $USER

### Compiling and flashing the ESP32
Check if PATH contains the toolchain and that IDF_PATH exists. Check whether /dev/ttyUSB0 (or similar) is accessible. make menuconfig changes are only required if you don't use the default /dev/ttyUSB0.

    make menuconfig
    make
    make flash

The *make* should compile **ble_ctf** from source and *make flash* should flash the ESP32 with the binary. 

### Troubleshooting
If something went wrong you should probably check the following first:
* Is the toolchain in the PATH variable?
* Does the IDF_PATH variable exist?
* Are the paths correct? Is everything where it should be?
* Does /dev/ttyUSB0 exist? Are you using non-'power only' USB cable?
* Do you have read/write access to /dev/ttyUSB0?

If you are still stuck, follow the official steps for installing the toolchain and the development framework: [Setup Toolchain](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/#setup-toolchain) If that didn't work either, you probably should buy the pre-flashed ESP32 instead.

