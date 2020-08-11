
---
layout: post
title: "Flashing superparasites to Typhoon"
date: 2020-08-11
categories:
  - Synthesizer
tags: [arm, diy, eurorack]
---

I guess you found this post because you want to flash the [superparasites](https://github.com/patrickdowling/superparasites) firmware to [Typhoon](https://github.com/jakplugg/Typhoon). As this is a custom firmware it's not as easy as building and flashing the official MI firmware images. Here's how it worked for me.

Use the Mutable Instruments Vagrant dev environment as pointed out here: [How to get started writing your own firmware for Mutable Instruments Clouds | by Tom Whitwell | Music Thing Modular Notes | Medium](https://medium.com/music-thing-modular-notes/how-to-get-started-writing-your-own-firmware-for-mutable-instruments-clouds-a08173cec317?fbclid=IwAR3LSf2NnDPxCUsaTKJbCYGAIi2X9W6lWwN_0Zq2lNH62wNtsG0sCW1t7q4)
```shell
cd /vagrant
git clone --recursive https://github.com/patrickdowling/superparasites.git
``` 

Download [GNU Toolchain | 5-2016-q3-update – Arm Developer](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads/5-2016-q3-update) to the `/vagrant` shared folder, then

```shell
tar -xvjf gcc-arm-none-eabi-5_4-2016q3-20160926-linux,-d-,tar.bz2 # better do that on the host if possible
sudo mv gcc-arm-none-eabi-5_4-2016q3 /usr/local/
```

Make the bootloader and the firmware:

```shell
cd /vagrant/superparasites/
make -f supercell/bootloader/makefile
make -f supercell/makefile VARIANT=MICROCELL
```

Upload the firmware:

```shell
export PGM_INTERFACE=stlink-v2
export PGM_INTERFACE_TYPE=hla
openocd -s /opt/local/share/openocd/scripts -f interface/stlink-v2.cfg -f target/stm32f4x.cfg -c "init" -c "halt" -c "sleep 200" -f stmlib/programming/jtag/erase_f4xx.cfg -c "flash write_image erase build/microcell/microcell_bootloader_combo.hex" -c "verify_image build/microcell/microcell_bootloader_combo.hex" -c "sleep 200" -c "reset run" -c "shutdown"
```

That should be it. If I forgot anything here, please shoot me a [mail](mailto:machine@chmanie.com).
