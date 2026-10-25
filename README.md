# Kinesis Advantage 360 Pro ZMK Config

## Modifying the keymap

[The ZMK documentation](https://zmk.dev/docs) covers both basic and advanced functionality and has a table of OS compatibility for keycodes. This repository builds against upstream ZMK, so all of the documentation applies, including the RGB Underglow and Backlight sections. For more information see [this note](#note).

The keymap lives in [config/adv360pro.keymap](config/adv360pro.keymap) and is shared by both halves; Kconfig options go in [config/adv360pro.conf](config/adv360pro.conf).

* If you would like to continue using GitHub we recommend using Nick Coutsos’s keymap editor: https://nickcoutsos.github.io/keymap-editor/.
* If you would prefer to leave GitHub and firmware flashing behind you can perform a one-time firmware update to gain access to Clique. Get started here: https://kinesis-ergo.com/360p-clique-upgrade/. Clique is designed for the Kinesis firmware and is not supported with the upstream ZMK firmware built by this repository.
* The left half firmware built by this repository has [ZMK Studio](https://zmk.dev/docs/features/studio) enabled, so the keymap can also be edited live from https://zmk.studio over USB. Unlock the keyboard first with Mod+Esc (`&studio_unlock` on the Mod layer).

Certain ZMK features (e.g. combos) require knowing the exact key positions in the matrix. They can be found in both image and text format [here](assets/key-positions.md)

### Home row mods

The base layer has "timeless" home row mods on `A S D F` and `J K L ;`, configured as described in [urob's zmk-config](https://github.com/urob/zmk-config#timeless-homerow-mods):

| Key | Hold | Key | Hold |
| :---: | --- | :---: | --- |
| A | GUI (Cmd / Win) | J | Ctrl |
| S | Alt (Option) | K | Shift |
| D | Shift | L | Alt (Option) |
| F | Ctrl | ; | GUI (Cmd / Win) |

Tapping a key types the letter as usual. Holding it while pressing a key on the *other* hand (or a thumb key) produces the modifier; keys on the same hand always resolve as taps unless the modifier is held past the tapping term. The `hml` and `hmr` behaviours are defined at the top of [config/adv360pro.keymap](config/adv360pro.keymap), where `tapping-term-ms`, `quick-tap-ms` and `require-prior-idle-ms` can be tuned; the troubleshooting section of urob's README explains which value to change for which symptom. Global defaults for ZMK's built-in `&mt` and `&lt` behaviours are set at the top of the keymap as well.

### Thumb keys and navigation

The left thumb Space key is a layer-tap (`&lt 1 SPACE`): tap it for Space, hold it for layer 1 (the keypad layer). On that layer `H J K L` are the arrow keys, vim style (left, down, up, right), and the right thumb Backspace sends Alt+Backspace to delete a word. On the base layer the right thumb key is a plain Backspace.

### Combos

Two combos on the right home row, active on the base layer only:

| Keys | Sends | macOS default |
| :---: | --- | --- |
| J + K | Cmd+Shift+[ | Previous tab |
| K + L | Cmd+Shift+] | Next tab |

Both keys have to go down within 18 ms of each other, and no other key may have been pressed in the previous 150 ms, so rolling `jk` or `kl` while typing does not trigger them. They are defined in the `combos` node of [config/adv360pro.keymap](config/adv360pro.keymap).

### Inner column keys

The keys next to the thumb clusters on the base layer:

| Key | Sends |
| --- | --- |
| Left, row 1 (next to `5`) | Caps Word (`&caps_word`): the next word is typed in capitals |
| Left, row 2 (next to `T`) | Cmd+Space |
| Left, row 3 (next to `G`) | Cmd+Shift+Alt+S (Shortcat) |
| Right, row 3 (next to `H`) | Right GUI (Cmd / Win) |

The keypad layer toggle that used to sit on the row 1 key is still available on the Fn layer (Fn + that key), and the same key toggles the keypad layer off again while it is active.

## Building the Firmware with GitHub Actions

### Setup

1. Fork this repo.
2. Enable GitHub Actions on your fork.

### Build firmware

1. Push a commit to trigger the build.
2. Download the `firmware` artifact. It contains the `*-left.uf2` and `*-right.uf2` files; the left half is built with ZMK Studio enabled.

A second `settings-reset` artifact contains firmware that wipes the stored settings (Bluetooth bonds etc.) of each half, see [Resetting settings](#resetting-settings).

## Building the Firmware in a local container

### Setup

#### Software

* Either Podman or Docker is required, Podman is chosen if both are installed.
* Make is also required

#### Windows specific

* If compiling on Windows use WSL2 and Docker [Docker Setup Guide](https://docs.docker.com/desktop/windows/wsl/).
* Install make using `sudo apt-get install make` inside the WSL2 instance.
* The repository can be cloned directly into the WSL2 instance or accessed through the C: mount point WSL provides by default (`/mnt/c/path-to-repo`).

#### macOS specific

On macOS [brew](https://brew.sh) can be used to install the required components.

* docker
* [colima](https://github.com/abiosoft/colima) can be used as the docker engine

```shell
brew install docker colima
colima start
```
> Note: On Apple Silicon (ARM based) systems you need to make sure to start colima with the correct architecture for the container being used.
> ```
> colima start --arch x86_64
> ```

#### Ubuntu/Debian specific

```shell
sudo apt-get install docker make
```

### Building the firmware

1. Execute `make` to build firmware for both halves or `make left` to only build firmware for the left hand side.
2. Check the `firmware` directory for the latest firmware build. The first part of the filename is the timestamp when the firmware was built.

### Cleanup

The built docker container and compiled firmware files can be deleted with `make clean`. This might be necessary if you updated your fork from V2.0 to V3.0 and are encountering build failures.

Creating the docker container takes some time. Therefore `make clean_firmware` can be used to only clean firmware without removing the docker container. Similarly `make clean_image` can be used to remove the docker container without removing compiled firmware files.

## Flashing firmware

Follow the programming instruction on page 8 of the [Quick Start Guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-QSG-v8-25-22.pdf) to flash the firmware.

### Overview

1. Extract the firmwares from the archive downloaded from the GitHub build job (If using the cloud builder) or the firmware folder (If building locally).
1. Connect the left side keyboard to USB.
1. Press Mod+macro1 to put the left side into bootloader mode; it should attach to your computer as a USB drive.
1. Copy `left.uf2` to the USB drive and it will disconnect.
1. Power off both keyboards (by unplugging them and making sure the switches are off).
1. Turn on the left side keyboard with the switch.
1. Connect the right side keyboard to USB to power it on.
1. Press Mod+macro3 to put the right side into bootloader mode to attach it as a USB drive.
1. Copy `right.uf2` to the mounted drive.
1. Unplug the right side keyboard and turn it back on.
1. Enjoy!

> Note: There are also physical reset buttons on both keyboards which can be used to enter and exit the bootloader mode. Their location is described in section 2.7 on page 9 in the [User Manual](https://kinesis-ergo.com/wp-content/uploads/Advantage360-ZMK-KB360-PRO-Users-Manual-v3-10-23.pdf) and use is described in section 5.9 on page 14. 

> Note: Some operating systems wont always treat the drive as ejected after the settings-reset file is flashed or may throw a spurious error, this doesn't mean that the flashing process has failed.

### Resetting settings

When moving from the Kinesis ZMK fork to the upstream ZMK firmware built by this repository (or between major ZMK/Zephyr versions) the two halves may fail to pair with each other or with previously paired computers. If that happens, flash the `settings-reset` firmware from the build artifacts to both halves (`*-settings-reset-left.uf2` on the left, `*-settings-reset-right.uf2` on the right), then flash the regular firmware again and re-pair your devices. See [ZMK's connection troubleshooting guide](https://zmk.dev/docs/troubleshooting/connection-issues) for details. The [settings-reset.uf2](/settings-reset.uf2) file in this repository was built from the Kinesis fork; prefer the freshly built one.

### Upgrading from V2 to V3

If you encounter a git conflict when updating your repository to V3.0 please follow the instructions on how to resolve it [here](UPGRADE.md).

Updating from V2.0 based firmwares to V3.0 based firmwares can be a rather complex process. There are reset files for every major firmware revision as well as documentation on the update process available [here](https://kinesis-ergo.com/support/kb360pro/#firmware-updates).

## Versioning

Starting on 11/15/2023 the Advantage 360 Pro will now automatically record the compilation date, branch and Git commit hash in a macro that can be accessed with Mod+V. This will type out the following string: YYYYMMDD-XXXX-YYYYYY, where XXXX is the first 4 characters of the Git branch and YYYYYY is the Git commit hash. In addition to this the builds compiled by GitHub actions are now timestamped and also record the commit hash in the filename. 

## N-Key Rollover

By default this keyboard has NKRO enabled, however for compatibility reasons the higher ranges are not enabled. If you want to use F13-F24 or the INTL1-9 keys with NKRO enabled add `CONFIG_ZMK_HID_KEYBOARD_NKRO_EXTENDED_REPORT=y` to [config/adv360pro.conf](/config/adv360pro.conf).

## Battery reporting

By default reporting the battery level over BLE is disabled as this can cause some computers to spontaneously wake up repeatedly. If you'd like to enable this functionality change `CONFIG_BT_BAS=n` to `CONFIG_BT_BAS=y` in [config/adv360pro.conf](/config/adv360pro.conf).

## Indicator LEDs

With upstream ZMK the three LEDs on each half are driven by ZMK's standard [RGB underglow](https://zmk.dev/docs/features/underglow) feature. They are off by default and can be toggled with `&rgb_ug RGB_TOG` on the Mod layer; the effect, colour and brightness are set with the `CONFIG_ZMK_RGB_UNDERGLOW_*` options in [config/adv360pro.conf](/config/adv360pro.conf). The layer colours and the CAPS/NUM/SCROLL LOCK modifier indicators of the Kinesis ZMK fork are not available, see [this note](#note).

## Changelog

The changelog for both the config repo and the underlying ZMK fork that the config repo builds against can be found [here](CHANGELOG.md).

## Updating ZMK

The ZMK version that gets built is pinned in [config/west.yml](config/west.yml) through the `revision:` of the `zmk` project. Change it to a newer commit, a release tag such as `v0.4.0`, or `main` to follow ZMK's development branch, then push to rebuild. See the ZMK blog post on [pinning ZMK](https://zmk.dev/blog/2025/06/20/pinned-zmk) for details, and [ZMK's beta testing guide](https://zmk.dev/docs/features/beta-testing#testing-features) for pointing the manifest at a feature branch.

Feedback on upstream ZMK should be submitted as a GitHub issue on the [ZMK repository](https://github.com/zmkfirmware/zmk) as opposed to this config repository.

## Note

This config repository builds against [upstream ZMK](https://github.com/zmkfirmware/zmk) using its built-in `adv360pro_left//zmk` and `adv360pro_right//zmk` board definitions, pinned in [config/west.yml](config/west.yml). It originally referenced [Kinesis' customised version of ZMK](https://github.com/ReFil/zmk/tree/adv360-z3.5-2), which is based on Zephyr 3.5 and adds Advantage 360 Pro specific functionality (layer and modifier indicator LEDs, battery status display, Clique support) over base ZMK. Those features are not available with upstream ZMK, and Kinesis cannot provide customer service for usage of base ZMK. If you need them, point `west.yml` back at the Kinesis fork and restore the `config/boards` directory from this repository's git history.

## Other support

Further support resources can be found on Kinesis.com:

* https://kinesis-ergo.com/support/kb360pro/#firmware-updates
* https://kinesis-ergo.com/support/kb360pro/#manuals

In the event of a hardware issue it may be necessary to open a support ticket directly with Kinesis as opposed to a GitHub issue in this repository.
* https://kinesis-ergo.com/support/kb360pro/#ticket

