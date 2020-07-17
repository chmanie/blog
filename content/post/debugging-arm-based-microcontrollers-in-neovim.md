
---
layout: post
draft: true
title: "Debugging ARM-based microcontrollers in (Neo)vim"
date: 2020-07-07
categories:
  - Programming
tags: [arm, vim, c++, debugging]
---

Coming from the JavaScript world, I'm used to the amazing debugging capabilities that browsers offer these days. I wanted to find a way to do debugging in a sane way, covering the basics like setting breakpoints, skipping through them and doing variable inspection at runtime. Ideally I wanted to use vim for that as this is where I do all of my coding! I ended up finding an elegant solution (at least in my opinion) that I would like to share with you.

**Heads up! This is a guide targeted at macOS specifically! Build, installation and configuration instructions might differ for other UNIX based systems and _especially_ for windows**


development for ARM based microcontrollers. In my examples I will sometimes refer to the STM32 libraries and the [STM32F3DISCOVERY](https://www.st.com/en/evaluation-tools/stm32f3discovery.html) board as that's what I'm using to test all this.


## Debugging with `gdb`
### Installing arm-gdb
([GNU Toolchain | GNU Arm Embedded Toolchain Downloads – Arm Developer](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads))

Install `gdb` (`arm-none-eabi-gdb`)

`brew cask install gcc-arm-embedded`

macOS Catalina
open Security & Privacy, in the general tab allow gcc-arm-none-eabi to run

### Installing a gdb-server (which connects to st-link via USB)

#### `stlink`

`brew install stlink`

[stlink/tutorial.md at develop · stlink-org/stlink · GitHub](https://github.com/stlink-org/stlink/blob/develop/doc/tutorial.md#using-the-gdb-server-1)

I had some problems with `st-link` , for example breakpoints not being recognised and generally being quite unstable

Run `st-util`

#### `open-ocd`

Build from source:

```shell
brew install autoconf automake texinfo
git clone https://git.code.sf.net/p/openocd/code openocd-code
cd openocd-code
./bootstrap
./configure
make
sudo make install
```

See [OpenOCD - Open On-Chip Debugger / Code](https://sourceforge.net/p/openocd/code/ci/master/tree/)

Run `openocd -f board/stm32f4discovery.cfg` (replace with your corresponding board / interface)

But this did (for the stm32f3discovery):

`openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg`

### `gdb config`

Create an `.gdbinit` file in the project directory:

```bash
file .pio/build/disco_f303vc/firmware.elf # point this to your firmware file compiled with the --debug flag
target remote localhost:3333 # :4242 for st-link
load
```

### Compile with debug mode

```shell
platformio -f -c vim run --target debug
```


### Makefile entry

```makefile
OPENOCD := /usr/local/bin/openocd

# ...

debug:
	platformio -f -c vim run --target debug && $(OPENOCD) -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

### Vim config

Using vim-plug

```vim
Plug 'chmanie/termdebugx.nvim'
```

Or

```vim
packadd termdebug
```

for a less feature-rich version 

```vim
" C(++) debugging
" See https://neovim.io/doc/user/nvim_terminal_emulator.html
" For a nice split window view
let g:termdebug_popup = 0
let g:termdebug_wide = 163
" Default is ARM debugging
" In the future, consider using https://github.com/embear/vim-localvimrc for
" project specific settings
let g:termdebugger = "arm-none-eabi-gdb"
let g:termdebugger_program = "pio device monitor -b 38400" " only works with termdebugx.nvim
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
4) See the `pio device monitor` output in the program window 

**Shortcuts**
* leader,  b to set breakpoint on line under cursor
* leader, b, c to remove that breakpoint
* leader,  c to continue code execution
* Shift + k for Evaluation

See https://neovim.io/doc/user/nvim_terminal_emulator.html for all the commands that are possible with the debugging terminal emulator

### References

https://github.com/MaskRay/ccls/wiki/Build
[GNU Toolchain | GNU Arm Embedded Toolchain Downloads – Arm Developer](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)
[Debugging with GDB: Top](https://sourceware.org/gdb/onlinedocs/gdb/)
[GitHub - ntfreak/openocd: Spen’s Official OpenOCD Read-Only Mirror (no pull requests)](https://github.com/ntfreak/openocd)
[GitHub - embear/vim-localvimrc: Search local vimrc files (“.lvimrc”) in the tree (root dir up to current dir) and load them.](https://github.com/embear/vim-localvimrc)
[gdb(1): GNU Debugger - Linux man page](https://linux.die.net/man/1/gdb)
[neovim/termdebug.vim at master · neovim/neovim · GitHub](https://github.com/neovim/neovim/blob/master/runtime/pack/dist/opt/termdebug/plugin/termdebug.vim)
[Nvim documentation: nvim_terminal_emulator](https://neovim.io/doc/user/nvim_terminal_emulator.html)
[OpenOCD User’s Guide: GDB and OpenOCD](http://openocd.org/doc/html/GDB-and-OpenOCD.html)
http://openocd.org/doc/pdf/openocd.pdf
https://docs.platformio.org/en/latest/projectconf/build_configurations.html
[cgdb](https://cgdb.github.io/)
https://www.reddit.com/r/neovim/comments/9myvqx/neovim_debugger/
https://gist.github.com/joegoggins/7763637
[stlink — Homebrew Formulae](https://formulae.brew.sh/formula/stlink)
[Debugging with GDB on STM32 — Dev  documentation](https://ardupilot.org/dev/docs/debugging-with-gdb-on-stm32.html)
[GitHub - stlink-org/stlink: Open source STM32 MCU programming toolset](https://github.com/stlink-org/stlink)
[ST-LINK on-board | SEGGER - The Embedded Experts](https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/?utm_source=platformio&utm_medium=docs#compatible-evaluation-boards)
