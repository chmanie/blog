+++
title = "Modern C++ development in (Neo)vim"
date = 2020-07-17

[taxonomies]
categories = ["Programming"]
tags = ["vim", "c++"]
+++

In this post I will try to document my journey of setting up a modern C(++) development environment in (neo)vim.

<!-- more -->

## tl;dr - What are we doing here?

We'll set up an IDE like C(++) dev environment in (neo)vim including:

- A Language Server Protocol implementation ([ccls](https://github.com/MaskRay/ccls)) and client ([coc.nvim](https://github.com/neoclide/coc.nvim))
- Syntax Highlighting using [vim-lsp-cxx-syntax-highlighting](https://github.com/jackguo380/vim-lsp-cxx-highlight)
- Linting via [cpplint](https://github.com/cpplint/cpplint) and [syntastic](https://github.com/vim-syntastic/syntastic)
- Formatting with [clang-format](https://clang.llvm.org/docs/ClangFormat.html)

## Tooling is hard.

It always was. All these things that you spend so.much.time. on even without having written a single piece of code towards your actual goal. But I like tooling. The power, the convenience of automation it provides _once it's set up properly_ can make many tasks quite joyful.
Coming from the ~~JavaScript~~ TypeScript world, I've been quite spoiled with helpful tools lately. 

There's the [TypeScript language server](https://github.com/sourcegraph/javascript-typescript-langserver), providing you with fantastic insights about your code and tools to refactor it. Then there's [eslint](https://eslint.org/) which, leveraging its great community has evolved into the de-facto standard of JavaScript code-style-checkers (if I can even call it that still). Honestly, when I start writing a new programming language, I check what kind of tools are out there to make my life easier in that realm. I felt I learned a lot from the seemingly arbitrary error messages that these tools provided, making my code style prettier and my implementations much more resilient.

Also, I *love* (neo)vim. Yes, you can (and have to!) spend a huge amount of time to [bend it to your liking](https://github.com/chmanie/dotfiles/blob/master/.config/nvim/init.vim). But I feel like once you and vim got acquainted and to like each other, it's a bond for life. I won't write more about my relationship with vim. As you're reading this I assume you have made up your mind already.

So when I started getting into C(++) development on microcontrollers, I naturally asked myself: "What's out there?". The answer lead me down a dark and extensive rabbit-hole of joy and frustration. Quite a couple of times I just felt so tempted to "just use VSCode" as many people suggested. But I don't want to. As of this writing I'm still using vim, hoping to continue this endeavor. In the following text I'm aiming to share my findings with you, not aiming for completeness or trying to cover all the use-cases. I'll try to keep it as general as possible though. 

**Heads up! This is a guide targeted at macOS specifically! Build, installation and configuration instructions might differ for other UNIX based systems and _especially_ for windows**

## Language Server Protocol

The Language Server Protocol is a protocol spearheaded by Microsoft which is trying to standardise editor <-> programming language communication and contextualisation.

To make use of that in vim, you can choose between a few plugins that are actively maintained and developed. Options include [vim-lsp](https://github.com/prabirshrestha/vim-lsp), [ALE](https://github.com/dense-analysis/ale) and others. The most feature-rich of all seems to be [coc.nvim](https://github.com/neoclide/coc.nvim) which is what I went for. For installation instructions please refer to the readmes of the corresponding project. The following instructions should apply for all the various LSP plugins though.

### coc.nvim

The installation of coc.nvim is not super-straightforward as it doesn't make any assumptions about your vim config (especially key-bindings), which I quite like. But that means that you have to do all the configuration yourself.

You can install it via [vim-plug](https://github.com/junegunn/vim-plug):

```vim
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

**This is by no means everything you need to do here! Please refer to the [example vim configuration](https://github.com/neoclide/coc.nvim#example-vim-configuration) to see how it is set up.**

### ccls

Now that we have the LSP client (the vim plugin) set up we have to pick an actual language server implementation for C(++). For that I found [ccls] to be the best choice as it is well maintained and documented and supported by platformIO by default (when using the `vim` IDE setting).

I built the current release of ccls as described here:
https://github.com/MaskRay/ccls/wiki/Build

```bash
brew install llvm # install llvm first

git clone --depth=1 --recursive https://github.com/MaskRay/ccls
cd ccls
brew info llvm
cmake -H. -BRelease -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/usr/local/Cellar/llvm/<Your Version>/lib/cmake
cmake --build Release
```

and installed it

```bash
cmake --build Release --target install
```

The version on homebrew is probably outdated, if you want to be on the safe side, build it from source

#### `ccls` configuration

In order to configure `ccls` for your project you will need to add a `.ccls` file to your project which contains all the environment variables and compiler flags. Read more about it [here](https://github.com/MaskRay/ccls/wiki/Project-Setup#ccls-file).

A very simple `.ccls` file might look like this:

```bash
clang

# add this to support `.h` files as C++ headers
%h -x
%h c++-header
```

I wanted my header files to just have the `.h` file ending (instead of .hh or .hpp). For that to work you'd have to add the `%h -x` and `%h c++-header` directives as you see above.

#### coc.nvim settings

To make `ccls` work with `coc.nvim` you have to make some changes to the `coc-settings.json` in your neovim config directory (mine is `~/.config/nvim`):

```js
  "languageserver": {
    "ccls": {
      "command": "ccls",
      "args": ["--log-file=/tmp/ccls.log", "-v=1"],
      "filetypes": ["c", "cc", "cpp", "c++", "objc", "objcpp"],
      "rootPatterns": [".ccls", "compile_commands.json"],
      "initializationOptions": {
         "cache": {
           "directory": "/tmp/ccls"
         },
         "client": {
          "snippetSupport": true
         }
       }
    }
  }

```

This configuration has worked quite well for me so far.

### Syntax Highlighting

What does syntax highlighting have to do with the LSP? Well, hear me out. As it seems, parsing and making sense of C++ code is hard. vim provides rudimentary syntax highlighting, and there are plugins to improve the situation but it's still far from what you might be used to from other programming languages. It turns out that we already _have_ a tool that can (and has to) understand C(++) code at a fundamental level: the LSP server. So there are vim syntax-highlighting plugins that use this fact and provide much nicer highlighting capabilities. I am using [vim-lsp-cxx-highlighting](https://github.com/jackguo380/vim-lsp-cxx-highlight) and so far quite like it.

Install it via `vim-plug` for example

```vim
Plug 'jackguo380/vim-lsp-cxx-highlight'
```

To make it work with `coc.nvim` and `ccls` I added the following options to the `coc-settings.json`:

```js
"initializationOptions": {
  // ...
  // This will re-index the file on buffer change which is definitely a performance hit. See if it works for you
  "index": {
    "onChange": true
  },
  // This is mandatory!
  "highlight": { "lsRanges" : true }
}
```

I also added these lines to my vim config to highlight even more features!

```vim
" c++ syntax highlighting
let g:cpp_class_scope_highlight = 1
let g:cpp_member_variable_highlight = 1
let g:cpp_class_decl_highlight = 1
```

The drawback of the syntax highlighting depending on the LSP server is that it is much slower than other solutions. So you're really trading beauty for performance here. I will see whether this will stick in the long run.

### Linting

Having proper linting in place can help tremendously to adhere to good coding standards and established best practises. On the C++ side I only know of the [cpplint](https://github.com/cpplint/cpplint) tool initially developed by Google.

To install the command line tool do

```bash
pip install cpplint
```

`cpplint` offers a few configuration options (which I have never used tbh, but hey, they're there!) that you can set in a `CPPLINT.cfg` which resides in the project root. See all configuration options [here](https://github.com/cpplint/cpplint/blob/develop/cpplint.py#L285).

If you're using ALE you just have to add it to the list of linters for C(++) files. coc.nvim sadly does not support it directly. For that we're using an additional vim plugin that has been around for ages!

```vim
Plug 'vim-syntastic/syntastic'
```

With a few more lines we can configure it to use `cpplint`:


```vim
let g:syntastic_cpp_checkers = ['cpplint']
let g:syntastic_c_checkers = ['cpplint']
let g:syntastic_cpp_cpplint_exec = 'cpplint'
" The following two lines are optional. Configure it to your liking!
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
```

### Auto-Formatting

Remember when you installed `llvm` earlier? If not, then now might be the perfect time to do so! It comes with a handful of wonderful tools that we can utilise for our quest.

```bash
brew install llvm
```

`llvm` specifically comes with the `clang-format` command line tool that can be used to auto-format a C(++) file according to pre-defined rules. These rules can be defined on a per-project basis by putting a `.clang-format` config-file in the root of your project. This how my file looks like for most of my projects:

```yaml
# https://clang.llvm.org/docs/ClangFormatStyleOptions.html
# To disable for a line use `// clang-format off`
BasedOnStyle: Google # https://google.github.io/styleguide/cppguide.html
IndentPPDirectives: BeforeHash
```

Keep in mind that you want to have this configuration very close to your `cpplint` configuration. Otherwise the two tools might fight against each other which would be... inconvenient.

Great, so we have the command-line set up for this, but what we want is the ability to format a buffer in vim using a handy shortcut. Well, there's a plugin for that!

```vim
Plug 'rhysd/vim-clang-format'
```

Using `:ClangFormat` command you can format a file according to the rules set in your `.clang-format` file. I mapped `<leader>f` to do the formatting:

```vim
nnoremap <Leader>f :<C-u>ClangFormat<CR>
```

To keep your vim config file clean you might want to do that in an ftplugin file. See mine [here](https://github.com/chmanie/dotfiles/blob/master/.config/nvim/ftplugin/cpp.vim).

## Epilogue

That's about it! I think I will keep updating this post as my dev environment changes over time. Be sure to check out my [neovim config file](https://github.com/chmanie/dotfiles/blob/master/.config/nvim/init.vim) for inspiration and context!
