---
layout: post
title: "How to make yourself feel at HOME as a Linux user stranded on an administrated Microsoft Windows machine"
date: 2021-12-10
categories: linux windows git-bash git bash wsl scoop powershell cmd autohotkey 
comments: true
toc: true
---

If after years on Linux (or MacOS) you find yourself using an administrated Microsoft Windows machine, then you may recover many tools from Linux and adapt it to your muscle-memory by cherry-picking some of the advice below:

# Scoop

Use [scoop.sh](https://scoop.sh) to install all applications into  your personal folder `%USERPROFILE%`, the analog of `$HOME`.
Some suggestions, in particular of command-line tools (whose installation Scoop by default restricts to):

```
    git delta
    clink
    vim-nightly

    less bat

    fd 
    grep ripgrep ag
    sed gawk 

    fzf peco 

    curl wget aria2

    ntop
    lf
    gow 
    ctags
    shellcheck
    make 
    gopass
    poppler

    python nodejs perl lua
```

## Unix Tools

Install `Gow`, a collection of (older versions of) Unix tools, as a fall back in case you did not install the package (with )he most recent version) of the tool itself (say `less`, `grep`, `sed`, `gawk`, `curl` or `wget`).

- [ntop](https://community.chocolatey.org/packages/ntop) is the Microsoft Windows analog of [htop](https://htop.dev).
- [lf](https://community.chocolatey.org/packages/lf) is a Go version of `ranger` that runs in particular natively on Windows.
- [gopass](https://community.chocolatey.org/packages/gopass) implements the [password-store](https://www.passwordstore.org) (and many more features) in particular on Windows.
- [poppler](https://community.chocolatey.org/packages/poppler) extracts text from PDF files (useful for treating them on the command-line).

## Vim

The current version of `vim` is linked to 32-bit (x86) DLLs of Python 3.6 which can be installed by `scoop install python@3.6.9 -a 32` (but beware of updating the `python` package).
`vim-nightly` instead is linked to 64-bit DLLs of Python 3.10;
use the package `python310` (from the Scoop bucket `Versions`) instead of `python` (from bucket `Main`) to ensure that an update by `scoop` keeps the major version  `3.10`.

```
bucket add versions && scoop install python310
```

To let Vim find the Python character encodings, the environment variable `PYTHONHOME` had to be set to `%USERPROFILE%\scoop\apps\python310\current`.

# Clink for CMD.exe

`cmd.exe` continues to be the standard shell for many applications.
`Clink` (installable by Scoop) adds features known from GNU Readline, best known from `Bash`, to its terminal [Conhost](https://en.wikipedia.org/wiki/Windows_Console#Windows_NT_and_Windows_CE);
most notably : a command history, command completion and line-editing bindings.
<!-- - change fonts in standard properties -->
To launch `clink` with `Conhost`, append `/K clink inject` to the command in the properties' sheet of the `cmd.exe` shortcut in Start Menu.

To recover the [most essential aliases](https://github.com/Konfekt/cmd-aliases.bat)  under the `cmd.exe` prompt, put [them](https://github.com/Konfekt/cmd-aliases.bat) into a folder listed in `%PATH%` (conveniently set by [Rapidee](https://community.chocolatey.org/packages/rapidee), say).

To jump to *frecent* (= frequent and recent) directories by typing `z` and a partial match of their path (say `z Doc` to change directory to `%USERPROFILE%\Documents`), put [z.lua](https://github.com/skywind3000/z.lua) into a folder listed in `%PATH%` and (a link to it) `%CLINK_DIR%`.

To customize the color scheme of the `cmd.exe` terminal [Conhost](https://en.wikipedia.org/wiki/Windows_Console#Windows_NT_and_Windows_CE), download [colortool](https://community.chocolatey.org/packages/colortool/) run in the `cmd` shell, say,

    colortool -b .\schemes\solarized_dark.itermcolors

Then open the properties sheet and hit "Ok" to apply the dark Solarized color scheme.

# PowerShell

See [my Powershell profile](https://github.com/Konfekt/powershell/tree/main) 
to bring muscle memory from Linux back to fruition by some aliases to search text in files, for `rsync`, to navigate the file system, operate on the clipboard, ... and much more;
loading the modules:

- [PSReadLine](https://www.powershellgallery.com/packages/PSReadLine) for Readline bindings
- [PSReadLineHistory](https://www.powershellgallery.com/packages/PSReadLineHistory) and lookup of previous commands
- [PSFzf](https://www.powershellgallery.com/packages/PSFzf) for fuzzy finding files, folders and previous commands
- [posh-git](https://www.powershellgallery.com/packages/posh-git) for a Git status prompt
- [z](https://www.powershellgallery.com/packages/z) to jump to frecent (= frequent and recent) directories by typing `z` and a partial match of their path (say `z Doc` to change directory to `%USERPROFILE%\Documents`) (or, alternatively use again [z.lua](https://github.com/skywind3000/z.lua))
- [PSEverything](https://www.powershellgallery.com/packages/PSEverything) for searching the indexed hard disc
- [Recycle](https://www.powershellgallery.com/packages/Recycle) for sending files to the Recycle bin (instead of deleting them), and
- [pscx](https://www.powershellgallery.com/packages/pscx) for miscellaneous helper tools.

# Git

## Symbolic Links

If you have admin rights, then [you can grant your account the right to create symbolic links](https://github.com/git-for-windows/git/wiki/Symbolic-Links#allowing-non-administrators-to-create-symbolic-links).
However, as a mere user without admin rights, you cannot [create symbolic (or soft) links](https://github.com/Konfekt/cmd-aliases.bat/blob/main/ln-s.bat), but you can instead [create junctions (to folders) and hard links (to files)](https://github.com/Konfekt/cmd-aliases.bat/blob/main/ln.bat).
Thus, a symbolic link in a repository cannot be recreated but is replaced by a text file containing the path it links to.
As a remedy, add the following code snippet to `%USERPROFILE%/.config/git/config` and run `git rm-symlinks` to replace all the repository's soft links by hard links:

```sh
# From https://stackoverflow.com/questions/5917249/git-symlinks-in-windows/16754068#16754068
rm-symlinks = "!__git_rm_symlinks() {\n  case \"$1\" in (-h)\n    printf 'usage: git rm-symlinks [symlink] [symlink] [...]\\n'\n    return 0\n  esac\n  ppid=$$\n  case $# in\n    (0) git ls-files -s | grep -E '^120000' | cut -f2 ;;\n    (*) printf '%s\\n' \"$@\" ;;\n  esac | while IFS= read -r symlink; do\n    case \"$symlink\" in\n      (*/*) symdir=${symlink%/*} ;;\n      (*) symdir=. ;;\n    esac\n\n    git checkout -- \"$symlink\"\n    src=\"${symdir}/$(cat \"$symlink\")\"\n\n    posix_to_dos_sed='s_^/\\([A-Za-z]\\)_\\1:_;s_/_\\\\\\\\_g'\n    doslnk=$(printf '%s\\n' \"$symlink\" | sed \"$posix_to_dos_sed\")\n    dossrc=$(printf '%s\\n' \"$src\" | sed \"$posix_to_dos_sed\")\n\n    if [ -f \"$src\" ]; then\n      rm -f \"$symlink\"\n      cmd //C mklink //H \"$doslnk\" \"$dossrc\"\n    elif [ -d \"$src\" ]; then\n      rm -f \"$symlink\"\n      cmd //C mklink //J \"$doslnk\" \"$dossrc\"\n    else\n      printf 'error: git-rm-symlink: Not a valid source\\n' >&2\n      printf '%s =/=> %s  (%s =/=> %s)...\\n'           \"$symlink\" \"$src\" \"$doslnk\" \"$dossrc\" >&2\n      false\n    fi || printf 'ESC[%d]: %d\\n' \"$ppid\" \"$?\"\n\n    git update-index --assume-unchanged \"$symlink\"\n  done | awk '\n    BEGIN { status_code = 0 }\n    /^ESC\\['\"$ppid\"'\\]: / { status_code = $2 ; next }\n    { print }\n    END { exit status_code }\n  '\n}\n__git_rm_symlinks"
```

## Proxy Setup

If your internet connection uses a proxy, say `http://proxy.com` at port `8888`, add the following lines to `%USERPROFILE%/.config/git/config` to connect with the default user account

```
[http]
  sslbackend = schannel
  proxy = http://:@proxy.com:8888
```

To switch authentication methods (as implemented by `curl` among `basic`, `digest`, `ntlm`, `negotiate`), say to `negotiate`, add the line

```
  proxyauthmethod = negotiate
```

## SSH

Windows 10 version 1803 (April 2018) or above comes with an [OpenSSH](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview) client.
To stop Git asking for the SSH passphrase, enable the OpenSSH(-agent) by: 


    Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service

and set the environment variable `%GIT_SSH%` to `C:\Windows\system32\OpenSSH\ssh.exe` (provided Windows was installed on `C:`).

See [here for further set-up](https://dev.to/qm3ster/setting-up-gitsshgpg-on-windows-5c85).

# Git Bash

To use a Linux shell under Windows, there is [WSL](https://docs.microsoft.com/en-us/windows/wsl/install) (Windows Subsystem for Linux), but it uses (in the default version 2) a virtual machine whose installation needs administrator rights. 
Instead, [Git BASH](https://gitforwindows.org/) to the rescue that loads your `.profile` and `.bash_profile` profile (but not `.bashrc`) and `.inputrc` (in `%USERPROFILE%`) for setting up your aliases, functions, ... and readline bindings.

It comes with [tig](https://jonas.github.io/tig/) and respects your `.tigrc`.

## Tools

Reuse [z.lua](https://github.com/skywind3000/z.lua) (or the original [z.sh](https://github.com/rupa/z)) to jump to a frecent directory by a part of its path is a worth-wile addition.
Have a look at [sensible.bash](https://github.com/mrzool/bash-sensible) to enable the most basic features and [oh-my-bash](https://ohmybash.nntoan.com/) for many more.
Many useful functions consist of merely a couple of lines, such as

```sh
rdm() {
  files="$(find . -maxdepth 1 \( -iname '*read*me*' -o -iname '*lies*mich*' -o -iname '*lisez*moi*' -o -iname '*leia*me*' -o -iname '*leggi*mi*' -o -iname '*lee*me*' \))"
  if [ -n "$files" ]; then
    "${PAGER:-less}" $files
    # In less, optionally view file $EDITOR (or $VISUAL) by hitting 'v'
  else
    builtin print 'No README files.'
  fi
}
```

to view the first found Readme file.
Be aware though that Git Bash comes with its proper `gpg` (in `/usr/bin/gpg`) [that cannot share its keys](https://dev.gnupg.org/T3020) with `gpg.exe` (in `%PATH%`).
One workaround is to use `gpg.exe` in `%PATH%` in Git Bash by overriding its built-in `gpg` by putting into `$PATH` the following shell script:

```sh
gpg="$(type 2>/dev/null -a gpg | head | tail -n 1)"
gpg=${gpg#* * }
echo $gpg
if [ "$gpg" != "$0" ] && command -v "$gpg" > /dev/null 2>&1; then
  $gpg "$@"
elif command -v gpg > /dev/null 2>&1; then
  gpg "$@"
else
  exit 127
fi
```

## Package Management

If you installed Git as a user (say by Scoop, in case it did not come pre-installed), [install Pacman](https://stackoverflow.com/questions/32712133/package-management-in-git-for-windows-git-bash/65204171#65204171) to add many Linux packages (other than `Vim`, `Tig`, ... that come with Git Bash), such as the e-mail client [Mutt](https://neomutt.org/).

## Environment Variables

To let `Java` write Unicode to the console, add the environment variable `%_JAVA_OPTIONS%` with value `_JAVA_OPTIONS = -Dfile.encoding=UTF-8`;
further suggestions

    HOME = %USERPROFILE%
    EDITOR = %USERPROFILE%\scoop\apps\vim\current\vim.exe
    PAGER = less
    BROWSER = firefox
    PDFVIEWER = mupdf
    XDG_CACHE_HOME = %USERPROFILE%\.cache
    XDG_CONFIG_HOME = %USERPROFILE%\.config
    XDG_STATE_HOME = %USERPROFILE%\.local\state
    XDG_DATA_HOME = %USERPROFILE%\.local\share

# Windows Terminal

Install the Windows Terminal (say by `scoop bucket add extras && scoop install windows-terminal`), and configure it by changing its settings.json (by hitting `Ctrl-Alt-,` inside it) as follows:

- To launch Git Bash:

```
"profiles": 
    "list": 
    [ {
            "commandline": "%USERPROFILE%/scoop/apps/git/current/usr/bin/bash.exe -i -l",
            "icon": "%USERPROFILE%/scoop/apps/git/current/mingw64/share/git/git-for-windows.ico",
            "startingDirectory": "%USERPROFILE%",
            "tabTitle": "Git Bash",
    } ]
```

If Git is installed in its standard path, then replace the `commandline` and `icon` strings by

```
            "commandline": "%PROGRAMFILES%/git/usr/bin/bash.exe -i -l",
            "icon": "%PROGRAMFILES%/Git/mingw64/share/git/git-for-windows.ico",
```

- To use Clink in the `cmd.exe` command prompt:

```
"profiles": 
    "list": 
    [ { "commandline": "cmd /k clink inject", } ]
```

- To start Powershell with an empty terminal:

```
"profiles": 
    "list": 
    [ { "commandline": "pwsh -NoLogo", } ]
```

- to use `Ctrl-Shift-V` to paste instead of `Ctrl-V` (which starts visual block mode in Vim):

```
"actions": 
    [
        { "command": "unbound", "keys": "ctrl+v" },
        { "command": "paste", "keys": "ctrl+shift+v" },
    ]
```

- to toggle Focus Mode (that hides the tabs) by hitting `F12` 

```
"actions": 
    [ { "command": "toggleFocusMode", "keys": "f12" }, ]
```

# Window Manager

## win-10-virtual-desktop-enhancer

The [Autohotkey](https://autohotkey.com/) script [win-10-virtual-desktop-enhancer](https://github.com/phazzzy/win-10-virtual-desktop-enhancer) brings back the key bindings of Window Managers such as [i3](https://i3wm.org), [dwm](https://dwm.suckless.org/) (or its fork [Awesome](https://awesomewm.org/)) to switch virtual desktops, provided that the section `KeyboardShortcutsModifiers` of its settings file `settings.ini` contains

```
[KeyboardShortcutsModifiers]
SwitchDesktopNum=Win
MoveWindowToDesktopNum=Win, Shift
```

## PowerToys

[Powertoys](https://community.chocolatey.org/packages/powertoys) finds the windows scattered all over the virtual desktops;
install it via Scoop by

    scoop bucket add versions
    scoop install dotnet3-sdk PowerToys

## Run-or-Raise

A run-or-raise application switcher (such as [jumpapp](https://github.com/mkropat/jumpapp) on X11) can be achieved by [AutoHotKey](https://www.autohotkey.com/) with a `ahk` script containing the function

```ahk
; From https://gist.github.com/dewaka/c494543b4cd2a2dcd09cc5a6aa0f7517
RunOrRaise(exePath, winID:="") {
  if (winID = "") {
    exeName := substr(exePath, instr(exePath, "\",, 0)+1)
    winID := "ahk_exe " exeName
  }
  If not WinExist(winID) {
    EnvGet, UserProfile, USERPROFILE
    Run, %exePath%, %UserProfile%
    WinWait, % winID
    WinActivate
  }
  else {
    If WinExist(winID) {
      If WinActive() {
        WinWait, % winID
        WinMinimize
      }
      else {
        WinWait, % winID
        WinActivate
      }
    }
  }
}
```

used as follows to run or raise `Excel` by simultaneously pressing `Windows+X`, [Gvim](https://community.chocolatey.org/packages/vim) by `Windows+G`, [TotalCommander](https://community.chocolatey.org/packages/totalcommander) by `Windows+T` and the standard browser by `Windows+B` ...:

```
#Return::RunOrRaise("wt.exe", "ahk_exe WindowsTerminal.exe")

#x::
  excel = %ProgramFiles%\Microsoft Office\root\Office16\EXCEL.EXE
  RunOrRaise(excel, "ahk_class XLMAIN")
  Return
#g::
  EnvGet, UserProfile, USERPROFILE
  gvim = %UserProfile%\scoop\apps\vim\current\gvim.exe
  RunOrRaise(gvim, "ahk_class Vim")
  Return
#t::
	EnvGet, Commander_Path, COMMANDER_PATH
	totalcmd = %Commander_Path%\totalcmd64.exe
	RunOrRaise(totalcmd)
  Return

; From https://www.autohotkey.com/board/topic/67330-how-to-open-default-web-browser/
DefaultBrowser() {
	; Find the Registry key name for the default browser
	RegRead, BrowserKeyName, HKEY_CURRENT_USER, Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.html\UserChoice, Progid

	; Find the executable command associated with the above Registry key
	RegRead, BrowserFullCommand, HKEY_CLASSES_ROOT, %BrowserKeyName%\shell\open\command

	StringGetPos, pos, BrowserFullCommand, ",,1

	pos := --pos

	StringMid, BrowserPathandEXE, BrowserFullCommand, 2, %pos%
	Return BrowserPathandEXE
}
#b::RunOrRaise(DefaultBrowser())
```

The Window ID of an application can be discovered by the [Window Spy](https://amourspirit.github.io/AutoHotkey-Snippit/WindowSpy.html) included in the Autohotkey installation.

## Window Operation Shortcuts

Other shortcuts to manipulate windows, such as `Windows+M` to toggle whether a window is maximized can be added:
```
#m::toggleMaxWindow()
; http://xahlee.info/mswin/autohotkey_toggle_maximize_window.html
toggleMaxWindow() {
  WinGet, WinState, MinMax, A
  if (WinState = 1) {
    WinRestore, A
  }
  else {
    WinMaximize, A
  }
}
```
## Screen Switching

Download the Autohotkey script [MoveMouseToMonitorV6.ahk](https://www.experts-exchange.com/articles/33932/Keyboard-shortcuts-hotkeys-to-move-mouse-in-multi-monitor-configuration-AutoHotkey-Script.html) to focus screen `1`, `2`, ... (that is, to move your mouse pointer to it) by pressing `Alt+1`, `Alt+2`, ...


# Autohotkey

Autohotkey is a programming language on Microsoft Windows;
the casual user mostly uses it to bind global shortcuts to set commands, similar to [xbindkeys](https://wiki.archlinux.org/title/Xbindkeys), but [much more](https://www.autohotkey.com/boards/viewtopic.php?p=87947#p87947) is possible.
<!-- ## Googling -->
One such shortcut could be googling for the selected text, say by hitting `Windows+S` and launching an `ahk` file containing the following code snippet:

```
#s::SearchInternet()

SearchInternet() {
	Send, ^c
	Sleep 50
	tcb:=TrimString(clipboard)
	Run, http://www.google.com/search?q=%tcb%
	Return
}

TrimString(s) {
	; Remove all blank lines
	Loop {
		StringReplace, s, s, `r`n`r`n, `r`n, UseErrorLevel
		if ErrorLevel = 0  ; No more replacements needed.
			break
	}
	; Remove quotes and commas
	StringReplace, s, s, ", , All 			;"
	StringReplace, s, s, `,, , All
	; Replace all CR+LF's and &'s by Spaces
	StringReplace, s, s, `r`n, %A_SPACE%, All
	StringReplace, s, s, &, %A_SPACE%, All

	Return s
}
```

## Mapping Capslock to Control and Escape

Mapping `Capslock` to the `Control` and `Escape` key at once, the former when it is kept pressed, the latter when it is quickly released, (as achieved by say [xcape](https://wiki.archlinux.org/title/Xcape) on Linux) is [possible by Autohotkey](https://wiki.archlinux.org/title/Xcape);
however, more robust is [dual-key-remap](https://github.com/ililim/dual-key-remap).
Create an [elevated shortcut](https://winaero.com/create-elevated-shortcut-to-skip-uac-prompt-in-windows-10/) to make it work when running an application under admin privileges as well.

## Keyboard instead of Mouse

Run `control access.cpl` to enable pointer movement by the numpad or download [keynavish](https://github.com/lesderid/keynavish) to quickly (by bisecting the screen space) move the mouse pointer anywhere on the screen.

# File Manager

One classic mighty file manager is [TotalCommander](https://community.chocolatey.org/packages/totalcommander) (there are [others](https://www.gpsoft.com.au/index.html)) which can be made [portable](https://www.ghisler.com/usbinst.htm).
It has many useful plug-ins, such as 

## Listers (= file viewers) ...

such as ...

- [CudaLister](https://totalcmd.net/plugring/CudaLister.html) to view source code files and 
- [uLister](https://totalcmd.net/plugring/oilister.html) to view any imaginable file type,
- [slister](https://totalcmd.net/plugring/slister.html) to view PDFs

## Text Search plug-ins ...

... such as [TextSearch](https://totalcmd.net/plugring/TextSearch.html)

## File Readers plug-ins ...

... such as

- [ADB](https://forum.xda-developers.com/t/guide-total-commander-with-android-adb-plugin.2105707/) to control your Android phone
- [Registry](https://totalcmd.net/plugring/registry.html) to edit the Registry
- [Uninstaller](https://totalcmd.net/plugring/uninstaller.html) to uninstall applications,
- [Environment Variable](https://totalcmd.net/plugring/envvars.html) to read and edit environment variables ...

and more plug-ins to connect to `SFTP`, `WEBDAV` and cloud accounts by the author of Total Commander himself.

# Default Editor

To use your favorite editor as a default editor, say `Gvim` run [NotepadReplacer](https://www.binaryfortress.com/NotepadReplacer/) once.
Use [Text Editor Anywhere](https://www.listary.com/text-editor-anywhere) to paste the currently edited text (say in a entry box of the Browser) into your favorite editor.

## Vim

To use the same `vimrc` on Microsoft Windows and Linux, create a junction `vimfiles` in `%USERPROFILE%` to `dotfiles\vim` by entering in the `cmd` prompt the line

```
   mklink /J vimfiles dotfiles\vim
```

where `dotfiles\vim` contains a file `vimrc` with the lines

```vim
let s:vimfiles_dir = split(&runtimepath, ',')[0]
if isdirectory(s:vimfiles_dir)
  let g:vimfiles_dir = s:vimfiles_dir
elseif has('unix')
  let g:vimfiles_dir = expand('$HOME') . '/.vim'
else " if has('win32')
  " see :help vimfiles
  let g:vimfiles_dir = expand('$HOME') . '\vimfiles'
endif
unlet s:vimfiles_dir

" Use '.vim' instead of 'vimfiles' on Windows, too, to ease syncing across OSs
if has('win32')
  set runtimepath+=$HOME/.vim,$HOME/.vim/after
endif
```

## Plug-ins

Install [vim-office](https://github.com/konfekt/vim-office) to read Microsoft Office files in Vim.
Make `Gvim` go full-screen on hitting `F11` by installing [gvimfullscreen_win32](https://github.com/derekmcloughlin/gvimfullscreen_win32) (say by [vim-plug](https://github.com/junnegunn/vim-plug)) and adding the following lines to your `vimrc`:

```vim
let g:gvimfullscreen_dll = g:vimfiles_dir .
        \ '\plugged\gvimfullscreen_win32\gvimfullscreen' .
        \ (has('win64') ? '_64' : '') . '.dll'
nnoremap <silent> <F11> :<c-u>call libcallnr(g:gvimfullscreen_dll, 'ToggleFullScreen', 0)<bar>redraw<CR>
```

Best served with [goyo](https://github.com/junegunn/goyo.vim).

# Clipboard History

Install [Ditto](https://community.chocolatey.org/packages/ditto) to build and search through your clipboard history.

To make it work in GVim and in the Windows Terminal as well add custom keystrokes in `HKCU\Software\Ditto\PasteStrings` respectively in the file `Ditto.Settings` in the portable version (installed by Scoop)

```
    [PasteStrings]
    gvim.exe={Esc}{DELAY 200}"{Plus}gP
    WindowsTerminal.exe=^+V 
```

To exclude clips pasted from password apps, say [pass-winmenu](https://github.com/geluk/pass-winmenu), add:

    RegexFilter_0=.*
    RegexFilterByProcessName_0=pass-winmenu.exe

# Screen Capture

Simultaneously hit `Windows+Shift+S` to start the [Snipping Tool](https://support.microsoft.com/en-us/windows/use-snipping-tool-to-capture-screenshots-00246869-1843-655f-f220-97299b865f6b) for capturing part of the screen;
Have a look at [ksnip](https://www.dedoimedo.com/computers/ksnip-review.html) for annotating the screenshot.



# Conclusion

Now, after all this configuring, using Microsoft Windows resembles using Linux, though the former reacts noticeably slower than the original setup on Linux, despite modern hardware.

Hopefully this helped!
It was a long journey just to come back close to start.

