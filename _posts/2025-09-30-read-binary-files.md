---
layout: post
title: "Reading binary files in the Shell and friends"
date: 2025-09-30
categories: less mutt vim libreoffice
comments: true
---

In Powershell or the `cmd.exe` prompt on Microsoft Windows, use the [vim-office](https://github.com/konfekt/vim-office) plug-in to display meaningful text of a binary files read by Vim.

On MacOS, Linux or in WSL / Git Bash on Microsoft Windows, to read a binary file not only in Vim, but more generally in `less`, `git`, `mutt`, `vim` or `ranger` (and its variants `lf`, `nnn` or `yazi`), you can use [lesspipe.sh](https://github.com/wofr06/lesspipe) or `lesspipe` (which comes installed on Debian and its derivatives, such as Ubuntu or Linux Mint).
See the [lesspipe wiki](https://github.com/wofr06/lesspipe/wiki/vim) on how to set up these tools for lesspipe to show read binary files.

To customize the text conversion, you can use run-mailcap (which comes installed on Debian and its derivatives, such as Ubuntu or Linux Mint) with a suitable [~/.mailcap](https://gist.github.com/Konfekt/9797372146e65a70a44c1e24a35ae0a2) file in which you can distinguish between the program displaying the text output, for example, `less`, `mutt`, `vim` or `ranger` (respectively `lf`, `nnn` or `yazi`).

The conversion from a file in a binary format to text requires the installation of appropriate external converters such as [unrtf](http://ftp.gnu.org/gnu/unrtf/), [pandoc](http://pandoc.org), [docx2txt.pl](https://github.com/arthursucks/docx2txt), [odt2txt](https://github.com/dstosberg/odt2txt), [xlscat](https://github.com/Tux/Spreadsheet-Read/tree/master/scripts), [xlsx2csv.py](https://github.com/dilshod/xlsx2csv) or [pptx2md](https://github.com/ssine/pptx2md), but already [LibreOffice](https://www.libreoffice.org/download/download/) or [Tika](https://tika.apache.org/download.html) can go a long way.

If you use Debian or one of its derivatives, such as Ubuntu or Linux Mint, this command installs some of these converters

```sh
apt install unrtf catdoc abiword xlsx2csv wv docx2txt odt2txt poppler-utils djvulibre-bin qpdfview p7zip-full
```
