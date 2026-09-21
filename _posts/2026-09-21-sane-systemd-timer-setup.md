---
layout: post
title: "A sane systemd timer setup"
date: 2026-09-21
categories: systemd timers linux cron anacron
comments: true
---

This is an AI generated update to the [2016 anacron setup]({% post_url 2016-12-11-sane-cron-setup %}) using a simplified systemd timers setup: 
Place each periodic job in a folder for its frequency and for the power and network conditions under which it should run.
systemd user timers then execute those folders:
a missed run after the machine was off is caught up, and a run that needs AC power or the network waits for the next hourly tick.

The folder layout is the same as in the [2016 anacron setup]({% post_url 2016-12-11-sane-cron-setup %}).

# Aim

Run the executables in

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
| | anacron.hourly/
| | | on/
| | | on_ac/
| | | on_line/
| | | on_line_ac/
```

Recurrence names and due-specs live in `~/.config/anacron.periods`. The shipped table is `daily` (calendar day), `weekly` (7d), `monthly` (calendar month), and `hourly` (1h). State folders:

- `on`: unconditionally
- `on_ac`: on AC power, for example a file index
- `on_line`: online, for example a remote backup
- `on_line_ac`: online and on AC power, for example software updates

# How a tick works

```
anacron@daily-on_ac.timer
        |  OnCalendar=hourly
        |  Persistent=true
        v
anacron@daily-on_ac.service
        v
run-anacron-folder.sh daily-on_ac
        |
        +-- missing folder          -> skip
        +-- battery (on_ac*)        -> skip; next hour
        +-- offline (on_line*)      -> skip; next hour
        +-- stamp still in period   -> skip
        +-- run-parts --exit-on-error ~/.config/anacron.daily/on_ac
        +-- write stamp (only if run-parts exits 0)
        +-- on failure: anacron-failure@daily-on_ac.service
```

`Persistent=true` starts a missed hourly tick after boot or resume. The stamp file `$XDG_STATE_HOME/anacron/<period>.<state>` is `YYYYMMDD` for day/month specs (anacron format) and `YYYYMMDDHHMMSS` for `<N>h`.

# Prerequisites

- `run-parts` on `PATH` (Debian `run-parts`, or [run-parts.sh](https://github.com/wolfbox/run-parts))
- `systemd-ac-power` for the AC check
- `nm-online` and `nc` for the online check (`github.com:22`, so a captive portal that only answers on HTTP does not count as online)

Jobs log to the user journal. `Nice=19`, `CPUSchedulingPolicy=idle`, and `IOSchedulingClass=best-effort` / `IOSchedulingPriority=7` on the service replace `chrt --idle` and `ionice`.

# Periods

`~/.config/anacron.periods` is the recurrence table. Due-specs: `calendar-day`, `calendar-month`, `<N>d`, `<N>h`.

```
# period  due-spec
daily     calendar-day
weekly    7d
monthly   calendar-month
hourly    1h
```

Add a period with one row, folders `~/.config/anacron.<period>/{on,on_ac,on_line,on_line_ac}`, and `systemctl --user enable --now anacron@<period>-<state>.timer`. The systemd templates stay unchanged. Period names are `[a-z][a-z0-9]*` (no hyphen: the instance split is `<period>-<state>`).

# Units

Two template files cover every period×state. The instance name is `<period>-<state>` and maps to `~/.config/anacron.<period>/<state>`:

| Instance | Folder | AC | Online | Period |
| --- | --- | --- | --- | --- |
| `daily-on` | `anacron.daily/on` | | | next calendar day |
| `daily-on_ac` | `anacron.daily/on_ac` | yes | | next calendar day |
| `daily-on_line` | `anacron.daily/on_line` | | yes | next calendar day |
| `daily-on_line_ac` | `anacron.daily/on_line_ac` | yes | yes | next calendar day |
| `weekly-on` | `anacron.weekly/on` | | | 7 days |
| `weekly-on_ac` | `anacron.weekly/on_ac` | yes | | 7 days |
| `weekly-on_line` | `anacron.weekly/on_line` | | yes | 7 days |
| `weekly-on_line_ac` | `anacron.weekly/on_line_ac` | yes | yes | 7 days |
| `monthly-on` | `anacron.monthly/on` | | | next calendar month |
| `monthly-on_ac` | `anacron.monthly/on_ac` | yes | | next calendar month |
| `monthly-on_line` | `anacron.monthly/on_line` | | yes | next calendar month |
| `monthly-on_line_ac` | `anacron.monthly/on_line_ac` | yes | yes | next calendar month |

`~/.config/systemd/user/anacron@.service`:

```ini
[Unit]
Description=Run anacron folder %i
Documentation=https://konfekt.github.io/blog/2026/09/21/sane-systemd-timer-setup
After=default.target
OnFailure=anacron-failure@%i.service

