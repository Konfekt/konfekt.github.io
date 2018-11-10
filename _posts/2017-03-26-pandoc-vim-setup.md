---
layout: post
title: "How to use Pandoc"
date: 2017-03-26
categories: pandoc markdown vim
comments: true
---

# Pandoc

Pandoc is the swiss-army knife for converting files from one markup format into another:

## What does Pandoc do?

Pandoc can convert documents from

- markdown, reStructuredText, textile, HTML, DocBook, LaTeX, MediaWiki markup, TWiki markup, OPML, Emacs Org-Mode, Txt2Tags, Microsoft Word docx, EPUB, or Haddock markup

to

- HTML formats: XHTML, HTML5, and HTML slide shows using Slidy, reveal.js, Slideous, S5, or DZSlides.
- Word processor formats: Microsoft Word docx, OpenOffice/LibreOffice ODT, OpenDocument XML
- Ebooks: EPUB version 2 or 3, FictionBook2
- Documentation formats: DocBook, GNU TexInfo, Groff man pages, Haddock markup
- Page layout formats: InDesign ICML
- Outline formats: OPML
- TeX formats: LaTeX, ConTeXt, LaTeX Beamer slides
- PDF via LaTeX
- Lightweight markup formats: Markdown, reStructuredText, AsciiDoc, MediaWiki markup, DokuWiki markup, Emacs Org-Mode, Textile

## What does Pandoc do for me?

I use `pandoc` to convert documents from

- markdown

to

- HTML
- Microsoft Word docx (force majeure!), OpenOffice/LibreOffice ODT, OpenDocument XML
- LaTeX Beamer slides
- PDF via LaTeX

## What does Pandoc do better than the specialized tools?

### Accessibility:

Code in `markdown` is easily readable text.
In comparison:

- `Markdown` syntax is handier than `(La)TeX` syntax (Donald Knuth, inventor of `TeX`, wondered why it took so long to evolve from `LaTeX` to a more efficient markup language that compiles down to `TeX`, such as `markdown`),
- in particular `Markdown` syntax is handier than `LaTeX Beamer` syntax,
- math formulas are more easily written in `Markdown` than in `Microsft Word` or `LibreOffice`,
- it is especially suited for creating short `HTML` articles, such as blog entries.

## What does Pandoc do worse than the specialized tools?

- Functions specific to a markup language
	- either cannot be used,
	- or can be used, but may turn compilation into other languages invalid.
	(The `pandoc` syntax is as reduced as the common base among all markup languages into which it converts.)
