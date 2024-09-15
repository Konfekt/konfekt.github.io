---
layout: post
title: "LF config for Linux and Windows"
date: 2024-09-15
categories: lf ranger Dotfiles linux windows os-independent config
comments: true
---

LF (List Files) is a terminal file manager written in Go, designed to be a simpler and faster alternative to Ranger.
It provides a minimalist and efficient interface for navigating and managing files directly from the command line.
It starts particularly faster than Ranger, using Python (3.11 on Debian 12), important if it is repeatedly called from the terminal (or Vim).

While Ranger offers a more feature-rich interface with previews and tabs, LF opts for a more minimalistic approach.
In comparison to Ranger,

- LF works on various operating systems, including Linux, macOS, and Windows.
- LF is generally faster and consumes fewer resources compared to Ranger.
- LF has fewer dependencies, making it easier to install and maintain.

# Getting Started with LF

To install LF, you can use the following commands depending on your operating system:

- **Linux**: `sudo apt-get install lf`
- **macOS**: `brew install lf`
- **Windows**: Download the binary from the [official GitHub repository](https://github.com/gokcehan/lf) or use a package manager such as [scoop](https://scoop.sh) and run `scoop install lf`.

After installation, you can start LF by simply typing `lf` in your terminal.

# Cross Platform Config

To set up `lf` (a terminal-based file manager) with shortcuts similar to Total Commander, you need to customize the `lfrc` configuration file.
The provided example configuration maps several Total Commander-like shortcuts to `lf` functionalities, such as

- renaming files with `<f-2>`,
- creating directories with `<f-7>`,
- opening files in gVim with `<f-4>`,
- deleting files with `<f-8>`, and 
- opening a terminal in the current directory with `<f-9>`.

Inspired by [Ali's post on using LF on Windows](https://ahrm.github.io/jekyll/update/2022/04/02/using-lf-file-manager-on-windows.html), we give a configuration works on both Windows and Linux.

Platform-specific configurations adjust behaviors such as file previews and terminal commands.
For example, on Linux/Unix, `less` is used for previews, and the path is copied using `xclip`.
On Windows, clipboard operations utilize `clip`, and paths are handled with Windows-specific commands.

Ensure the `LF_CONFIG_HOME` environment variable is set on Windows.
You can do this via the Environment Variables dialogue, using `RapidEE`, or by adding it to the appropriate shell profile:

- For Clink, add to `clink/scripts/envars.lua`: `os.setenv("LF_CONFIG_HOME", userprofile .. "\\.config")`
- For PowerShell, add to `Profile.ps1`: `$env:LF_CONFIG_HOME = "$env:XDG_CONFIG_HOME"`
- For MSYS2 (e.g., Git Bash), add to `.bash_profile`: `export LF_CONFIG_HOME="$XDG_CONFIG_HOME"`

While [lesspipe.sh](https://github.com/wofr06/lesspipe/?tab=readme-ov-file#2-usage) as a previewer handles many binary formats, much more performant is a custom-made [scope.sh](https://gist.github.com/Konfekt/c3ccd2ce83804103d8ac77b2e863f532), especially when used in MSYS2 (Git Bash).

```sh
{% raw %}
# ~/.config/lf/lfrc
set info size
set dircounts

# Leverages LESSOPEN=lesspipe.sh for previews ...
set previewer less
# # ... but is much slower than scope.sh
# set previewer ~/.config/ranger/scope.sh

# Key mappings
map . :read; cmd-history-prev; cmd-enter
map , read
map ; toggle-preview

# Commands for opening files
cmd open-with-gui &$@ "$fx" # For GUI applications
cmd open-with-cli $$@ "$fx" # For CLI/TUI applications
map O push :open-with-gui<space>
map o push :open-with-cli<space>

# Create directories
cmd mkdir %mkdir "$@"
map <f-7> push :mkdir<space>

# Load platform-specific configurations
${{ lf -remote "send $id source ~/.config/lf/unix.lfrc" && exit }}
source ~/.config/lf/win32.lfrc
{% endraw %}
````

```sh
{% raw %}
# ~/.config/lf/unix.lfrc
set period 1

&[ "$LF_LEVEL" -eq 1 ] || lf -remote "send $id echoerr \"Warning: You're in a nested lf instance!\""

cmd on-redraw %{{
    if [ "$lf_width" -le 80 ]; then
        lf -remote "send $id set ratios 1:2"
    elif [ "$lf_width" -le 160 ]; then
        lf -remote "send $id set ratios 1:2:3"
    else
        lf -remote "send $id set ratios 1:2:3:5"
    fi
}}

# Set the terminal title to the current directory
cmd on-cd &printf '\033]0;lf - %s\007' "${PWD/#$HOME/\~}" > /dev/tty
# Unset pane_path when quitting
cmd on-quit &printf '\033]7;\033\\' > /dev/tty
on-cd

map i $less "$f"

# Quick rename
cmd rename %mv -i "$f" "$0"
map <f-2> push :rename<space>

# Open file in gvim
map <f-4> &gvim "$f"

# Create files
# cmd touch %touch "$@"
# map <f-4> push :touch<space>

# Open file manager in current directory
map . push &xdg-open<space>.<enter>

# Needs `pipx install trash-cli` on Unix
cmd trash %trash $fx
map <f-8> trash

# Open terminal in current directory
map <f-9> push &urxvt<space>-cd<space>$PWD<enter>

# Copy file path
map Y %echo $fx | xclip -rmlastnl -selection clipboard

map <c-z> $kill -STOP "$PPID"
{% endraw %}
```

```sh
# ~/.config/lf/win32.lfrc
# Too slow on Windows
# set period 1

set filesep " "

map i $less %fx%

# Quick rename
cmd rename %move /-Y %f% $0
map <f-2> push :rename<space>

# Open file in gvim
map <f-4> &gvim %f%

# Open file manager in current directory
map . push &start<space>.<enter>

# Needs `npm install -g trash-cli` on Windows and `pipx install trash-cli` on Unix
cmd trash %trash %fx%
map <f-8> trash

# Open terminal in current directory
map <f-9> push &start<space>pwsh<space>-wd<space>.<enter>

# Copy file path
map Y %echo %fx% | clip
```

# Vim Integration

For Vim users, [this snippet](https://gist.github.com/Konfekt/8e484af2955a0c7bfe82114df683ce0f) maps `-` to open `lf` or a fallback (e.g., `yazi`, `nnn`, `ranger`, or `netrw`) at the current file.
