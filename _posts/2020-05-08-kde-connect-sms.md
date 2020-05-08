---
layout: post
title: "Sending SMS with KDE Connect"
date: 2020-05-08
categories: KDE Connect send SMS bash shell script
comments: true
---

[KDE Connect](https://community.kde.org/KDEConnect) is a [useful](https://userbase.kde.org/KDE_Connect/Tutorials/Useful_commands) application to let your smart phone and computer interact:
for example, to use the smart phone as remote control, share files and the clipboard,  ..., and send SMS from the computer!
However, the last function is yet unreleased in a graphical user interface, and from the command line many switches have to be typed.
The following shell script, name it `kc-sms`, does away with these by using the first associated smart phone to show up.
To send the SMS `How are you?` to the number `8288880000`, run `kc-sms 8288880000 'How are you?'`.

```sh
#!/bin/sh

if ! command -v kdeconnect-cli >/dev/null 2>&1; then
  echo "kde-connect-cli not found! Please install KDE Connect."
  exit 1
fi

if [ $# -eq 0 ] || [ "$1" = -h ] || [ "$1" = --help ]; then
  echo "kc-sms - send SMS using KDE Connect on the command line"
  echo "usage: kc-sms <Phone Number> <Message> [Parameters]"
  echo "example: kc-sms 01721234567 'How are you?' --refresh"
  exit 2
fi

name="$(kdeconnect-cli --list-available --name-only | head -n 1)"
if kdeconnect-cli --name "$name" --destination "$1" --send-sms "$2" "${@:3}"; then
  echo "successfully sent SMS to $1 by phone $name"
else
  echo "error sending SMS to $1 by phone $name"
fi
```

