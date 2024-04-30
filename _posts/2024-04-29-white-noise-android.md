---
layout: post
title: "White Noise under Android to Remedy Swallowed Syllables"
date: 2024-04-29
categories: AIMP Vanilla Music white background noise Android
comments: true
---

# Swallowed Initial Syllables due to Overly Eager Bluetooth Power-Saving

Some [Bluetooth headsets](https://old.reddit.com/r/audible/comments/cc0llh/bluetooth_1st_word_of_every_sentence_is_missing/), like [Otium's Audio](https://www.otiumobile.com/products/otium-wireless-bluetooth-sports-headphones-in-ear-earbuds-sweatproof-earphones-stereo-with-mic-bass-noise-cancelling-bluetooth-v4-1-for-iphone-android-smartphones), go into power-saving mode as soon as no sound is played.
This happens in every pause in a dialogue, frequent in language learning courses such as [Pimsleur's](https://it.wikipedia.org/wiki/Sistema_d%27apprendimento_linguistico_Pimsleur) and causes the first few syllables of every sentence following such a pause to be swallowed.

# Work Around

To work around this on Android by keeping your bluetooth connection alive via permanent background sounds:

1. download some [white](https://youtu.be/watch?v=E3_eIwDWS_c) [noise](https://youtu.be/watch?v=qU8o3_T5y5M) sound file onto your phone, for example,
    - with [yt-dlp](https://github.com/yt-dlp/yt-dlp) onto your computer and then onto your phone,
    - or directly with [New Pipe](https://github.com/TeamNewPipe/NewPipe/releases/)
1. install [AIMP](https://play.google.com/store/apps/details?id=com.aimp.player),
1. start `AIMP` and open the sound file with it (optionally click on repeat in the lower right corner),
1. lower the sound file volume by, say -50 decibel, by clicking

    1. on the equalizer symbol in the upper right corner,
    1. the burger menu in the lower right corner and,
    1. clicking `normalize volume` twice.

1. go to AIMP's settings page (by clicking the gearwheel), 

    - In Sound, Audio Focus, disable the focus option for playing.
    - In Playback, Startup, set it to retake where left

1. All set.
   Now just open `AIMP` every time you need background noise to keep your Bluetooth connection alive, say, when you listen to some dialogue with pauses.

While `AIMP` is a great player, here it fell victim to its power as other players do not offer the same features, in particular, the missing focus options.
Other white noise apps, like [Noice](https://trynoice.com/) have this enabled by default, but while open source, require monthly fees to use local sound files.

# Companion Players

Good companions music player apps, say to play the (formerly interrupted) dialogues, are [Vanilla Music](https://play.google.com/store/apps/details?id=ch.blinkenlights.android.vanilla) and [Neutron MP](https://neutroncode.com/).
