---
layout: post
title: "Dimming (and brightening external) screens using redshift hooks"
date: 2023-05-22
categories: dim external screen monitor redshift hook ddc
comments: true
---


[Redshift](http://jonls.dk/redshift/) shifts the color palette of the screen towards the red spectrum at nighttime (and undoes it during daylight).
To dim as well the (external) screens on Linux *hooks* are called for; citing `man redshift`:

> Executables (e.g. scripts) placed in folder ~/.config/redshift/hooks will be run when a certain event happens.
    The first parameter to the script indicates the event and further parameters may indicate more details about the event.
    The event period-changed is indicated when the period changes (night, daytime, transition).
    The second parameter is the old period and the third is the new period.
    The event is also signaled when Redshift starts up with the old period set to none.
    A simple script to handle these events can be written like this:
>
    #!/bin/sh
    case $1 in
		    period-changed)
		        exec notify-send "Redshift" "Period changed to $3"
    esac

# Solution

The following script (named, say, `brightness.sh`, marked executable by `chmod a+x brightness.sh` and put into `~/.config/redshift/hooks`) sets the brightness levels `$brightness_day/transition/night` on the respective period changes of

- the laptop screen via [xbacklight](https://gitlab.com/wavexx/acpilight/) and
- every external screen via [ddcutil](https://www.ddcutil.com/command_setvcp/)


```sh
#!/bin/sh

# Set brightness values from 1 to 100 for each status
brightness_day="90"
brightness_transition="55"
brightness_night="35"

# In case computer wakes up and older brightness.sh process running
killall --user "$USER" --older-than 1h --exact 'brightness.sh'

# Adjust brightness of internal screen via xbacklight
if command -v brightnessctl >/dev/null 2>&1; then
  set_brightness_internal() { brightnessctl set "$1"%; }
elif command -v brightnessctl; then
  # Set fade time to one minute
  fade_time=60000
  steps_per_second=15
  steps=$(("$fade_time" * "$steps_per_second" / 1000))
  set_brightness_internal() { xbacklight -set "$1" -time $fade_time -step "$steps"; }
else
  set_brightness_internal() { return; }
fi

# Adjust brightness of external screen via ddcutil
if command -v ddcutil >/dev/null 2>&1; then
  set_brightness_external() {
    if [ "$(LC_ALL=C xrandr | grep -wc connected)" -gt 1 ]; then
      buses="$(ddcutil detect --terse | awk '
      BEGIN { RS=""; FS="\n" }
      /^Display/ {
      for (i=1; i<=NF; i++) {
        if ($i ~ /I2C bus:/) {
          split($i, bus, "/dev/i2c-")
          bus_number = bus[2]
        }
      }
      print bus_number
      }')"
      for bus in $buses; do
        ddcutil setvcp 0x10 "$1" --bus="$bus"
      done
    fi
  }
else
  set_brightness_external() { return; }
fi

case $1 in
  period-changed)
    case $3 in
      night)
        set_brightness_internal $brightness_night
        set_brightness_external $brightness_night
        ;;
      transition)
        set_brightness_internal $brightness_transition
        set_brightness_external $brightness_transition
        ;;
      daytime)
        set_brightness_internal $brightness_day
        set_brightness_external $brightness_day
        ;;
    esac
    ;;
esac
```

# Caveat

As of now redshift's hooks do not run on many distributions due to [a faulty set-up of AppArmor inhibiting these](https://github.com/jonls/redshift/issues/850).
There are three options:

- use [GammaStep](https://gitlab.com/chinstrap/gammastep/), forked from Redshift, to overcome such shortcomings, or
- wait for [this fix](https://github.com/jonls/redshift/pull/864/files) to be included in Redshift and tickle down on Debian and other major distributions,
- fix it yourself by adding

    ```
    owner @{HOME}/.config/redshift.conf r,
    /etc/passwd* r,
    ```

    to `/etc/apparmor.d/local/usr.bin.redshift` by `sudoedit /etc/apparmor.d/local/usr.bin.redshift` and running `sudo systemctl reload apparmor.service`.
