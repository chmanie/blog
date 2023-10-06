+++
title = "STM32 - Erasing the flash using OpenOCD"
date = 2020-08-19

[taxonomies]
categories = ["Embedded"]
tags = ["arm", "openocd", "stm32"]
+++

Let's say you have flashed your STM32 MCU but accidentally overwrote a GPIO configuration used for ST-Link communication (discovery boards are prone to this). You might get the error message when trying to flash:

<!-- more -->

```
Error: init mode failed (unable to connect to the target)
```

There is a handy tool with the straightforward name [STM32 ST-LINK utility](https://www.st.com/en/development-tools/stsw-link004.html#get-software) which can be used to perform a [full chip erase](https://www.newbiehack.com/categories/newbiehack-tutorial-ARM-video9-created5202016113040AM-nomenu). Unfortunately this tool only works in Windows :(.

So what we unix users are "stuck with" is [OpenOCD](http://openocd.org/) (which is awesome!). To wipe the flash we have to first put the device into reset mode. We can do this by pulling the `NRST` pin (see your chip's reference manual) low. The blog post I linked to above has some more information on this.

Users of discovery boards can just press and hold the reset button. Then you can issue the following command:

```bash
openocd -f interface/stlink-v2.cfg -f target/stm32f3x.cfg -c "init" -c "halt" -c "wait_halt" -c "stm32f1x mass_erase 0" -c "sleep 200" -c "reset run" -c "shutdown"
```

You need to replace `target/stm32f3x.cfg` with the appropriate config file for your MCU. You can also try to use the configuration file for the discovery board that you're using:

```bash
openocd -f board/stm32f3discovery.cfg -c "init" -c "halt" -c "wait_halt" -c "stm32f1x mass_erase 0" -c "sleep 200" -c "reset run" -c "shutdown"
```

This should have "unbricked" your device.

During the process you might get the error:

```
Error: timed out while waiting for target halted
```

I found that this is a bit of a timing issue. I found the following process to be working for me on the STM32F3DISCOVERY:

1) Hold the reset button, then plug in the USB connector to power on the device
2) Wait for about 1-2 seconds
3) Release the reset button, then immediately issue the command above

Good luck!
