---
layout: post
title: "What is a Zettelkasten and how to build one in Vim"
date: 2021-12-14
categories: Zettelkasten Vim
comments: true
---

Buzz for the [Zettelkasten](https://zettelkasten.de/) note taking system is on the rise, in which  all text files are gathered in a single folder, optionally linked to each other by their names, and a particular file is retrieved by searching for its content:
A **Zettelkasten** is a folder of assorted (text) files, called **zettels**, that (can) link to each other by their names.
Historically, the file folder is rather a card file box, whereas nowadays it is a directory to store (text) files (in Markdown format).

# Concept

Folders, be it of paper or digital data, are a classic concept to sort files.
However a file rarely fits exclusively into a single folder.
If we think of the folder as a tag, given by its name, then a folder attaches a single tag to a file;
however, usually multiple tags match a file.

Since nowadays searching for words through thousands of files is a matter of milliseconds, assigning multiple tags to a file and retrieving it by searching for them is at least as fast as storing it in (multiple) folders and searching for one of them.
Attaching tags (that is, writing them into the text file) is however more convenient than putting the same file into multiple folders, especially since these tags usually need not be explicitly added, but already appear in the text.
It is also closer to how our mind works, as it re(dis)covers a file from memory by recollecting assorted words of its content, rather than a single title or tag.

# Usage 

Instead of sorting files by folders, that is, attaching to each file a single tag, the name of the folder, write key words into each file (or let the content speak for itself) and search for them (by a text searcher such as `grep`, `rg` or `ag`).
Similar to how you search for one of your browser's bookmarks by its title in the address bar instead of looking for it among possible folders.

To ease the creation of zettels, we wish:

- to create a note with the given search terms, if none existed yet, 
- to insert a link to a note by searching for its content,
- to open a linked note by positioning the cursor on its link and then hitting a some key combination.

What follows is an implementation of such a Zettelkasten manager in [Vim](https://www.vim.org):

# Vim Implementation

Below a configuration of [notational-fzf-vim](https://github.com/alok/notational-fzf-vim) that comes with a command (`:NV`) to fuzzily find a note by its content.
The clou of [notational-fzf-vim](https://github.com/alok/notational-fzf-vim) (using below adaption of `NV_note_handler()`):
if no note is found (or a special key combination, say `'ctrl-x`, is hit), then one is created, with its title given by the search terms (whose separating blanks are replaced with hyphens).

The paths of the Zettelkasten (and additional folders of notes) are listed in `g:nv_search_paths`, that of the Zettelkasten coming first.
To jump to a note, type `gf` when the cursor is on the file name (made possible by including `g:nv_search_paths` in `&path` and setting `&suffixesadd`).

Finally, a link to a note can be inserted by hitting `ctrl-z` (customizable by `g:insert_note_key`) that searches for the term before the cursor, which can then be refined in a fuzzy searcher.

```vim
" set up search paths
let g:nv_search_paths  = []
let s:zettelkasten = $HOME . '/zettelkasten'
if !isdirectory(expand(s:zettelkasten))
  call mkdir(expand(s:zettelkasten), 'p')
endif
let g:nv_search_paths += [s:zettelkasten]
let g:nv_search_paths += [$HOME . '/diary']
let g:nv_search_paths += [$HOME . '/notes']
let g:nv_search_paths += [$HOME . '/Desktop']

if exists('+shellslash') && !&shellslash
  let g:nv_search_paths = map(copy(g:nv_search_paths), 'tr(v:val, "/", "\\")')
endif
let g:nv_main_directory = g:nv_search_paths[0]

" make gf jump to any file in g:nv_search_paths
let s:glob = ''
let s:opt  = ''
for path in g:nv_search_paths
	let path = resolve(path)
  if exists('+shellslash') && !&shellslash
	  let path = tr(path, "\\", "/")
  endif
  let s:glob .= path . '/*.md,'
  let s:opt  .= path . '/**,'
endfor


" add link to file by fuzzy search in insert mode; adapted from Father
" Robert's https://www.frrobert.com/blog/linkingzettelkasten-2020-05-11-0735
function! NV_make_note_link(l)
  let line = split(a:l[0], ':')
  let ztk_id = fnamemodify(l:line[0], ':~:.:r')
  try
    let ztk_title = substitute(l:line[2], '\#\+\s\+', '', 'g')
  catch
    let ztk_title = ztk_id
  endtry
  let mdlink = '[' . ztk_title .']('. ztk_id .')'
  return mdlink
endfunction
let s:search_paths = join(map(copy(g:nv_search_paths), 'shellescape(v:val)'))
function! s:complete_file()
  return fzf#vim#complete({
        \ 'source':  'rg --follow --smart-case --no-heading --line-number --color never --no-messages "" ' . s:search_paths,
        \ 'reducer': function('NV_make_note_link'),
        \ 'options': '--multi --reverse --margin 15%,0',
        \ 'up':    5})
endfunction
" }}}

let s:insert_note_key = get(g:, 'nv_insert_note_key', 'ctrl-z')
augroup nv
  autocmd!
  exe 'autocmd BufRead,BufNewFile ' . s:glob .
        \ ' setlocal path+=' . s:opt . ' | ' .
        \ ' inoremap <buffer><expr> ' . s:insert_note_key . ' <sid>complete_file()'
  autocmd FileType markdown setlocal suffixesadd+=.markdown,.mdown,.mkd,.mkdn,.mdwn,.md
augroup END

" custom handler function
let s:main_dir = g:nv_main_directory
let s:ext = get(g:, 'nv_default_extension', '.md')
let s:keymap = get(g:, 'nv_keymap',
      \ {'ctrl-s': 'split',
      \ 'ctrl-v': 'vertical split',
      \ 'ctrl-t': 'tabedit',
      \ })
let s:create_note_key = get(g:, 'nv_create_note_key', 'ctrl-x')
let s:yank_key = get(g:, 'nv_yank_key', 'ctrl-y')
" Separator for yanked files
let s:yank_separator = get(g:, 'nv_yank_separator', "\n")

function! s:yank_to_register(data)
  let @" = a:data
  silent! let @* = a:data
  silent! let @+ = a:data
endfunction

" adapted from
" https://github.com/aldur/notational-fzf-vim/blob/cb08be1899abc74eeb559cf98123bd75f1a92ce8/plugin/notational_fzf.vim#L158
function! NV_note_handler(lines) abort
  " exit if empty
  if a:lines == [] || a:lines == ['','','']
    return
  endif
  " Expect at least 2 elements, `query` and `keypress`, which may be empty
  " strings.
  let query    = a:lines[0]
  let keypress = a:lines[1]
  " `edit` is fallback in case something goes wrong
  let cmd = get(s:keymap, keypress, 'edit')
  " Preprocess candidates here. expect lines to have fmt
  " filename:linenum:content

  " Handle creating note.
  if keypress ==? s:create_note_key
    " replace blanks to more easily open file when positioning cursor on its name
    let filename = tr(query, ' ', '-')
    let candidates = [fnameescape(s:main_dir  . '/' . filename . s:ext)]
  elseif keypress ==? s:yank_key
    let pat = '\v(.{-}):\d+:'
    let hashes = join(filter(map(copy(a:lines[2:]), 'matchlist(v:val, pat)[1]'), 'len(v:val)'), s:yank_separator)
    return s:yank_to_register(hashes)
  else
    let filenames = a:lines[2:]
    let candidates = []
    if empty(filenames)
      " If there are no matches, then create a note
      let filename = tr(query, ' ', '-')
      let candidates = [fnameescape(s:main_dir  . '/' . filename . s:ext)]
    else
      for filename in filenames
        " Don't forget trailing space in replacement.
        let linenum = substitute(filename, '\v.{-}:(\d+):.*$', '+\1 ', '')
        let name = substitute(filename, '\v(.{-}):\d+:.*$', '\1', '')
        " fnameescape instead of shellescape because the file is consumed
        " by vim rather than the shell
        call add(candidates, linenum . fnameescape(name))
      endfor
    endif
  endif

  for candidate in candidates
    execute join([cmd, candidate])
  endfor

endfunction
```