[Service]
Type=oneshot
Nice=19
CPUSchedulingPolicy=idle
IOSchedulingClass=best-effort
IOSchedulingPriority=7
TimeoutStartSec=infinity
WorkingDirectory=%h
Environment=CRONJOB=1
Environment=DISPLAY=:0
Environment=PATH=%h/bin:%h/.local/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
SyslogIdentifier=anacron-%i
ExecStart=%h/.config/bin/run-anacron-folder.sh %i
```

`~/.config/systemd/user/anacron-failure@.service`:

```ini
[Unit]
Description=Notify that anacron folder %i failed
Documentation=https://konfekt.github.io/blog/2026/09/21/sane-systemd-timer-setup
After=default.target

[Service]
Type=oneshot
Environment=DISPLAY=:0
ExecStart=%h/.config/bin/anacron-failure-notify.sh %i
```

`run-parts --exit-on-error` leaves the stamp unchanged when a script fails, so the next hourly tick retries. `OnFailure=` starts `notify-send` and `mailx` with the journal excerpt (`~/.config/bin/anacron-failure-notify.sh`).

`~/.config/systemd/user/anacron@.timer`:

```ini
[Unit]
Description=Hourly retry for anacron folder %i
Documentation=https://konfekt.github.io/blog/2026/09/21/sane-systemd-timer-setup

[Timer]
OnBootSec=5min
OnCalendar=hourly
RandomizedDelaySec=5min
Persistent=true
AccuracySec=1min

[Install]
WantedBy=timers.target
```

`TimeoutStartSec=infinity` is required: monthly update jobs can run longer than the default 90s start timeout. `PATH` puts `%h/bin` first so linger-at-boot matches the old user crontab (environment.d otherwise puts those dirs last).

`~/.config/bin/run-anacron-folder.sh` (mark executable):

```sh
#!/usr/bin/env bash
# Run executables in ~/.config/anacron.<period>/<state> when the period is due
# and the state's AC/online conditions hold.
# Periods and due-specs: $XDG_CONFIG_HOME/anacron.periods

set -o errexit -o nounset -o pipefail
[[ "${TRACE:-0}" == "1" ]] && set -o xtrace

SECONDS_PER_HOUR=3600
ONLINE_HOST=github.com
ONLINE_PORT=22
NM_ONLINE_TIMEOUT_SEC=10
NC_CONNECT_TIMEOUT_SEC=3

: "${HOME:=$(getent passwd "${USER:-$(id -un)}" | cut -d: -f6)}"
: "${XDG_CONFIG_HOME:=$HOME/.config}"
: "${XDG_STATE_HOME:=$HOME/.local/state}"

PERIODS_FILE="$XDG_CONFIG_HOME/anacron.periods"
declare -A PERIOD_DUE=()

