---
layout: post
title: "Reading binary files in the Shell and friends"
date: 2025-09-30
categories: less mutt vim libreoffice
comments: true
---

Preview PDFs, Office documents, and other binary formats directly in terminals and TUIs without launching heavy GUI apps:  
Install converters such as poppler, unrtf, pandoc, LibreOffice, or Tika, then enable `lesspipe` (or `vim-office`), and tailor `~/.mailcap` to route previews per context.

- On Windows --- whether in PowerShell or Command Prompt (cmd.exe) --- use the [vim-office](https://github.com/konfekt/vim-office) plugin to extract and display human‑readable text from binary files in Vim.

- On macOS, Linux, or on Windows via Windows Subsystem for Linux (WSL), Git Bash (or more generally MSYS2), use [lesspipe.sh](https://github.com/wofr06/lesspipe) or lesspipe to preview binary files in tools such as `less`, `mutt`, `vim`, or file managers like `ranger`, `lf`, `nnn`, and `yazi`.  
See the [lesspipe wiki](https://github.com/wofr06/lesspipe/wiki/vim) for setup instructions to integrate with these programs and preview binary files.

- To customize text conversion, use run-mailcap (installed by default on Debian and derivatives such as Ubuntu or Linux Mint) with a suitable [~/.mailcap](https://gist.github.com/Konfekt/9797372146e65a70a44c1e24a35ae0a2) file that routes callers (`less`, `mutt`, `vim`, and file managers like `ranger`, `lf`, `nnn`, and `yazi`) to appropriate handlers to the appropriate handlers (using [mutt_bg_run](https://github.com/RichiH/mutt_bgrun/blob/master/mutt_bgrun)).

```
application/pdf; $HOME/.config/mutt/bin/mutt_bgrun "${PDFVIEWER:-zathura}" %s; test=test -n "$DISPLAY"; nametemplate=%s.pdf; description="PDF Document"
application/pdf;                   pdftotext       -nopgbrk -q -- %s -; test=test -n "$VIM"; nametemplate=%s.pdf; copiousoutput
application/pdf; pdftotext -l 20 -nopgbrk -q -htmlmeta -- %s - \| w3m -dump -T text/html; test=ps -o comm= -p "$PPID" | grep -Eq '^(mutt|neomutt)$'; nametemplate=%s.pdf; copiousoutput
```

The conversion from a file in a binary format to text requires the installation of appropriate external converters such as [unrtf](http://ftp.gnu.org/gnu/unrtf/), [pandoc](http://pandoc.org), [docx2txt.pl](https://github.com/arthursucks/docx2txt), [odt2txt](https://github.com/dstosberg/odt2txt), [xlscat](https://github.com/Tux/Spreadsheet-Read/tree/master/scripts), [xlsx2csv.py](https://github.com/dilshod/xlsx2csv) or [pptx2md](https://github.com/ssine/pptx2md), but already [LibreOffice](https://www.libreoffice.org/download/download/) or [Tika](https://tika.apache.org/download.html) can go a long way.

If you use Debian or one of its derivatives, such as Ubuntu or Linux Mint, these packages have you covered:

```sh
sudo apt install unrtf catdoc abiword xlsx2csv wv docx2txt odt2txt poppler-utils djvulibre-bin w3m qpdfview p7zip-full zathura
```
