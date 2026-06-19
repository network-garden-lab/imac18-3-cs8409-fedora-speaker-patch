# iMac18,3 CS8409 Built-in Speaker Patch for Fedora

## Fedora 44 / kernel 7.0.12 test patch

An experimental test patch for Fedora 44 / kernel 7.0.12-201.fc44.x86_64 has been added.

- Patch: `patches/imac18-3-cs8409-speaker-fedora-7.0.12.patch`
- Test notes: `FEDORA_44_KERNEL_7.0.12_TEST_NOTES.md`

On my iMac18,3, this 7.0.12 test patch enables internal speaker playback through normal desktop audio paths, including PipeWire and Firefox / YouTube playback.

This does not replace the Fedora 6.19.12 patch. It is provided as an additional experimental test patch for Fedora 44 / kernel 7.0.12.

Known limitations still remain:

- headphone switching is not fixed
- microphone input is not fixed
- other output/profile switching may still disturb the audio state
- kernel log warnings such as `out of range cmd` and temporary CS42L83 I2C read/write failures may still appear

### Fedora 44 / kernel 7.0.12 usage

For Fedora 44 / kernel 7.0.12-201.fc44.x86_64, use the 7.0.12-specific patch:

    patch -p1 < patches/imac18-3-cs8409-speaker-fedora-7.0.12.patch

The general build and install flow is the same as the Fedora 6.19.12 instructions, but make sure you are working with a matching Fedora 44 / kernel 7.0.12 source tree and the matching kernel build files for your running kernel.

Do not apply the Fedora 6.19.12 patch directly to kernel 7.0.12. Use the dedicated 7.0.12 test patch instead.

## Overview

This repository provides experimental CS8409 patches and test notes for enabling built-in speaker playback on the 2017 27-inch iMac 5K / iMac18,3 running Fedora. The original confirmed patch targets Fedora kernel 6.19.12, and an additional experimental test patch is available for Fedora 44 / kernel 7.0.12.

The iMac18,3 audio system uses a CS8409 HDA codec together with Apple-specific external amplifier control. On Fedora, the internal HDA audio path can work, but the built-in speakers may remain silent unless the external amplifier initialization sequence is executed.

This patch adds an iMac18,3-specific CS8409 fixup that initializes the required CS42L83 / TAS576 amplifier path and keeps the built-in speaker playback route stable.

Japanese supplemental notes are available in [README_JA.md](README_JA.md).

## Related article

A Japanese technical report is available on Qiita.  
Non-Japanese readers may use browser translation if needed.

