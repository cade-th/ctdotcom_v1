---
title: A Bluetooth Keyboard, Part II
date: 2025-04-01
description: Making a bluetooth keyboard from the ground up
tag: Bluetooth
author: Cade Thornton
---

## Zephyr and nrf52840 setup

I'll begin this part as if I were setting up Zephyr from the ground up on a Linux machine (Ubuntu 22.04 in my case), but the steps should be similar on any Linux distribution. If you're on windows, I'd recommend using the nrfConnect VS Code extension (which will limit you to nordic chips), because otherwise, the setup I'm going for is very difficult on anything but a unix machine.

To begin, Zephyr has a meta-tool called West that behaves much like a package-manager/debugger/build tool all in one. Theoretically, the entire zephyr workflow, from setup to blinking an LED, is:
```
mkdir zephyr_workspace && cd zephyr-workspace
pip install west
west init .
west update .
cd ~/zephyr_workspace/zephyr
west sdk install
cd ~/zephyr-workspace
west build -p always -b nrf52840 samples/basic/blinky
west flash

```

Unfortunately, West is written in python so we will not be getting any nice, automatic ways of using it like you would with Cargo, apt, npm, etc. I've been handling python dependencies using python virtual environments just as the official Zephyr docs recommend, so we will go with that. It isn't too annoying, but does require us to use a specific shell and source a bash script like so:

```
mkdir zephyr-workspace
cd zephyr-workspace
python -m venv myenv
```
This creates a myenv directory in the current directory. We can then do 
```
source myenv/bin/activate
```
to activate our virtual environment, which for all we care right now, adds a string before our shell prompt like so:
```
(venv) cade@ubuntu$
```
and allows us to start using pip to install packages. We'll also need the following according to the official zephyr docs:

```
sudo apt install --no-install-recommends git cmake ninja-build gperf \
  ccache dfu-util device-tree-compiler wget \
  python3-dev python3-pip python3-setuptools python3-tk python3-wheel xz-utils file \
  make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1
```

There are three paradigms for making a zephyr project, but I've found the simplest and best one to be making a workspace represented by a directory that all of our zephyr related projects will live in. We've already set this up in zephyr-workspace/

Now we can finally do

```
pip install west
``` 

to get the west tool working. Only a couple more things to do:
1. Get the zephyr source code
2. Get the zephyr sdk
3. Build a blink LED example

Getting the source code is as easy as this:
```
west init .
west update .
cd ~/zephyrproject/zephyr
west sdk install
```

That should take a while to install, but when it's done, we're ready to do a basic blinky!
All that will take is:
```
west build
```
but we'll add some flags to indicate our board and a sample blinky code:
```
cd ~/zephyrproject/zephyr
west build -p always -b nrf52840 samples/basic/blinky
```

and drum roll please ...

```
west flash
```

There we go! A minimal Zephyr setup and an LED blinked. This kind of developer-friendly tooling with west is an ***extremely*** rare gem in the embedded systems industry. I believe it's what most microcontroller vendors will target in the future, especially once someone (maybe you or me?) develops a VS Code extension in the flavor of Platform IO but just for the Zephyr project. Nordic Semiconductor has already done something similar with their nrfConnect VS Code extension, but you obviously can't use that for any other MCU like an STM32. With such tooling, we could finally move on from Arduino being the standard for embedded systems education, and get a full bluetooth and usb device stack into the hands of students everywhere!

But I digress. Now, we will set up our source code and begin getting a "hello world" example running... on USB!



