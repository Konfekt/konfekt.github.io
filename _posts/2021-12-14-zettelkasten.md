---
layout: post
title: "What is a Zettelkasten and how to build one in Vim"
date: 2021-12-14
categories: Zettelkasten Vim
comments: true
---

Buzz around the [Zettelkasten](https://zettelkasten.de/) note-taking method is on the rise, in which  all text files are gathered in a single folder and can be interlinked by name.
Files are accessed by searching their content.

# Zettelkasten: Embracing Tags Over Folders

A **Zettelkasten** consists of assorted text files, or **zettels**, which may link to each other by name.
Traditionally, these were stored in a card file box; today, they are kept in a digital directory, often in Markdown format.

Folders, whether paper-based or digital, traditionally organize files.
However, since our computers (with tools like `(rip)grep`) search across the contents of thousands of files in milliseconds, a folder tree to organize files is no longer required:

If we think of the folder as a category tag, then a folder attaches a single tag to a file;
yet, a file often pertains to more than one category.
Tagging, which integrates keywords directly into the text, is by any means much more practical than managing multiple folders;
even more so since these tags usually need not be explicitly added, but already appear in the text.
(Similar to finding a browser bookmark by its title instead of folder navigation.)

Best, this method aligns with the workings of our mind, as we often recall files by remembering related words rather than a single label.

# Convenient Note and Link Creation by Keywords

Our Zettelkasten should enable us to:

- automatically create a note with designated search terms if absent,
- insert a link to a note through content searches, and
- open linked notes by placing the cursor near the link and some key combo.

Here's our take on implementing such a note-taking system in Vim:

A configuration of [notational-fzf-vim](https://github.com/alok/notational-fzf-vim) that comes with a command (`:NV`) to fuzzily find a note by its content.
The clou of [notational-fzf-vim](https://github.com/alok/notational-fzf-vim) (using below adaption of `NV_note_handler()`):
if no note is found (or a special key combination, say `'ctrl-x`, is hit), then one is created, with its title given by the search terms (whose separating blanks are replaced with hyphens).

The paths of the Zettelkasten (and additional folders of notes) are listed in `g:nv_search_paths`, that of the Zettelkasten coming first.
To jump to a note, type `gf` when the cursor is on the file name (made possible by including `g:nv_search_paths` in `&path` and setting `&suffixesadd`).

Finally, a link to a note can be inserted by hitting `C(trl)-z` (customizable by `g:insert_note_key`) that searches for the term before the cursor, which can then be refined in a fuzzy searcher.

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
        \ 'source':  'rg --follow --smart-case --color=never --no-messages --files-with-matches "" ' . s:search_paths,
        \ 'reducer': function('NV_make_note_link'),
        \ 'options': '--multi --reverse --margin 15%,0 --preview=''rg --pretty --context 3 -- {q} {1}''',
        \ 'up':    5})
endfunction
" }}}

let s:insert_note_key = get(g:, 'nv_insert_note_key', '<c-z>')
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

