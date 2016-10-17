---
layout: post
title: "Vim: leave the leader"
date: 2016-10-03
categories: vim plug-in vimrc
comments: true
---

**_TL;DR_**:
Swapping `:` and `,` makes key bindings more intuitive and ergonomic.
[Vim-Aliases](https://github.com/konfekt/vim-alias) step in for `<leader>` key mappings.

---

There are

- people mapping the command-line key `:` to `;` (to ease pinkie pain) and
- people mapping the `<leader>` key `/` to `,` (to be better reachable).

However, inside a line, we maneuver most efficiently by first [looking up a character](https://github.com/unblevable/quick-scope) by `f`/`F` (or `t`/`T`) and then hemming it in by `;`/`,`.

Because either mapping cripples this, better **swap `:` and `,`** by

```vim
nnoremap : ,
nnoremap , :
```

This way, not only is pinkie pain eased, but also the back-and-forth motions `;`/`:` are consistent with `f`/`F`,`t`/`T`, `(g)n`/`N` and `/`/`?`, and therefore more intuitive than `;`/`,`.

Then instead of **various leader mappings**, commonly used for commands such as, say

```vim
nnoremap <leader>gc :<c-u>!git commit<cr>
```

let a **[command-line alias](https://github.com/konfekt/vim-alias)** step in:

```vim
Alias gc !git\ commit
```

This way, the former `<leader>` key commands are better reachable (by now hitting `,` to enter command-line) and what's more:
the alias offers more flexibility than its corresponding leader key mapping by letting you add parameters like `!git commit -a`.
Flexibility, which

- a leader key mapping cannot achieve at all, for example adding the parameters

```vim
    !git commit -m 'my commit message'
```

- or which even additional leader key mappings

```vim
    nnoremap <leader>gc :<c-u>!git commit<cr>
    nnoremap <leader>gca :<c-u>!git commit -a<cr>
```

can only achieve by side effects:
because `<leader>gca` shadows `<leader>gc`, it must wait for a timeout to be fired.

Finally, because `,` and command-line aliases replace the former `<leader>` key commands, the `<leader>` key is freed up;
for example, for random ad-hoc commands.

---

For consistency and less pinkie pain, add to `nnoremap : ,` and `nnoremap , :` the following mappings:

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

" NOTE: Causes lag when 'q' is hit, for example when
" - stopping to record a macro or
" - exiting a buffer by a custom mapping to 'q'.
nnoremap q, q:
xnoremap q, q:

nnoremap q: <NOP>
xnoremap q: <NOP>
```