- [FedoraでiMac 2017 Retina 5K（iMac18,3）の内蔵スピーカーを鳴らした記録](https://qiita.com/network-garden-lab/items/45c2c7a2f270b0828271)

## Experimental status

This patch is experimental and has only been tested on one iMac18,3 system.

It is shared as a working result and investigation record, not as a complete or upstream-ready solution.

Please read the known limitations before testing, especially the notes about sound profile / configuration switching.

## Quick start

Apply the patch to an unmodified Linux 6.19.12 source tree:

    patch -p1 < /path/to/imac18-3-cs8409-speaker-fedora-6.19.12.patch

Build the affected objects:

    make sound/hda/codecs/cirrus/cs8409.o
    make sound/hda/codecs/cirrus/cs8409-tables.o

See the sections below for details, limitations, and feedback information.

## Target environment

Tested environment:

- Machine: iMac 2017 27-inch 5K
- Model ID: iMac18,3
- OS: Fedora
- Kernel source: Linux 6.19.12
- Audio path: CS8409 / CS42L83 / TAS576
- Target output: Built-in speakers

## What works

Confirmed in the tested environment:

- Built-in speaker playback works
- YouTube / normal desktop playback works
- ALSA / PipeWire analog-stereo output works
- The patch applies cleanly to an unmodified linux-6.19.12 source tree
- cs8409.o builds successfully
- cs8409-tables.o builds successfully

## What is not covered

This patch currently focuses only on built-in speaker playback.

Not covered:

- Headphone switching
- Internal microphone input
- External microphone input
- Apple-equivalent speaker tuning
- Other iMac / MacBook models
- Other kernel versions
- Ubuntu patch replacement

## Known limitations

Built-in speaker playback is stable enough for normal use in the tested environment.

However, the internal speaker routing may not be identical to Apple's original speaker tuning.

### Sound profile / configuration switching

> [!WARNING]
> Changing the sound profile / configuration in the desktop sound settings may temporarily disturb or break playback.
>
> The previous volume state may not be preserved when switching profiles. After switching back to Analog Stereo Output, playback may resume with unexpectedly loud noise.
>
> Please lower the volume before testing profile changes.

> [!NOTE]
> This patch is intended for the analog-stereo built-in speaker output path.
>
> Some desktop sound profiles such as 2.1ch or 4.0ch may appear in PipeWire / desktop sound settings, but they do not currently correspond to a verified or supported iMac18,3 internal speaker mapping in this patch.
>
> Similar profile options were also observed on a working Ubuntu 22.04 setup, so their presence is not specific to this patch.

In the tested environment, switching the profile back to Analog Stereo Output restored playback without requiring a manual driver reset.

This behavior is documented as an observed limitation. It may also be useful for further investigation by others.

At the center balance position, the sound is usable in normal listening and may not feel obviously wrong to most users. However, when changing the left/right balance in the desktop sound settings, or by using pactl, the balance does not behave exactly like normal left/right stereo.

Observed behavior:

- Left balance:
  - Higher frequencies sound stronger
  - Upper speaker side seems more prominent

- Center balance:
  - Usable for normal listening
  - Slight tonal / spatial imbalance may be noticeable if listening carefully

- Right balance:
  - Lower frequencies sound stronger
  - Lower speaker side seems more prominent
  - Sound may feel more muffled

This suggests that the current routing may still be affected by the iMac internal high/low or upper/lower speaker paths rather than perfectly reproducing Apple's original stereo speaker mapping.

This is documented here for transparency.

## Patch file

Patch:

    patches/imac18-3-cs8409-speaker-fedora-6.19.12.patch

The patch modifies only these files:

    sound/hda/codecs/cirrus/cs8409.c
    sound/hda/codecs/cirrus/cs8409.h
    sound/hda/codecs/cirrus/cs8409-tables.c

## Applying the patch

From the root of an unmodified Linux 6.19.12 source tree:

    patch -p1 < /path/to/imac18-3-cs8409-speaker-fedora-6.19.12.patch

Expected result:

    patching file sound/hda/codecs/cirrus/cs8409.c
    patching file sound/hda/codecs/cirrus/cs8409.h
    patching file sound/hda/codecs/cirrus/cs8409-tables.c

## Build check

A minimal compile check can be done with:

    make sound/hda/codecs/cirrus/cs8409.o
    make sound/hda/codecs/cirrus/cs8409-tables.o

Expected result:

    CC [M]  sound/hda/codecs/cirrus/cs8409.o
    CC [M]  sound/hda/codecs/cirrus/cs8409-tables.o

Note:

Building these .o files only confirms that the patched source compiles. To actually use the patch, the patched driver must be built and installed as part of the kernel / module build process.

## Runtime confirmation

After installing the patched kernel or module and rebooting, check dmesg:

    sudo dmesg | grep -iE "imac|cs8409|cs42l83|tas576"

Expected iMac-specific log examples:

    iMac probe hook setup
    iMac amp init start
    iMac TAS576 boot reset setup start
    iMac TAS576 TDM slot setup start
    iMac speaker DAC 0x02/0x03 stream clear skipped
    iMac amp init end

Also check the audio device and current PipeWire sink:

    aplay -l
    pactl get-default-sink
    pactl list short sinks

Then test playback with a low volume first:

    speaker-test -c 2 -t wav

Start with a low volume, especially if you have changed the sound profile / configuration.

## Debug logs

The public-clean version keeps normal dmesg output relatively quiet.

Important phase logs remain as codec_info(), while detailed register read/write logs were moved to codec_dbg().

Normal logs show major initialization phases. Debug logs keep detailed I2C / COEF / GPIO / playback hook information.

If low-level tracing is needed, enable dynamic debug for the CS8409 driver.

## Feedback and test reports

Feedback from other users is welcome.

This patch has been tested on one iMac18,3 environment, but the CS8409 / CS42L83 / TAS576 audio path may behave differently depending on kernel version, distribution, firmware state, and hardware revision.

If you test this patch, please share your result.

Please open a GitHub Issue for test reports, questions, reproduction results, or failure reports.

Both successful and unsuccessful test results are useful, especially if you are using the same iMac18,3 model with a different Fedora or kernel version.

Useful information includes:

- Machine model
- Distribution
- Kernel version
- Patch result
- Build result
- Runtime result
- Built-in speaker behavior
- Left/right balance behavior
- High/low or upper/lower speaker balance impression
- Sound profile / configuration switching behavior
- Whether Analog Stereo Output restores playback after switching profiles
- Unexpectedly loud noise after switching profiles, if any
- Pop noise at shutdown, if any
- Headphone switching status, if tested
- Microphone input status, if tested

Useful logs:

    sudo dmesg | grep -iE "imac|cs8409|cs42l83|tas576"
    aplay -l
    pactl get-default-sink
    pactl get-sink-volume @DEFAULT_SINK@

Reports of both success and failure are useful. Failure reports are especially helpful if they include the exact model, kernel version, and relevant dmesg output.

## License and provenance

This repository contains a patch intended to be applied to the Linux kernel source tree.

The patch modifies CS8409-related Linux kernel driver files and is provided under the GPL-2.0-only license, in line with the licensing of the Linux kernel source tree.

This repository does not attempt to relicense the original Linux kernel code. The patch should be understood as a set of changes against the upstream Linux kernel source.

Please refer to the Linux kernel source tree and its licensing documentation for the license terms of the original files.

## Acknowledgements

This work was made possible by studying existing public information, Linux kernel source code, community reports, and real hardware behavior on an iMac18,3 system.

Related public work and references include:

- The Linux kernel project and its HDA / Cirrus codec driver sources
- Fedora and Ubuntu Linux environments used for comparison and testing
- Public CS8409 / Apple HDA codec investigation work, including community repositories and issue discussions
- egorenar/snd-hda-codec-cs8409
- davidjo/snd_hda_macbookpro
- Public reports from GitHub Issues, forums, Reddit, and other community notes related to Apple CS8409 audio behavior

Thanks also to the public technical information and community reports that helped guide this investigation.

## Development process

This patch was developed through an iterative, human-led debugging process on a real iMac18,3 machine.

ChatGPT was used as an assistant for log interpretation, hypothesis organization, code review, cleanup planning, and documentation drafting. All code changes were applied and tested manually on the actual hardware.

The final patch was verified by applying it to a clean linux-6.19.12 source tree and building the modified CS8409-related objects.

## Development notes

This patch is based on the finding that Fedora's CS8409 HDA path can output PCM correctly, but the iMac built-in speakers remain silent unless the Apple-specific external amplifier path is initialized.

The key route is:

    PCM -> CS8409 DAC -> Speaker Pin -> CS42L83 / TAS576 amplifier -> Built-in speakers

Without amplifier initialization:

- PCM path works
- Speaker pins can be controlled
- External output may work
- Built-in speakers remain silent

With this patch:

- The iMac18,3-specific fixup initializes the external amplifier path
- Built-in speaker playback becomes available
- Speaker DAC cleanup behavior is adjusted to keep playback stable

## Status

Current status:

- Built-in speaker playback: working
- Normal desktop playback: working
- Patch application test: passed
- Compile test: passed
- Headphones: not handled
- Microphone: not handled
- Speaker tuning: not Apple-equivalent / not fully verified
