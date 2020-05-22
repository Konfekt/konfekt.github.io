---
layout: post
title: "Fast and Easy batch conversion of Videos to WebM"
date: 2020-05-08
categories: ffmpeg webm mp4 convert batch parallel bash shell script
comments: true
---

The `WebM` video format saves a lot of space with little loss in quality in comparison to other high quality formats such as `MP4`.
The following shell scripts convert a batch of video files, say from the `MP4` format, into the `WebM` format as fast and conveniently as possible, preferably using [GNU parallel](https://www.gnu.org/software/parallel/).

Customize the following parameters to your needs:

- `-crf 10` : the Constant Rate Factor (crf) sets video quality from `0` (lossless) to `63` (worst);
    default is 23.
- `-c:a/v ...` : transform the audio/video data using the codec `...`, for example, `libvpx`.
- `-b:a/v ...` : set the audio/video bitrate to `...`, for example, `1M` for one megabyte.
- `-cpu-used` : from 0 (default) to 5
    Use 1 or 2 for faster but worse encoding
- `-row-mt = 1` : enable for better multi-threading starting from `libvpx 1.6`.

See Google's (one of the driving forces behind the `VPX` codecs) [recommendations](https://developers.google.com/media/vp9/settings/vod/) for sensible bitrates and quality factors for basic uses such as videos downloaded and watched on a mobile phone.
The [official site](https://trac.ffmpeg.org/wiki/Encode/VP9), [these notes](https://github.com/Kagami/webm.py/wiki/Notes-on-encoding-settings) and this [blog entry](https://blog.programster.org/VP9-encoding) for recommendation on encoding by `ffmpeg` using the `libvpx` codec.
Finally, [these notes](https://notepad.patheticcockroach.com/4263/a-brief-tutorial-to-encode-in-x265-and-opus-using-ffmpeg/) discuss the audio encoding.

# Single Pass

This script converts in a single pass, faster, but accepting suboptimal quality:

```sh
#!/usr/bin/env bash

# Single pass for faster conversion but not optimal quality:

convert() {
  echo "Started processing $1 ..."
  cpu_cores="$(getconf _NPROCESSORS_ONL)"
  ffmpeg \
    -y -loglevel error \
    -i "$1" \
    -threads "${cpu_cores:-1}" -cpu-used 1 -quality good \
    -tile-columns 2 -frame-parallel 0 -auto-alt-ref 1 -lag-in-frames 25 \
    -c:v libvpx-vp9 -b:v 750k -minrate 375k -maxrate 1088k -crf 33 \
    -c:a libopus -ac 2 -b:a 48k -vbr on -compression_level 10 -frame_duration 40 -application audio \
    -movflags faststart \
  "${1%.[^.]*}".webm
  echo "... finished processing $1."
  } && export -f convert

if command -v parallel >/dev/null 2>&1; then
    parallel convert {} {.}.webm ::: "$@"
else
  for f in "$@"; do
    convert "$f" "{f%.[^.]*}".webm
  done
fi
```

# Double Pass

This script converts in two passes, slower, but achieving optimal quality:

```sh
#!/usr/bin/env bash

# Double pass for optimal quality but slower conversion:

convert() {
cpu_cores="$(getconf _NPROCESSORS_ONL)"
echo "Started first pass processing $1 ..."
ffmpeg \
  -y -loglevel error \
  -i "$1" \
  -pass 1 -speed 4 \
  -threads "${cpu_cores:-1}" -cpu-used 1 -quality good \
  -tile-columns 4 -frame-parallel 0 -auto-alt-ref 1 -lag-in-frames 25 \
  -c:v libvpx-vp9 -b:v 750k -minrate 375k -maxrate 1088k -crf 33 \
  -c:a libopus -ac 2 -b:a 48k -vbr on -compression_level 10 -frame_duration 40 -application audio \
  -movflags faststart \
  -f webm \
  /dev/null &&
  echo "... finished first and started second pass processing $1 ..."
ffmpeg \
  -y -loglevel error \
  -i "$1" \
  -pass 2 -speed 1 \
  -threads "${cpu_cores:-1}" -cpu-used 1 -quality good \
  -tile-columns 4 -frame-parallel 0 -auto-alt-ref 1 -lag-in-frames 25 \
  -c:v libvpx-vp9 -b:v 750k -minrate 375k -maxrate 1088k -crf 33 \
  -c:a libopus -ac 2 -b:a 48k -vbr on -compression_level 10 -frame_duration 40 -application audio \
  -movflags faststart \
  "${1%.[^.]*}".webm &&
  echo "... finished second and last pass processing $1."
} && export -f convert

if command -v parallel >/dev/null 2>&1; then
    parallel convert {} {.}.webm ::: "$@"
else
  for f in "$@"; do
    convert "$f" "${f%.[^.]*}".webm
  done
fi
```

