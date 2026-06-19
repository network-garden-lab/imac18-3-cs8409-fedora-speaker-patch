# Fedora 44 / kernel 7.0.12 test notes for iMac18,3 CS8409 speaker patch

## Status

Experimental test patch for Fedora 44 kernel 7.0.12-201.fc44.x86_64 on iMac18,3.

This patch is based on the existing Fedora 6.19.12 iMac18,3 CS8409 internal speaker patch, adapted to the kernel 7.0.12 CS8409 source layout.

## Tested environment

- Machine: iMac 2017 Retina 5K 27-inch
- Model Identifier: iMac18,3
- Codec: Cirrus Logic CS8409
- Codec subsystem ID: 106b:1000
- OS: Fedora Linux 44
- Kernel: 7.0.12-201.fc44.x86_64
- PipeWire: 1.6.6
- Secure Boot: not supported on this system

## Patch file

- patches/imac18-3-cs8409-speaker-fedora-7.0.12.patch

The patch applies cleanly to a clean linux-7.0.12 source tree in my test environment.

## Confirmed working

The following were confirmed on my iMac18,3:

- custom snd-hda-codec-cs8409.ko loads from updates/cirrus
- internal analog-stereo is available as the default sink
- direct ALSA playback works
  - speaker-test -D hw:0,0 -c 2 -r 44100
- PipeWire/default playback works
  - speaker-test -D default -c 2 -r 44100
  - speaker-test -D default -c 2 -r 48000
- Firefox / YouTube playback works through the internal speakers
- pause / resume works
- GNOME GUI left-right balance affects playback
- switching away from analog-stereo and returning to analog-stereo can recover playback

## Important behavior

Playback cleanup still skips clearing speaker DAC stream assignments:

- speaker DAC 0x02 / 0x03 stream clear skipped

This behavior appears to be important for keeping internal speaker playback stable.

## Known limitations

Still not fixed:

- headphone switching
- internal microphone input
- external microphone input
- 4ch / 2.1ch profiles
- other iMac models
- other kernel versions

Other output/profile switching can still disturb the audio state, similar to the previous Fedora 6.19.12 test environment.

## Known warnings

The kernel log still shows warnings such as:

- out of range cmd 0:1:7f0:3000b7
- temporary CS42L83 I2C read/write failures

In my test environment, playback continues to work after these warnings, but they should be considered known issues for now.

## Notes

This is not a final upstream-quality fix. It is an experimental test patch intended to help other iMac18,3 users reproduce and validate the internal speaker playback behavior on Fedora 44 / kernel 7.0.12.
