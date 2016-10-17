---
layout: post
title: "Vim: All-in-one Omni and file-path completion"
date: 2016-10-14
categories: vim vimrc
comments: true
---

The following Vim snippet makes `<Ctrl-Space>` (and `Ctrl-Shift-Space>`) complete code (`:help new-omni-completion`) and file paths (`:help compl-filename`) in insert mode.
A file path is recognized by a preceding `/` (respectively `\` in Microsoft Windows).
After having completed a directory name, hit `/` (respectively `\`) to complete the file names inside it.

```vim
if has('win32')
    function! s:isBehindPath()
      return getline('.') =~# '\v' . '(\\|\a:)\f*$'
    endfunction
    function! s:isBehindDir()
      return getline('.') =~# '\\$'
    endfunction
    inoremap <expr> \
          \ (pumvisible() && <SID>isBehindDir()) ?
          \ "\<C-y>\<C-x><C-f>" : "\\"
else
    function! s:isBehindPath()
      return getline('.') =~# '\/\f*$'
    endfunction
    function! s:isBehindDir()
      return getline('.') =~# '\/$'
    endfunction
    inoremap <expr> /
          \ (pumvisible() && <SID>isBehindDir()) ?
          \ "\<C-y>\<C-x><C-f>" : "/"
endif

inoremap <expr><silent> <C-SPACE>   pumvisible() ? "<C-n>" : (<SID>isBehindPath() ? "<C-X><C-F>" : "<C-X><C-O>")
inoremap <expr><silent> <C-S-SPACE>   pumvisible() ? "<C-p>" : (<SID>isBehindPath() ? "<C-X><C-F>" : "<C-X><C-O>")
```

These mappings couple well with syntax code-completion (`:help ft-syntax-omni`), enabled by

```vim
autocmd Filetype *
        \   if empty(&l:omnifunc) |
        \       setlocal omnifunc=syntaxcomplete#Complete |
        \   endif
```

---

The key combinations `<Ctrl-Space>` and `<Ctrl-Shift-Space>` are recognized by *GVim*, that is, if Vim is started in a GUI.
If Vim is started in terminal, then neither `<C-Space>` nor `<C-S-Space>` is recognized by default.

- To make *X-Term* recognize `Ctrl-Space`, add

```vim
  if has('unix') && !has('gui_running') && &term =~? '^xterm'
      inoremap <C-@> <C-Space>
  endif
```

to `~/.vimrc`.

- To make *URxvt* recognize `Ctrl-Space` and `Ctrl-Shift-Space`, add for example

```sh
  URxvt.keysym.C-space : \033Q32;5\~
  URxvt.keysym.C-S-space : \033Q32;6\~
```

to `~/.Xresources` and

```vim
  exe "set <F13>=\eQ32;5~"
  exe "set <F14>=\eQ32;6~"
  imap <F13> <C-Space>
  imap <F14> <C-S-Space>
```

to `~/.vimrc`.

Alternatively, rather than going through this trouble mapping to `<Ctrl-Space>`, replace it by a key combination recognized by every terminal, such as `<Ctrl-]>` or `<Ctrl-F>`.
