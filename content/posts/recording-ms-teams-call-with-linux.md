---
title: "Recording Ms Teams Call With Linux"
date: 2022-02-07T14:54:57+01:00
draft: false
tags:
- Linux
- MS-Teams
---

**This article explains how to record an audio stream on a Linux system running PulseAudio.**

You can record both input streams (microphones) and output streams (whatever you are hearing in your headphones). However, each stream goes to a separate file. If you want to record, say, a MS Teams conversation, record both streams and merge them afterwards in audacity.

First, find out the PulseAudio name of the stream you want to record, you can even use a bluetooth headset met USB dongle, it doesn't matter as long you are using PulseAudio.

### Record input (microphone)
If you want to record a microphone, run:
```bash
pacmd list-sources | egrep '^\s+name: .*alsa_input'
```

Here is my output (only one device connected, there could be more devices):
```bash
name: <alsa_input.usb-0b0e_Jabra_Link_380_305075464F92-00.analog-mono>>
```

### Record output (speakers)
If you want to record an output (e.g. the person talking to you on MS Teams), similarly run:
```bash
pacmd list-sources | egrep '^\s+name:.*\.monitor'
```

Once you know the name of the stream you want to record, run:
```bash
parecord --channels=1 -d <STREAM_NAME> filename.wav
```

For instance, if I wanted to record my jabra output, I would run:
```bash
parecord --channels=1 -d alsa_output.usb-0b0e_Jabra_Link_380_305075464F92-00.iec958-stereo.monitor filename.wav
```
When you are done recording, terminate the process by pressing Ctrl-C.

_--channels=1 forces your recording to mono, which is usually what you want._

_**Note**_: The recording is not compressed, so make sure you have enough disk space. 
Assuming the standard sample rate (44kHz) and sample size (16 bit), you will need 320 MB per hour of recording.

` `  
Credits to _Roman Cheplyaka_ [https://ro-che.info/articles/2017-07-21-record-audio-linux](https://ro-che.info/articles/2017-07-21-record-audio-linux)
