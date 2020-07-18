
---
layout: post
title: "Debugging Arm-based microcontrollers in (Neo)vim with gdb"
date: 2020-07-18
categories:
  - Programming
tags: [arm, vim, c++, debugging]
---

Coming from the JavaScript world, I'm used to the amazing debugging capabilities that browsers offer these days. I wanted to find a way to do debugging in a sane way, covering the basics like setting breakpoints, skipping through them and doing variable inspection at runtime. Ideally I wanted to use vim for that as this is where I do all of my coding! I ended up finding an elegant solution (at least in my opinion) that I would like to share with you.

**Heads up! This is a guide targeted at macOS specifically! Build, installation and configuration instructions might differ for other UNIX based systems and _especially_ for windows**

**Psst**: you also might want to check out the blog post I have written on how to set up a (Neo)vim dev environment for C(++) with all bells and whistles: [Modern C++ development in (Neo)vim](/post/2020/07/17/modern-c-development-in-neovim/)

## See it in action!

[![asciicast](https://asciinema.org/a/u6JReRp4qOqEjXVzCUTY0ETXV.svg)](https://asciinema.org/a/u6JReRp4qOqEjXVzCUTY0ETXV)

In my examples I will sometimes refer to the STM32 libraries and the [STM32F3DISCOVERY](https://www.st.com/en/evaluation-tools/stm32f3discovery.html) board as that's what I'm using to test all this.

## Installing `gdb`

We will need the [GNU Arm Embedded Toolchain](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads) which includes the Arm version of `gbd` (`arm-none-eabi-gdb`). You can easily install it via homebrew:

```shell
brew cask install gcc-arm-embedded`
```

**Note on macOS Catalina**: Catalina is blocking the `arm-none-eabi-gdb`
executable by default. Run it once in your console, then open Security & Privacy system settings, then, in the general tab allow gcc-arm-none-eabi to run.

## Installing a gdb-server

The `gdb`-server is the tool that actually connects to your microcontroller programmer via USB. For STM32 MCUs there are two options: [stlink](https://github.com/stlink-org/stlink/blob/develop/doc/tutorial.md#using-the-gdb-server-1) (`st-util`) or [OpenOCD](http://openocd.org/). I will focus on OpenOCD here as this seems to be working much better for me and supports way more devices.

I highly recommend building openOCD from source as the homebrew formulas seem to
be outdated. This is how I went about it:

```shell
brew install autoconf automake texinfo
git clone https://git.code.sf.net/p/openocd/code openocd-code
cd openocd-code
./bootstrap
./configure
make
sudo make install
```

See also [OpenOCD - Open On-Chip Debugger / Code](https://sourceforge.net/p/openocd/code/ci/master/tree/)

Connect your board and run

```shell
openocd -f board/stm32f4discovery.cfg # replace with your corresponding board / interface
```

to see if it's working. If it looks something like this, we're in business!

```txt
Info : STLINK V2J27M15 (API v2) VID:PID 0483:374B
Info : Target voltage: 2.906461
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f4x.cpu on 3333
Info : Listening on port 3333 for gdb connections
```

You can also specify a custom interface and/or mcu target:

```shell
openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

See [here](https://sourceforge.net/p/openocd/code/ci/master/tree/tcl/) for all
supported devices and targets.

## Configuring your project for gdb

To connect our project to `gdb` we need to configure a few things within our
project directory. I am using a [platformIO]() project as an example but if you
know how to build your project in debug mode (`-g`) this shouldn't be a problem.

Create an `.gdbinit` file in the project directory. This will be executed every
time the `gdb` client connects to the server:

```bash
file .pio/build/disco_f303vc/firmware.elf # point this to your firmware file compiled with the --debug (platformIO) or -g flag
target remote localhost:3333 # points gdb to our OpenOCD server
load # loads all the debug symbols
```

To start debugging you'd compile the project with the debug flag, then start the
OpenOCD server and keep it running in the background. I created a Makefile entry
for this, for convenience:

```makefile
OPENOCD := /usr/local/bin/openocd

# ...

debug:
	platformio -f -c vim run --target debug && $(OPENOCD) -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

## Using the (Neo)vim debugger

The debugger solution that we are going to use is actually baked into vim (from
v8.1) and Neovim. It's called `termdebug`. See [here](https://www.dannyadam.com/blog/2019/05/debugging-in-vim/) for a more in-depth introduction and [here](https://vimhelp.org/terminal.txt.html#terminal-debug) for the official manual.

The `termdebug` plugin is disabled by default, so we have to enable it in the
`.vimrc` / `init.vim`:

```vim
packadd termdebug
```

We are making further adjustments to point the `termdebug` plugin to the arm
`gdb`:

```vim
" Default is ARM debugging
" In the future, consider using https://github.com/embear/vim-localvimrc for
" project specific settings
let g:termdebugger = "arm-none-eabi-gdb"
```

Then, if you like, add a few goodies for a nice split window view and some keymappings:

```vim
" C(++) debugging
" See https://neovim.io/doc/user/nvim_terminal_emulator.html
" For a nice split window view
let g:termdebug_popup = 0
let g:termdebug_wide = 163
" Map ESC to exit terminal mode
tnoremap <Esc> <C-\><C-n>
nnoremap <silent> <leader>b :Break<CR>
nnoremap <silent> <leader>bc :Clear<CR>
nnoremap <silent> <leader>c :Continue<CR>
```

### Usage

1) Run `make debug` in another console window
2)  Open file to debug, then `:Termdebug`
3) Set and remove breakpoints, see evaluated values, manage control flow with commands below

**Shortcuts**
* leader, b to set breakpoint on line under cursor
* leader, b, c to remove that breakpoint
* leader, c to continue code execution
* Shift + k for evaluation

See [the manual](https://vimhelp.org/terminal.txt.html#terminal-debug) for all the commands that are possible with the debugging terminal emulator


### Supercharged `termdebugger`

When in the debugger, normally the window in the bottom left would show the
program output. As this is not the case for remote debugging I thought I could
try to make better use of that and maybe even show the uart output of the
device, if there is any. For that I had to fork the `termdebugger` plugin and
pour it into its own vim-plug compatible one.

To install add

```vim
Plug 'chmanie/termdebugx.nvim'
```

and remove the `packadd` line.


Then you are able to define the command that will be executed in the `program`
window of the debugger (in my case the `platformio device monitor` command):

```vim
let g:termdebugger_program = "pio device monitor -b 38400" " only works with termdebugx.nvim
```

## That's it!

I hope I haven't missed anything. If you're having problems setting this up,
feel free to reach out on one of the channels below.

## References

Some links I stumbled upon during gathering the information in no particular
order.

* [GNU Toolchain | GNU Arm Embedded Toolchain Downloads – Arm Developer](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)
* [Debugging with GDB: Top](https://sourceware.org/gdb/onlinedocs/gdb/)
* [GitHub - ntfreak/openocd: Spen’s Official OpenOCD Read-Only Mirror (no pull requests)](https://github.com/ntfreak/openocd)
* [GitHub - embear/vim-localvimrc: Search local vimrc files (“.lvimrc”) in the tree (root dir up to current dir) and load them.](https://github.com/embear/vim-localvimrc)
* [gdb(1): GNU Debugger - Linux man page](https://linux.die.net/man/1/gdb)
* [neovim/termdebug.vim at master · neovim/neovim · GitHub](https://github.com/neovim/neovim/blob/master/runtime/pack/dist/opt/termdebug/plugin/termdebug.vim)
* [Nvim documentation: nvim_terminal_emulator](https://neovim.io/doc/user/nvim_terminal_emulator.html)
* [OpenOCD User’s Guide: GDB and OpenOCD](http://openocd.org/doc/html/GDB-and-OpenOCD.html)
* http://openocd.org/doc/pdf/openocd.pdf
* https://docs.platformio.org/en/latest/projectconf/build_configurations.html
* [cgdb](https://cgdb.github.io/)
* https://www.reddit.com/r/neovim/comments/9myvqx/neovim_debugger/
* https://gist.github.com/joegoggins/7763637
* [stlink — Homebrew Formulae](https://formulae.brew.sh/formula/stlink)
* [Debugging with GDB on STM32 — Dev  documentation](https://ardupilot.org/dev/docs/debugging-with-gdb-on-stm32.html)
* [GitHub - stlink-org/stlink: Open source STM32 MCU programming toolset](https://github.com/stlink-org/stlink)
* [ST-LINK on-board | SEGGER - The Embedded Experts](https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/?utm_source=platformio&utm_medium=docs#compatible-evaluation-boards)