load_periods() {
  local period spec
  if [[ ! -f $PERIODS_FILE ]]; then
    echo "missing periods file ${PERIODS_FILE}" >&2
    exit 2
  fi
  while read -r period spec _; do
    [[ -z ${period:-} || $period == \#* ]] && continue
    if [[ ! $period =~ ^[a-z][a-z0-9]*$ ]]; then
      echo "invalid period name: ${period}" >&2
      exit 2
    fi
    if [[ $spec != calendar-day && $spec != calendar-month && ! $spec =~ ^[0-9]+[dh]$ ]]; then
      echo "invalid due spec for ${period}: ${spec}" >&2
      exit 2
    fi
    PERIOD_DUE[$period]=$spec
  done <"$PERIODS_FILE"
  if [[ ${#PERIOD_DUE[@]} -eq 0 ]]; then
    echo "no periods in ${PERIODS_FILE}" >&2
    exit 2
  fi
}

usage() {
  local p
  echo "usage: $0 <period>-{on|on_ac|on_line|on_line_ac}" >&2
  echo "periods:" >&2
  for p in "${!PERIOD_DUE[@]}"; do
    echo "  ${p}  ${PERIOD_DUE[$p]}" >&2
  done
  exit 2
}

load_periods

INSTANCE="${1-}"
if [[ $INSTANCE != *-* ]]; then
  usage
fi
PERIOD="${INSTANCE%%-*}"
STATE="${INSTANCE#*-}"
if [[ -z ${PERIOD_DUE[$PERIOD]+x} ]]; then
  usage
fi
case "$STATE" in
  on | on_ac | on_line | on_line_ac) ;;
  *)
    usage
    ;;
esac
DUE_SPEC="${PERIOD_DUE[$PERIOD]}"

FOLDER="$XDG_CONFIG_HOME/anacron.${PERIOD}/${STATE}"
STAMP_DIR="$XDG_STATE_HOME/anacron"
STAMP_FILE="$STAMP_DIR/${PERIOD}.${STATE}"

if [[ ! -d $FOLDER ]]; then
  echo "skip ${INSTANCE}: missing folder ${FOLDER}" >&2
  exit 0
fi

if [[ $STATE == on_ac || $STATE == on_line_ac ]] && ! systemd-ac-power; then
  echo "skip ${INSTANCE}: on battery" >&2
  exit 0
fi

if [[ $STATE == on_line || $STATE == on_line_ac ]]; then
  if ! nm-online --timeout="$NM_ONLINE_TIMEOUT_SEC" --quiet; then
    echo "skip ${INSTANCE}: offline (nm-online)" >&2
    exit 0
  fi
  if ! nc -zw"$NC_CONNECT_TIMEOUT_SEC" "$ONLINE_HOST" "$ONLINE_PORT"; then
    echo "skip ${INSTANCE}: offline (${ONLINE_HOST}:${ONLINE_PORT})" >&2
    exit 0
  fi
fi

stamp_date_iso() {
  local last=$1
  [[ ${#last} -ge 8 ]] || return 1
  echo "${last:0:4}-${last:4:2}-${last:6:2}"
}

stamp_to_epoch() {
  local last=$1 iso
  case ${#last} in
    8)
      iso="$(stamp_date_iso "$last")"
      date -d "$iso" +%s
      ;;
    14)
      iso="$(stamp_date_iso "$last") ${last:8:2}:${last:10:2}:${last:12:2}"
      date -d "$iso" +%s
      ;;
    *)
      return 1
      ;;
  esac
}

period_is_due() {
  local last=$1 spec=$2
  local today last_iso due_on n last_epoch now
  today=$(date +%Y%m%d)
  case "$spec" in
    calendar-day)
      [[ ${#last} -ge 8 && ${last:0:8} < $today ]]
      ;;
    calendar-month)
      [[ ${#last} -ge 6 && ${last:0:6} < $(date +%Y%m) ]]
      ;;
    *d)
      n=${spec%d}
      last_iso=$(stamp_date_iso "$last") || return 0
      date -d "$last_iso" >/dev/null 2>&1 || return 0
      due_on=$(date -d "${last_iso} +${n} days" +%Y%m%d)
      [[ $today -ge $due_on ]]
      ;;
    *h)
      n=${spec%h}
      last_epoch=$(stamp_to_epoch "$last") || return 0
      now=$(date +%s)
      [[ $now -ge $((last_epoch + n * SECONDS_PER_HOUR)) ]]
      ;;
  esac
}

stamp_now() {
  case "$DUE_SPEC" in
    *h) date +%Y%m%d%H%M%S ;;
    *) date +%Y%m%d ;;
  esac
}

if [[ -f $STAMP_FILE ]]; then
  last=$(tr -d '[:space:]' <"$STAMP_FILE")
  if [[ $last =~ ^[0-9]{8}([0-9]{6})?$ ]] && ! period_is_due "$last" "$DUE_SPEC"; then
    echo "skip ${INSTANCE}: not due (stamp ${last})" >&2
    exit 0
  fi
fi

mkdir --parents --verbose "$STAMP_DIR"
run-parts --regex='^[a-zA-Z0-9_\.-]+$' --exit-on-error -v -- "$FOLDER"
stamp_now >"$STAMP_FILE"
```

# Enable the timers

Linger keeps the user systemd instance running after logout, so the timers keep firing while the machine is on:

```sh
loginctl enable-linger "$USER"
```

Install and start every instance listed in `anacron.periods`:

```sh
systemctl --user daemon-reload
while read -r period spec _; do
  [[ -z ${period:-} || $period == \#* ]] && continue
  for state in on on_ac on_line on_line_ac; do
    systemctl --user enable --now "anacron@${period}-${state}.timer"
  done
done <"${XDG_CONFIG_HOME:-$HOME/.config}/anacron.periods"
```

`Persistent=true` on first enable starts every instance that has no timer stamp yet. The runner then skips folders whose anacron stamp is still current, and runs folders that are due. Drop a script into a missing folder later; the next hourly tick picks it up.

Unit files live in `~/.config/systemd/user/` and are git-tracked. Enablement for these timers is tracked as relative symlinks in `systemd/user/timers.target.wants/` (other `*.wants/` stay untracked because they point at `/usr/lib`). After a clone: `systemctl --user daemon-reload`. `loginctl enable-linger` is per machine.

Install a comment-only user crontab. The timers are then the only scheduler:

```sh
crontab ~/.config/crontab
```

with `~/.config/crontab` containing only comments, for example:

```
# Periodic jobs: systemd user timers anacron@.timer (see ~/.config/systemd/user/).
# Installing this file clears the user crontab: crontab ~/.config/crontab
```

# Logs and one-shot runs

```sh
systemctl --user list-timers 'anacron@*'
journalctl --user -u anacron@daily-on_line.service
systemctl --user start anacron@weekly-on_ac.service
```

`start` on the service runs the folder now if the period is due and the conditions hold. The stamp still prevents a second run in the same period.

# SSH-backed jobs

The service sets `CRONJOB=1` and `DISPLAY=:0`. Backup jobs that call [`sshstart`](https://github.com/vaeth/sshstart) can use `sshstart -eGA` when `CRONJOB=1` and reuse the login gpg-agent SSH cache. `environment.d` can pin `SSH_AUTH_SOCK` to `gpg-agent`'s SSH socket.

# Example

A daily backup as soon as the machine is online goes in `~/.config/anacron.daily/on_line/`. A shorter form of the [backup2cloud.sh](https://github.com/Konfekt/backup2cloud.sh) job:

```sh
#!/bin/bash

set -o xtrace -o errtrace -o errexit -o nounset -o pipefail

[ -z "$HOSTNAME" ] && HOSTNAME="$(uname -n)"
if command -v sshstart >/dev/null 2>&1; then
  if [ "${CRONJOB:-0}" = 1 ]; then
    eval "$(sshstart -eGA)"
  else
    eval "$(sshstart -eA)"
  fi
fi

FROM_FOLDER=$HOME
TO_FOLDER="$USER@rsync.server.com:/users/$USER"

FILES_FILE=$XDG_CONFIG_HOME/backup/files
EXCLUDE_FILE=$XDG_CONFIG_HOME/backup/exclude
LOG_FILE=$XDG_CACHE_HOME/backup/cloud/log

RSYNC_BKP_ARGS="--recursive --archive --hard-links --ignore-errors --modify-window=1 --delete --compress --partial --human-readable --info=progress2 "
SSH_ARGS="-v -P"

mkdir --parents "$(dirname "${LOG_FILE}")"
rsync $RSYNC_BKP_ARGS --rsh="ssh $SSH_ARGS" --log-file="$LOG_FILE" --files-from="$FILES_FILE" --exclude-from="$EXCLUDE_FILE" "$FROM_FOLDER" "$TO_FOLDER"
```
