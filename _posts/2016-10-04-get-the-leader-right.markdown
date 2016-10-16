---
layout: post
title: "Getting the leader right"
date: 2016-10-03
categories: vim plug-in vimrc
comments: true
---

**_TLDR_**: Swapping `:` and `,` makes key bindings more consistent and intuitive. [Vim-Aliases](https://github.com/konfekt/vim-alias) jump in to replace leader key mappings.

---

There are

- people mapping the leader key (which is `/` by default) to `,` and
- people mapping to `:` to `;`

However, inside a line, we maneuver most efficiently first looking up a character by `f`/`F` (or `t`/`T`) and then hemming it in by `;`/`,`.
Because either mapping cripples this, better **swap `:` with `,`**.

Then

- the back-and-forth motions `;`/`:` are not only consistent with `f`/`F`,`t`/`T`, `(g)n`/`N` and `/`/`?`,
- but also more intuitive and more easily reachable than `;`/`,`;
- hitting `,` instead of `:` to enter the command-line saves pinkie pain.

That the default leader key `/` is not as easily reachable as `,` is remedied by [**vim-aliases**](https://github.com/konfekt/vim-alias):
**Instead of various leader mappings**, commonly used for commands such as, say

```vim
    nnoremap <leader>gc :<c-u>!git commit<cr>
```

**a command-line alias jumps in**:

```vim
    Alias gc !git commit
```

Then

- the aliases offer more flexibility by letting you add parameters like `!git commit -a -p`.
Flexibility, which leader key mapping cannot achieve at all, for example,

```vim
    !git commit -m 'my commit message'
```

or which even a bunch of additional leader key mappings

```vim
    nnoremap <leader>gc :<c-u>!git commit<cr>
    nnoremap <leader>gca :<c-u>!git commit -a<cr>
    nnoremap <leader>gcp :<c-u>!git commit -p<cr>
    nnoremap <leader>gcap :<c-u>!git commit -a -p<cr>
```

can only achieve by side effects:
because `<leader>gcap` shadows both `<leader>gc` and `<leader>gca`, these latter mappings therefore must wait for a timeout;

- because `,` and aliases serve as a replacement for the commands usually mapped to `<leader>...`, the `<leader>` key is freed up, for example for random ad-hoc commands.

---

For consistency and less pinkie pain, add to `nnoremap : ,` the following mappings:

```vim
    nnoremap : ,
    xnoremap : ,
    onoremap : ,

    nnoremap , :
    xnoremap , :
    onoremap , :

    nnoremap g: g,
    nnoremap g, <NOP>

    nnoremap @, @:
    nnoremap @: <NOP>

    " NOTE: Causes lag when 'q' is hit, for example
    " - after recording a macro or
    " - existing a buffer by a custom mapping to 'q'.
    nnoremap q, q:
    nnoremap q: <NOP>
```

