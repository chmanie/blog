+++
title = "ARM development with platformIO"
date = 2020-07-07
draft = true

[taxonomies]
categories = ["Programming"]
tags = ["vim", "c++"]
+++

## Where's my package manager?

Well, in C(++) land, at least for microcontrollers, there is nothing that's really comparable to something like [npm](https://npmjs.org) or [crates](https://crates.io/). But the project that might come close to a similar experience is [platformIO](https://platformio.org/). It's a platform that provides you with an opinionated set of tools to bootstrap a project with all the tooling set up for you. There is some sort of package manger (called libraries), a project scaffolding tool, a tool to flash your boards and communicate with them and even a debugger! PlatformIO offers a flagship IDE, which is based on VSCode, but that's not why we're here. For vim we will need the [CLI](https://platformio.org/install/cli), whose installation directions you can read [here](https://docs.platformio.org/en/latest/core/installation.html). On macOS it's as easy as

```bash
brew install platformio
```

To scaffold a project structure with the tools prepared for (neo)vim development use the `project init` command:

```bash
mkdir disco-temp && cd disco-temp
platformio project init --ide vim --board disco_f303vc --project-option "framework=stm32cube"
```

This command sets up a new platformIO project with the `vim` ide settings for the stm32f3discovery board. If you wish to use the official CubeHAL framework make sure to add the `--project-option "framework=stm32cube"` flag which will pull the necessary files for you and set appropriate compiler flags.

You can then start adding your source files to the created `src` directory. I won't go into much detail about how to get started with platformIO in general as its [excellent documentation](https://docs.platformio.org/en/latest/core/index.html) already has got you covered.
