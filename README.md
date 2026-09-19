# VyOS for Radxa E52C

> Unofficial community build of **VyOS Rolling** for the **Radxa E52C**.

[![GitHub Release](https://img.shields.io/github/v/release/VyARM-Community/radxa-e52c?style=for-the-badge)](https://github.com/VyARM-Community/radxa-e52c/releases)
[![GitHub Downloads](https://img.shields.io/github/downloads/VyARM-Community/radxa-e52c/total?style=for-the-badge)](https://github.com/VyARM-Community/radxa-e52c/releases)
[![Build Status](https://img.shields.io/github/actions/workflow/status/frogro/vyos-arm64-board-builder/build-board-candidate.yml?branch=main-test&style=for-the-badge)](https://github.com/frogro/vyos-arm64-board-builder/actions/workflows/build-board-candidate.yml)
[![Donate with PayPal](https://img.shields.io/badge/Donate-PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/FGrootens)


## Table of Contents

- [Quick Start](#quick-start)
- [First Boot and Login](#first-boot-and-login)
- [Optional Helper Scripts](#optional-helper-scripts)
- [Supported Hardware](#supported-hardware)
- [Updates](#updates)
- [Releases](#releases)
- [Changelog](CHANGELOG.md)
- [Contributing](CONTRIBUTING.md)
- [Security](SECURITY.md)
- [License and Trademarks](#license-and-trademarks)
- [Support the Project](#️-support-the-project)

This repository provides an unofficial VyOS image for the Radxa E52C by combining:

- the VyOS ARM64 userspace and configuration system;
- a board-adapted VyOS kernel, modules and firmware, with the E52C vendor U-Boot boot chain;
- first-boot networking helpers and additional network, Wi-Fi and cellular modem drivers and firmware.

**Upstream attribution:** VyOS provides the routing userspace and configuration framework; Armbian provides board integration references and boot-chain support; Radxa designs and documents the E52C hardware.

> [!WARNING]
> This is an independent community project built from components provided by **VyOS**, **Armbian**, and **Radxa** ecosystems. It is not produced, supported, sponsored, certified, or endorsed by the VyOS project, Sentrium S.L., Armbian, Armbian d.o.o., Radxa, or Radxa Computer Co., Ltd. Rolling releases may contain regressions and should be tested before production use.

## Quick Start

### 1. Download the ready-to-use image

Open the [Releases page](https://github.com/VyARM-Community/radxa-e52c/releases) and download:

```text
vyos-VERSION-radxa-e52c-network.img.xz
vyos-VERSION-radxa-e52c-network.img.xz.sha256
```

Verify the download on Linux:

```bash
sha256sum -c vyos-VERSION-radxa-e52c-network.img.xz.sha256
```

Use a target drive larger than the uncompressed image and a boot medium supported by the E52C. Replace `VERSION` in all examples with the version from the selected release.

### 2. Flash with balenaEtcher

[balenaEtcher](https://etcher.balena.io/) is available for Linux, Windows, and macOS.

1. Start balenaEtcher.
2. Select `vyos-VERSION-radxa-e52c-network.img.xz` directly.
3. Select the target boot medium.
4. Click **Flash**.
5. Wait for flashing and verification to finish.

> [!CAUTION]
> Flashing destroys all data on the selected target drive. Verify the destination carefully.

### 3. Flash from Linux with `dd`

Replace `/dev/sdX` with the complete target device, not a partition such as `/dev/sdX1`.

#### Option A: write the compressed image directly

```bash
sudo umount /dev/sdX?* 2>/dev/null || true
xz -dc vyos-VERSION-radxa-e52c-network.img.xz | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
sync
sudo eject /dev/sdX
```

#### Option B: extract first, then write

This may be faster on slower systems because decompression and writing do not occur simultaneously.

```bash
xz -dk vyos-VERSION-radxa-e52c-network.img.xz
sudo umount /dev/sdX?* 2>/dev/null || true
sudo dd if=vyos-VERSION-radxa-e52c-network.img of=/dev/sdX bs=4M status=progress conv=fsync
sync
sudo eject /dev/sdX
```

---

## First Boot and Login

1. Connect the E52C Ethernet port to a network that provides DHCP.
2. Insert or attach the flashed boot drive.
3. Power on the E52C.
4. Allow approximately 60–90 seconds for first-boot configuration.
5. Find the assigned address in your router or DHCP server.
6. Connect over SSH.

Default credentials for this image:

```text
Username: vyos
Password: vyos
```

Example:

```bash
ssh vyos@192.168.1.100
```

Replace the example address with the address assigned to your E52C.

> [!IMPORTANT]
> Change the default password immediately after the first login. For stronger security, configure SSH key authentication and stop using password-based login.

Change the password:

```text
configure
set system login user vyos authentication plaintext-password 'YOUR_NEW_PASSWORD'
commit
save
exit
```

Check Ethernet locally from the serial console (`ttyS2`, 1500000 baud):

```bash
ip -4 -br addr show eth0
```

<details>
<summary><strong>First-boot diagnostics</strong></summary>

```bash
cat /config/dhcp-wan-firstboot-wrapper.log
cat /config/dhcp-wan-ssh-setup.log
systemctl status dhclient@eth0.service --no-pager -l
```

The first-boot marker is:

```text
/config/.dhcp-wan-ssh-firstboot-done
```

</details>

---

## Optional Helper Scripts

The image includes helper scripts under `/usr/local/share/vyos-arm64-firstboot/`, with convenience links in `/home/vyos`. Run the setup helpers as the `vyos` user.

### Configure locale, time and regional settings

Run this helper as the `vyos` user, without `sudo`:

```bash
/home/vyos/set-locales.sh
```

It guides you through the time zone, console keyboard layout, wireless regulatory country, DNS servers and NTP servers. It also persists the system locale as `C.UTF-8`, restarts Chrony and checks time synchronization. Review the proposed settings before applying them. Committing wireless settings may briefly restart an active access point.

### Configure a wireless access point

```bash
/home/vyos/ap-dhcp-wan-setup.sh
```

This is separate from the automatic wired DHCP and SSH setup. Run it only when an access point, DHCP server, DNS forwarding, and NAT are required.

### Configure a modem

```bash
/home/vyos/modem-connect.sh
```

Modem support depends on the modem, transport, drivers, firmware, carrier, and APN.

---

## Supported Hardware

Peripheral results below are project-level references, not confirmation that every listed device has been tested on this exact E52C image. Consult the release notes for board-specific validation.

### Wi-Fi adapters

`ap-dhcp-wan-setup.sh` does not hard-code a specific chipset. It enumerates every `phy` under `/sys/class/ieee80211`, reads supported interface modes from the kernel with `iw phy <phy> info`, and only offers devices that report AP mode support.

#### Hardware reported working with the project

- MediaTek MT7921-class M.2/PCIe Wi-Fi 6 adapters
- Realtek RTL8852-class M.2/PCIe Wi-Fi 6 adapters
- MediaTek MT7612U-based USB adapters

#### Expected to work

- Other Linux `mac80211` adapters that advertise AP mode

Adapters whose drivers only support client or station mode cannot be used by the AP helper.

Useful diagnostics:

```bash
ip link show
iw dev
dmesg
```

### Cellular modems

`modem-connect.sh` supports PCIe- and USB-attached modems, preferring native VyOS WWAN configuration where supported and using helpers for device-specific initialization and the AT/RNDIS fallback path.

#### Tested with the project

- Fibocom FM350-GL (Revision: 81600.0000.00.29.24.02,  SVN: 10), including automatic FCC unlock over the AT port

#### Expected to work with compatible drivers and firmware

- Quectel RM505Q
- Intel XMM7560-based modems
- Other QMI- or MBIM-capable modems supported by ModemManager

Actual connectivity also depends on the SIM carrier, APN, regional firmware, and supported bands.

---

## Updates

### Update through the board channel

**On a fresh installation of the channel-enabled image, the E52C update channel is configured automatically during first-boot setup.** You do not need to enter an update URL. No automatic image installation is enabled.

To install the latest image, run this command in operational mode:

```text
add system image latest
```

Follow the installer prompts, retain your configuration and previous image, then reboot when ready as described below.

<details>
<summary><strong>Existing installations: configure the channel once if it is missing</strong></summary>

If you retain an older configuration when installing a new image, the channel URL is not automatically added. An existing administrator-defined URL is also preserved. On a compatible installation, set the E52C channel once:

```text
configure
set system update-check url 'https://github.com/VyARM-Community/radxa-e52c/releases/latest/download/image-version.json'
commit
save
exit
```

After that, use `add system image latest` for subsequent updates.

</details>

The channel becomes usable when the first release containing `image-version.json` has been published. This feed points to the matching E52C ISO with additional network support.

### Update using a specific ISO

Alternatively, copy the full HTTPS download link of the desired `.iso` from the [Releases page](https://github.com/VyARM-Community/radxa-e52c/releases):

```text
add system image https://github.com/VyARM-Community/radxa-e52c/releases/download/RELEASE_TAG/vyos-VERSION-radxa-e52c-network.iso
```

Replace `RELEASE_TAG` and `VERSION` with the actual release values. Use the `.iso` for system-image updates; `.img.xz` is intended for initial installation or recovery.

### Configuration, reboot and rollback

1. Save your current configuration with `configure`, `save`, then `exit`, and keep a separate backup of `/config/config.boot`.
2. Run one of the installation commands above and follow its prompts. Choose to retain the current configuration when requested.
3. Keep the previous working image as a fallback. Use `show system image` to inspect the installed images and boot selection.
4. Reboot when ready with `reboot`.
5. Check networking and the configured services after boot.

To select a retained image for the next boot, use its exact name from `show system image`:

```text
set system image default-boot IMAGE_NAME
reboot
```

These are operational-mode commands. The E52C uses vendor U-Boot/extlinux with board-specific hooks for the native VyOS image lifecycle. Image renaming is not supported on this boot path.

> [!IMPORTANT]
> Older E52C installations without these extlinux lifecycle hooks cannot safely migrate through their old ISO installer. Use a fresh `.img.xz` installation for that migration and retain the old boot medium for recovery. Only install an ISO matching the E52C and the installed image variant. Consult the release notes for the validation status of the exact image.

Preserved configurations remain authoritative after an update: an older configuration does not automatically receive the new channel URL. Use the configuration commands above when needed.

---

## Releases

Prebuilt images are published on the [GitHub Releases page](https://github.com/VyARM-Community/radxa-e52c/releases).

Each release should normally contain:

```text
vyos-VERSION-radxa-e52c-network.img.xz
vyos-VERSION-radxa-e52c-network.img.xz.sha256
```

Use the compressed `.img.xz` for initial installation. A matching `.iso` and `.iso.sha256` are provided for supported system-image updates.

See [CHANGELOG.md](CHANGELOG.md) for the changes in each release.

The published VyOS Rolling is a reference; package versions may differ because the image is built later against the rolling package repository.

---

## License and Trademarks

The repository contains or builds software from multiple upstream projects. Their respective licenses remain in effect. Review the license and copyright files included in the repository and generated image.

“VyOS” and associated marks are trademarks of their respective owner. “Armbian” and associated marks are trademarks of Armbian d.o.o. “Radxa”, “E52C”, and associated marks are trademarks of Radxa Computer Co., Ltd.

These names are used solely to identify compatibility, upstream software, boot-chain and kernel components, and supported hardware. No affiliation, sponsorship, certification, or endorsement is claimed.

This repository does not redistribute third-party logo artwork.

---

## ❤️ Support the Project

If this project saved you time or made it easier to run VyOS on the Radxa E52C, please consider supporting its development.

Contributions help cover hardware, testing, maintenance, and development time.

[![Donate with PayPal](https://img.shields.io/badge/Donate-PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/FGrootens)

Thank you for your support. ☕
