---
layout: page
title: Vim
---

# VIM

For a set of plugins that makes a lot of sense, see <https://www.youtube.com/watch?v=wlR5gYd6um0>.

## ToDo

[ ] check out the exchange.vim plugin
[ ] learn about expression register
[ ] stop using dw etc and start using diw and daw
[ ] stop using esc and start using ctrl-[

## Build

* <https://github.com/Valloric/YouCompleteMe/wiki/Building-Vim-from-source>
* flags: <http://vimdoc.sourceforge.net/htmldoc/various.html>

For fedora:
```
sudo yum-builddep vim-X11
```

To build vim with xterm_clipboard enabled and python so that YouCompleteMe works:
```
git clone https://github.com/vim/vim.git
cd vim
./configure --with-features=huge \
            --enable-multibyte \
            --enable-pythoninterp=yes \
            --enable-python3interp=yes \
            --with-x \
make
sudo make install
```

## Shortcuts/commands
* S( | when in visual mode, sound selection with (
* cs[( | with cursor on [, change surround from [ to (
* g_ | go to last non-blank char. like $ but stops with cursor on last char
* i_ctrl-a | insert last inserted text
* i_ctrl-o | drop to normal mode for 1 command
* i_ctrl-h | backspace
* i_ctrl-u | erase to beginning of line
* center the current line | zz
* scroll up or down 1 line | ctrl-e / ctrl-y
* force filetype to xml | :set ft=xml
* inc/dec number | ctrl-a / ctrl-x
* jump to last change | '.
* jump to previous/next change | g; / g,
* reselect block | gv
* select inserted block | ,gV
* diff two buffers in open windows | :windo diffthis
* convert current buffer to unix line endings | :setlocal ff=unix
* open a new file in a new vertical split window | :vnew filename
* navigate quickfix (vimgrep output)| [q and ]q
* navigate diff | [c and ]c
* select next match | gn
* toggle folding | zi
* toggle current fold |  za
* toggle current fold recursively |  zA
* close current or parent fold | zc
* open fold to reveal cursor | zv
* toggle markdown fold method between hierarchical and show all headers | :FoldToggle
* read a file and insert it into the current buffer | :r <filename>

## Regex
Regex reference: <http://www.vimregex.com/>

* very magic | \v
* entire match | \0
* nth capture group | \n
* match EOL | \n
* substitute CR | \r
* whitespace char | \s
* none of a, b, or c | [^abc]
* like * but non-greedy | \{-}
* substitute entire group | \0
* substitute nth match group | \n

## Populate args list

    :args **/*.java

## Search and Replace in Multiple Files

    :argdo %s/foo/bar/ge | update

The `e` is to suppress errors for when the string doesn't exist in a file.
The `update` is to save the file after changing it.

## Pretty Print JSON

    :%!python -m json.tool

## Digraphs

To list:

    :dig
    :digraphs

To insert: <Ctrl-K>nn

* ™ | TM
* © | Co
* ® | Rg
* ✓ | OK
* ✗ | XX
* ° | DG
* á | a'
* ¿ | ?I
* ² | 2S
* ₂ | 2s
* · | .M
* ∞ | 00
* ≠ | !=

See also: http://ricostacruz.com/cheatsheets/vim-digraphs.html

## Pathogen

Add a plugin:

    $ cd bundle
    $ git submodule add <plugin-repo-url>
    $ git commit -m 'Add <plugin>'

Update a plugin:

    $ git pull origin master

Update all plugins:

    $ git submodule foreach git pull origin master

Get the update on another machine:

    $ git submodule init
    $ git submodule update

Remove a plugin:

    $ git submodule deinit bundle/vim-rvm
    $ git rm bundle/vim-rvm
    $ git rm --cached bundle/vim-rvm
    $ rm -rf .git/modules/bundle/vim-rvm

## Fugitive

If diff is not working, there are likely some hidden buffers with diff turned on.
Do this:

    :bufdo diffoff

## Recent Files

Use `:browse old` to see a list of recent files.
Use `'0` to return to the most recent file, then `'1` etc.

## Snippets

For a cheap snippet, `,sfoo` will insert foo.txt at the cursor position from normal mode:

    nnoremap ,sfoo :-1read $HOME/.vim/snips/foo.txt<CR>

where `,s` stands for "snippet".