- For more complex documents, there is certain consensus that the similar `asciidoc` format is more powerful and better suited (publisher's choice).
- Pandoc is still in development:
	- the output sometimes rough and needs to be retouched,
	- documentation is incomplete,
	- smaller ecosystem of tools, like editors and IDEs, for example:
		- `LaTeX` supports forward and inverse document search, that lets you jump from a position in the source `TeX` file to the corresponding position in the compiled `pdf` file, and the other way around.
		There is no such thing for markdown:
		markdown first compiles to `TeX` and then to `pdf`.
		- The `Vim` plugin for `markdown` is young and basic in comparison to that for `LaTeX` which is stable and powerful.

# Markdown

Markdown is simple, concise and intuitive:
Its cheat-sheet and documentation are one.

![documentation (= cheat-sheet) of the markdown syntax](/images/cheatsheet.png "cheatsheet")

Examples:

## Source:

```markdown
# An emphasized *itemization:*

- dog
- fox

# A bold **enumeration**:

1. Mum
0. Dad

# A table

|        | mum    | dad    |
|--------|--------|--------|
| weight | 100 kg | 200 kg |
| height | 1,20 m | 2,10 m |

```

## Output:

### An emphasized *itemization:*

- dog
- fox

### A bold **enumeration**:


1. Mum
0. Dad

### A table

|        | mum    | dad    |
|--------|--------|--------|
| weight | 100 kg | 200 kg |
| height | 1,20 m | 2,10 m |

# Compiling and Viewing

We use

- a `Makefile`, that sets a couple of compilation options, and
- a main markdown file, that sets a couple of document options.

Which parameters can be set by the command line, and which in the document, this choice is somewhat arbitrary and perhaps a shadow of `pandoc`'s unfinished state.

## Pandoc parameters

We can pass many options to `pandoc`, among those the most important ones (for us) are:

```sh
General options:

  --from = FORMAT
   Specify input FORMAT such as markdown, rst, ..
  --to = FORMAT
     Specify output FORMAT such as html, LaTeX, ..
  --output = FILE
     Write output to FILE instead of stdout.

Writer options:

  --standalone
     Produce output with header and footer.
  --table-of-contents
     Include a generated table of contents in output.
  --self-contained
     Produce standalone HTML file without external dep.
```

## Makefile

By a makefile, instead of having to pass the options for

- compilation,
- running,
- checking and cleaning,

each time on the command line, we call `make (run/check/clean)` and use those once and for all set in the makefile.

```sh
NAME = main
FILES = intro.md content.md conclusion.md
DEP = $(NAME).pandoc $(FILES)

PANDOC_OPTIONS=--standalone \
		--toc --number-sections \
		--filter pandoc-citeproc

PANDOC_DOCX_OPTIONS=
PANDOC_ODT_OPTIONS=
PANDOC_HTML_OPTIONS=
PANDOC_LATEX_OPTIONS=--include-in-header ~/.pandoc/headers/latex/header.tex

all: latex pdf
docx: $(DEP)
		pandoc \
				$(PANDOC_OPTIONS) \
				$(PANDOC_DOCX_OPTIONS) \
				--from markdown --to docx \
				$(NAME).pandoc $(FILES) --output $(NAME).docx
odt: $(DEP)
		pandoc \
				$(PANDOC_OPTIONS) \
				$(PANDOC_DOCX_OPTIONS) \
				--from markdown --to odt \
				$(NAME).pandoc $(FILES) --output $(NAME).odt
html: $(DEP)
		pandoc \
				$(PANDOC_OPTIONS) \
				$(PANDOC_HTML_OPTIONS) \
				--from markdown --to html5 \
				$(NAME).pandoc $(FILES) --output $(NAME).html
latex: $(DEP)
		pandoc \
				$(PANDOC_OPTIONS) \
				$(PANDOC_LATEX_OPTIONS) \
				--from markdown --to latex \
				$(NAME).pandoc $(FILES) --output $(NAME).tex
pdf: latex
		latexrun $(NAME).tex

run: run-pdf
run-docx: docx
		libreoffice --nologo $(NAME).docx \
		>/dev/null /2> &1 &
run-odt: odt
		libreoffice --nologo $(NAME).odt \
		>/dev/null /2> &1 &
run-html: html
		$(BROWSER) $(NAME).html
run-pdf: pdf
		$(DFVIEWER) $(NAME).pdf \
		>/dev/null /2> &1 &

clean: clean-docx clean-odt clean-html clean-pdf
clean-docx:
		rm --force --verbose *.docx
clean-odt:
		rm --force --verbose *.odt
clean-html:
		rm --force --verbose *.html
clean-pdf:
		latexrun --clean-all

.PHONY: all run clean
```

The command `make`, corresponding to the entry `all:`, generates the output file, in our case the `pdf` document.
For example,

- `make docx` generates a `docx` document,
- `make html` generates a `HTML` document,
- `make latex` generates a `TeX` document,

The option `all: latex pdf` is the default option, that is,

- `make` generates first a `TeX` and then a `pdf` document.

We recommend [latexrun](https://github.com/aclements/latexrun "latexrun") as a good `LaTeX` "debugger".
Still, note that we first have to spot first the error in the `TeX`, then the corresponding one in the `markdown` document.

The command `run` displays the output file, for example,

- `make run-html` shows the `HTML` document in a browser (such as `Firefox`),
- `make run-odt` shows the `ODT` document in LibreOffice,

The option `run` is the default option, that is,

- `make run` displays the `pdf` document in a pdf-viewer (such as zathura).

Finally, `make clean` removes all output files.

## Main file

This file sets at the top the title, author and date of the document.
Below, additional options,

- one general option, `lang` that controls for example the labeling of the table of content and references, and
- various TeX options, such as:

    - document type,
    - font size, and
    - depth of the section numbering.

```markdown
% Pandoc é bem massa!
% [Konfekt](mailto:Konfekt@users.noreply.github.com)
% FLISoL, CESMAC Maceió, 8. Abril 2017

---
lang:                 pt

# latex:
babel-lang:           brazilian
documentclass:        scrartcl
classoption:          final,DIV = calc,headings = normal,bibliography = totoc
fontsize:             12pt
citecolor:            Sepia
linkcolor:            Sepia
urlcolor:             Sepia
toc-depth:            3
secnumdepth:          2
# bibliography:         pandoc.bib
...
```

# Setting up Vim

Let us facilitate compilation and editing of `pandoc` files, the first by built-in functionality, latter by dedicated plugins.

## Automatic compilation and reload

To make Vim compile our file after every save, add to the (newly created, if necessary) file `~/.vim/after/ftplugin/markdown.vim` the line:

```vim
autocmd BufWrite <buffer> silent lmake!
```

If the output is

- `pdf` (via TeX), then the pdf viewer `zathura` automatically reloads the changed pdf file,
- `html`, then the `Firefox` plugin `autoreload` automatically reloads the changed html file.

## Editing enhancements

The plugin `vim-pandoc`

- completes references in your library when hitting the ` < Tab > ` key.
- folds sections and code,
- gives a Table of Contents.

The plugin `UltiSnips` expands lengthier markdown syntax such as

- `[` to `[link](http://url "title")`, and
- `![` to `![alt](url "title")`

The plugin `vim-template` prefills, on editing

- a new `makefile`, the makefile with the above boilerplate `makefile` code, and
- a new `pandoc` file, the `pandoc` file with the above boilerplate main file code.
