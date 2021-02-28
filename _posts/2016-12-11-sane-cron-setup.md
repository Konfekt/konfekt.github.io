---
layout: post
title: "A sane Anacron setup"
date: 2016-12-11
categories: cron anacron linux
comments: true
---

`cron` is a program that, continuously sitting in the background, runs an executable at a set time of day, as set up in a configuration file `crontab`.
`anacron` is a program, that, continually called by `cron`, runs an executable at a set day (week, month), as set up in a configuration file `anacrontab`.

An *`(ana)cron` job* is every such entry in the `(ana)crontab`, that tells `(ana)cron` which executable to run at which time/day.
Whereas `cron` will skip a job if the computer was down at the set time, `anacron` will only skip a job if the computer was down during the whole set day (week, month).
That is, `anacron` seizes every chance it gets to run the executable.
Therefore, whereas `cron` is made for servers, that are always on, `anacron` it made for personal computers, such as desktops, laptops, tablets, that are sometimes turned off.

Here, we want to go one step further and make `anacron` only run an executable if the computer is online and/or on power supply:
Instead of

- configuring `anacron` by listing the executables and their frequency (day, week or month) in a configuration file,
- we place them in appropriate folders, depending on their frequency (daily, weekly or monthly) and condition under which to be run (none, online, on power supply, or both).

# Aim

Run the executables in the folders

```
~
| .config/
| | anacron.daily/
| | | on/
| | | on_ac/
| | | on_line/
| | | on_line_ac/
| | anacron.weekly/
| | | on/
| | | on_ac/
| | | on_line/
| | | on_line_ac/
| | anacron.monthly/
| | | on/
| | | on_ac/
| | | on_line/
| | | on_line_ac/
```

daily (in `anacron.daily`), weekly (in `anacron.weekly`) and monthly (in `anacron.monthly`) either

- unconditionally (in `on`),
- when on power supply (= AC/DC), for example, updating a file index (in `on_ac`),
- when online, for example, updating an online backup (in `on_line`), or
- when on power supply and online, for example, updating software (in `on_line_ac`).


# Prerequisites

Besides having installed `cron` and `anacron`, the following programs have to be marked executable and placed into `$PATH`
(note that we will add the folder `~/bin` to `$PATH`, so that they all can be put inside `~/bin`.):

