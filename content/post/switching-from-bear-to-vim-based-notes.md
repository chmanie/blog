
---
layout: post
title: "Switching from Bear to vim-based notes"
date: 2020-08-28
categories:
  - Meta
tags: [vim, note-taking]
---

It might not come as a surprise to you that I’m a heavy [vim](/post/2020/07/17/modern-c-development-in-neovim/) [user](/post/2020/07/18/debugging-arm-based-microcontrollers-in-neovim-with-gdb/). Whenever I write anything, I’d like for it to be inside of vim. This has been true for almost anything _except_ note-taking. Why? Because I like to take a lot of notes on the go (in fact, I started the draft of this blog post on my phone) and sync them with my laptop. The number of iOS-apps that play well with vim based solutions _and_ have pleasant use interface is quite small.

That’s why I’ve been using [Bear](https://bear.app/) to take notes in the past years. It works wonderfully and is a fantastic app for this use case (and the mobile app is almost perfect!). But... it’s not vim!

So I’ve been on the lookout for alternatives that would ideally tick all these boxes:

- [x] Be compatible with vimwiki
- [x] Sync with a mobile app that has an acceptable UI
- [x] The ability to attach images/screenshots to my notes

That doesn't seem like a lot to ask but I haven't found a good solution. Until now. Using certain vim-plugins and the right iOS app with a few adjustments we can create the exact experience I was striving for. Let me tell you how I did it.

## The mobile app

[1Writer](https://1writerapp.com/) has been around for ages in the App-Store and still keeps improving. The last update (at the time of writing this) is just 2 weeks old. It is amazingly powerful and provides (among others) these features that are important to me:

- A beautiful and useful interface
- Note syncing via a bunch of services
- Cross-note links
- The ability to upload images
- Tags
- A programming API (!)

Read on if you like to know how I configured it to play nicely with markdown-based notes in vim.

## The vim parts

When using Bear my notes haven't really been connected with each other and it was fine for me that way. However, I really got to like [vimwiki](https://github.com/vimwiki/vimwiki) which follows a more structured approach for personal note-taking. The idea of logically structuring the notes and making them accessible in a way that could also be useful for others seemed very appealing to me. So I gave it a try and am now in the process of restructuring parts of my notes as a wiki. vimwiki also greatly enhances markdown files in vim to be treated as notes.

For all of the other, unstructured notes I am using [notational-fzf-vim](https://github.com/Alok/notational-fzf-vim) (I actually was a [Notational Velocity](http://notational.net/) user before switching to Bear). It provides an unobtrusive and fast way to access notes via a full-text search and easily lets you create new ones.

Set them up to your own liking, I don't want to go into detail here on how to set them up to work for your personal workflow (in fact, I'm kind of assuming that you already have figured out a workflow in vim).

I also want to mention another plugin here that is making my note-taking life easier which is [md-img-paste.vim](https://github.com/ferrine/md-img-paste.vim). It allows you to use whatever picture is in your system's clipboard and paste it into a markdown file in vim. This will copy the file to the folder you defined (a subfolder of the file's directory) and create an appropriate markdown link to it - super useful! I set it up to copy the images to the `img` subfolder whenever I press leader-p:

```vim
nmap <buffer><silent> <leader>p :call mdip#MarkdownClipboardImage()<CR>
let g:mdip_imgdir = 'img'
```

### tl;dr:

```vim
Plug 'vimwiki/vimwiki'
Plug 'Alok/notational-fzf-vim'
Plug 'ferrine/md-img-paste.vim'
```

## The glue

Alright, let's sync up the two parts, shall we?

I configured my main vimwiki folder to be synced with [Nextcloud](https://nextcloud.com/), but you can also use Dropbox, iCloud or any other WebDAV-compatible service.

It looks like this:

```vim
let g:vimwiki_list = [{'path': '~/Nextcloud/Notes',
                      \ 'syntax': 'markdown', 'ext': '.md',
                      \ 'index': 'Wiki'}]
let g:vimwiki_global_ext = 0
```

This configures vimwiki to live in the synced `Notes` folder, sets its main syntax to markdown and lets it create `.md` files instead of `.wiki` files. This makes it possible for 1Writer to parse them properly. The index file is set to `Wiki.md` but you can use any filename you desire here. Furthermore I am disabling global wikis for vimwiki to not interfere with my usual markdown writing workflow. You don't have to do that.

On the notational-fzf-vim side my config looks like this:

```vim
let g:nv_search_paths = ['~/Nextcloud/Notes']
nnoremap <leader>e :NV<CR>
```

I'm telling it to look for notes only in the synced vimwiki folder and access it using leader-e (a fairly random choice).

You can now create linked notes (using vimwiki) or unlinked notes using notational-fzf-vim that are synced via the service of your choice. Nothing special so far, right?

Let's head over to 1Writer. Add the synced folder to it using the service of your choice and wait for the notes to be synced. The folder should be pinned to the left side so that you can quickly access your notes. You should be able to preview and edit the notes you've created in vim in 1Writer's beautiful interface.

But wait, we're not done yet!

There are a few adjustments that can further improve your mobile note-taking experience. For example you can "favourite" the `Wiki.md` by long-pressing or swiping to have fast access to it. I also did that with a file called `To Do.md` :)
You probably also want to make the synced folder the default folder in the 1Writer settings to always create new notes in that one.

### Creating linked wiki-documents in 1Writer
Linking between wiki-documents should work out of the box with 1Writer. But there's one more use case. What if you want to create a new wiki page that is linked to from the current document (like you would do in vimwiki)? 1Writer's JavaScript API has got you covered!

You can add the following script to your actions (tap the three dots in the bottom-left of any note and press the + button):

```js
ui.input('Note name', '', 'Give it a fancy name', name => {
  const [start, end] = editor.getSelectedRange();
  editor.replaceTextInRange(start, end, `[${name}](${name})`);
  editor.newFile('', name);
});
```

Give it an appropriate name and symbol to your liking, then save it ("close").

Now, from any note you can press the three dots again, tap this action, type in a name (e.g. "My new note") and it will create a vimwiki type reference to a newly created document:

```markdown
[My new note](My new note)
```

The filename will be `My new note.md`. You can adjust this as well in the script above.

## Importing from Bear

Let's assume we're comfortable with this setup and want to migrate all of our notes from Bear. That's actually very easy! Select all of your notes (cmd+A) and select "Export notes" from the "File" menu. Now select the folder you created earlier in this process and all of your notes will be exported there (you can also choose to export images which will create subfolders for every image). Personally I like to start from a clean slate here, so I put all of the notes I exported directly into an "archive" folder. I'm moving notes out of there as I go.

Alright! I hope I could tell you some new things in this post. Let me know if this workflow works for you and especially if you have suggestions to improve it.
