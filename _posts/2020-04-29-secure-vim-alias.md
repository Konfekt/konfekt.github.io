---
layout: post
title: "Traceless Vim Editing"
date: 2020-04-29
categories: gpg secure secret encrypt leak traces private vim
comments: true
---

A file `secret.txt` to be encrypted better be edited without leaving traces such as its path, swap files, backup or undo data.
To achieve this in Vim from the shell (say `Bash` or `ZSH`), open it by the command `vims secret.txt` using the shell alias

```sh
vims=" vim -n -i NONE \
  +'set backupskip=*' \
  +'augroup Secure | autocmd BufWrite * setlocal noundofile | augroup END' \
  +'silent doautoall Secure BufWrite"
alias vims="$vims"
unset vims
```

added to the configuration file (`~/.bashrc` respectively `~/.zshrc`).

---

If the secret file is a temporary file, for example, kept in `/tmp` on Linux, then the same can be achieved automatically, without any alias, using the following permanent `Vim` settings

```vim
"  don't store marks of temporary files in temp or shm directories
let &viminfo .=
      \ ',r' . (has('win32') ? $TMP : $TMPDIR) .
      \  (exists('$XDG_CACHE_HOME') ? ',r' . $XDG_CACHE_HOME : '') .
      \  (exists('$XDG_RUNTIME_DIR') ? ',r' . $XDG_RUNTIME_DIR : '') .
      \ ',r/var/tmp,r/dev/shm,r/run/shm,r/var/run/shm' .
" don't backup temporary files (such as those in temp or shm directories)
let &backupskip .= ',' .
  \ '/var/tmp/*,/dev/shm/*,/run/shm/*,/var/run/shm/*,' .
  \  (exists('$XDG_CACHE_HOME') ? escape(expand($XDG_CACHE_HOME), '\') . '/*,' : '') .
  \  (exists('$XDG_RUNTIME_DIR') ? escape(expand($XDG_RUNTIME_DIR), '\') . '/*,' : '') .
  \ '*~,COMMIT_EDITMSG,MERGE_MSG'
" don't keep swap file or undo data of temporary files
augroup vimrc_skip
  autocmd!
  exe 'autocmd BufNewFile,BufRead' &backupskip 'setlocal noswapfile'
  exe 'autocmd BufWrite' &backupskip ' setlocal noundofile'
augroup END
silent doautoall vimrc_skip BufNewFile,BufRead,BufWrite
```

added to the configuration file (`~/.vimrc` in Linux and MacOS or `%USERPROFILE%/_vimrc` in Microsoft Windows).

Otherwise, open the file in `Vim` by the command `:EditSecurely secret.txt` given by

```vim
command! -nargs=1 -complete=file_in_path EditSecurely set viminfofile=NONE | edit <args> | setlocal noswapfile nobackup noundofile
```

---

Once a file has been encrypted, say by [`Gnu Privacy Guard`](https://www.gnupg.org/), conveniently edit it using the [vim-gnupg](https://github.com/jamessan/vim-gnupg/) plug-in without leaving traces.