- The script [run-parts.sh](https://raw.githubusercontent.com/wolfbox/run-parts/master/run-parts.sh) runs sequentially all executables in a folder.
- The script [on_ac_power](https://raw.githubusercontent.com/OpenRC/openrc/master/scripts/on_ac_power) tests whether the computer runs on battery or AC/DC.
- The command-line tool `nm-online` tests whether the computer is online.
    However, the computer may be online, but behind a paywall or certain ports are blocked, such as the port `21` for `SSH`-connections.
    Therefore a better bet for checking connectivity is the program `nc`, though the check depends on a chosen server being up and running, say `github.com`.
    Both `nm-online` and `nc` are included in many Linux distributions.

- The schedulers `chrt` and `ionice` run programs in the background, so that they will not hog the CPU respectively hard disc.
    Both `chrt` and `ionice` are included in many Linux distributions.
- The auxiliary tool `cronic` to pass on (standard out and standard error) output only if the passed command (= cron job) fails (exits nonzero or crashes).
    For this error output (by default readable by `mailx`) to be most useful, the shell script of every cron job should begin with `set -o xtrace -o errtrace -o errexit -o nounset -o pipefail` to show debug output, exit on error or use of undeclared variable or pipe error.
    The tiny shell script `cronic` is available at [Chuck Houpt's homepage](http://habilis.net/cronic/) (and one of its many variants, such as `chronic` in Joey Hess's [moreutils](https://joeyh.name/code/moreutils/), included in many Linux distributions).

# Quick-Start

To readily set up the `anacron` as shown next step by step, clone [this repository](https://github.com/Konfekt/anacron) and follow its quick-start guide.

# Implementation

We assume the user name `konfekt`, that is, the user's home directory `~` is `/home/konfekt`.

## Setting up the Executables

Create the folder `~/.config/cron.hourly`.
Inside it, put the four executables

```
anacrontab_on.sh
anacrontab_on_ac.sh
anacrontab_on_line.sh
anacrontab_on_line_ac.sh
```

First, `anacrontab_on_line_ac.sh` runs anacron only if the computer is  online and on AC/DC, and reads

```sh
#!/bin/sh

# Do not run jobs when on battery power
(command -v on_ac_power >/dev/null /2> &1 && on_ac_power) || exit 0

# Do not run jobs when off-line
# (command -v nm-online >/dev/null /2> &1 && nm-online --timeout=10 --quiet) || exit 0
(nc -zw3 github.com 22) || exit 0

mkdir --parents "$XDG_DATA_HOME/anacron"
/usr/sbin/anacron -S "$XDG_DATA_HOME/anacron" -t "$XDG_CONFIG_HOME/anacrontab/on_line_ac" -s
```

Accordingly `anacrontab_on_line.sh` runs anacron only if the computer is  online, and reads

```sh
#!/bin/sh

# Do not run jobs when off-line
# (command -v nm-online >/dev/null /2> &1 && nm-online --timeout=10 --quiet) || exit 0
(nc -zw3 github.com 22) || exit 0

mkdir --parents "$XDG_DATA_HOME/anacron"
/usr/sbin/anacron -S "$XDG_DATA_HOME/anacron" -t "$XDG_CONFIG_HOME/anacrontab/on_line_ac" -s
```

and `anacrontab_on_ac.sh` runs anacron only if the computer is on AC/DC, and reads

```sh
#!/bin/sh

# Do not run jobs when on battery power
(command -v on_ac_power >/dev/null /2> &1 && on_ac_power) || exit 0

mkdir --parents "$XDG_DATA_HOME/anacron"
/usr/sbin/anacron -S "$XDG_DATA_HOME/anacron" -t "$XDG_CONFIG_HOME/anacrontab/on_line_ac" -s
```

and `anacrontab_on.sh` runs anacron unconditionally, and reads

```sh
#!/bin/sh

mkdir --parents "$XDG_DATA_HOME/anacron"
/usr/sbin/anacron -S "$XDG_DATA_HOME/anacron" -t "$XDG_CONFIG_HOME/anacrontab/on_line_ac" -s
```

## Setting up Anacron

Create the folder `~/.config/anacrontab`.
Inside it, put the four anacron configuration files

```
~/.config/anacrontab/on
~/.config/anacrontab/on_ac
~/.config/anacrontab/on_line
~/.config/anacrontab/on_line_ac
```

First, `~/.config/anacrontab/on_line_ac` runs all due daily, weekly and monthly anacron jobs to be run when the computer is online and on AC/DC.
It reads

```
SHELL=/bin/bash
USER=konfekt
HOME=/home/konfekt
BASH_ENV=/home/konfekt/.bash_profile
XDG_CONFIG_HOME=/home/konfekt/.config
XDG_CACHE_HOME=/home/konfekt/.cache
XDG_DATA_HOME=/home/konfekt/.local/share
PATH=/home/konfekt/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
# RANDOM_DELAY=5
# START_HOURS_RANGE=10-16

# period  delay  job-identifier         command
1         0      daily.on_line_ac       cronic chrt --idle 0 ionice -c2 -n7 run-parts.sh -v $XDG_CONFIG_HOME/anacron.daily/on_line_ac
7         5      weekly.on_line_ac      cronic chrt --idle 0 ionice -c2 -n7 run-parts.sh -v $XDG_CONFIG_HOME/anacron.weekly/on_line_ac
@monthly  10     monthly.on_line_ac     cronic chrt --idle 0 ionice -c2 -n7 run-parts.sh -v $XDG_CONFIG_HOME/anacron.monthly/on_line_ac
```

Accordingly,

- `~/.config/anacrontab/on_line` runs all due daily, weekly and monthly anacron jobs to be run when the computer is online.
- `~/.config/anacrontab/on_ac` runs all due daily, weekly and monthly anacron jobs to be run when the computer is on AC/DC.
- `~/.config/anacrontab/on` runs all due daily, weekly and monthly anacron jobs to be run unconditionally.

They read accordingly, that is, `on_line_ac` is replaced everywhere by `on_line`, `on_ac`, `on` respectively.

## Setting up Cron

Add a file

```sh
~/.config/crontab
```

with content

```sh
SHELL=/bin/bash
USER=konfekt
HOME=/home/konfekt
BASH_ENV=/home/konfekt/.bash_profile
XDG_CONFIG_HOME=/home/konfekt/.config
XDG_CACHE_HOME=/home/konfekt/.cache
XDG_DATA_HOME=/home/konfekt/.local/share
PATH=/home/konfekt/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
# RANDOM_DELAY=5
# START_HOURS_RANGE=10-16

# period        command
* * * * * *     cronic chrt --idle 0 ionice -c2 -n7 run-parts.sh -v $XDG_CONFIG_HOME/cron.hourly
```

On the terminal, run

```sh
crontab ~/.config/crontab
```

This way, `cron` checks, via `anacron`, every minute for all due (daily, weekly and monthly) jobs to be run unconditionally or when the computer is either online, on AC/DC, or online and on AC/DC.


# Extras

If you run jobs that need an SSH connection, for example an online backup over rsync, then the program [`keychain`](http://www.funtoo.org/Keychain "keychain") may interest you:
A shell script that will cache your passphrase, so that `cron` jobs can access it.

To set it up, initialize it on start-up of the user's session by

```sh
eval "$(keychain --agents ssh --quiet --eval ~/.ssh/id_rsa)"
```

(for example, `KDE` has the folder `~/.config/autostart-scripts` for this purpose) and call

```sh
[ -z "$HOSTNAME" ] && HOSTNAME="$(uname -n)"
[ -f ~/.keychain/"$HOSTNAME"-sh ] && . ~/.keychain/"$HOSTNAME"-sh
```

before the `cron` job, for example one using `rsync` to back up files to a distant `SSH` server.

# Example

For an example anacron job, here is a sample shell script which backs up all the files inside the home directory that are listed in `files`, except those listed in `exclude`, to a server;
see [this repository](https://github.com/Konfekt/backup2cloud.sh) for an expanded version and full instructions:

```sh
#!/bin/bash

set -o xtrace -o errtrace -o errexit -o nounset -o pipefail

# set up keychain to pass passphrase to ssh in cron jobs
[ -z "$HOSTNAME" ] && HOSTNAME="$(uname -n)"
[ -f ~/.keychain/"$HOSTNAME"-sh ] && . ~/.keychain/"$HOSTNAME"-sh
# CONFIG
FROM_FOLDER=$HOME
TO_FOLDER="$USER@rsync.server.com:/users/$USER"

FILES_FILE=$XDG_CONFIG_HOME/backup/files
EXCLUDE_FILE=$XDG_CONFIG_HOME/backup/exclude

LOG_FILE=$XDG_CACHE_HOME/backup/cloud/log

# because --files-from disables --recursive in --archive it must enable explicitly
RSYNC_BKP_ARGS="--recursive --archive --hard-links --ignore-errors --modify-window=1 --delete --compress --partial --human-readable --info=progress2 "
SSH_ARGS="-v -P"

# BACKUP Local -> Server:
LOG_FILE_DIR=$(dirname "${LOG_FILE}")
mkdir --parents "$LOG_FILE_DIR"

rsync $RSYNC_BKP_ARGS --rsh="ssh $SSH_ARGS" --log-file="$LOG_FILE" --files-from="$FILES_FILE" --exclude-from="$EXCLUDE_FILE" "$FROM_FOLDER" "$TO_FOLDER"
```

To run it daily as soon as the computer is online, put it into the folder :

```
~
| .config/
| | anacron.daily/
| | | on_line/
```
