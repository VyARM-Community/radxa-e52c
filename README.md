# VyOS for Radxa E52C

Unofficial community ARM64 images for the **Radxa E52C**, built by the [VyOS ARM64 board builder](https://github.com/frogro/vyos-arm64-board-builder).

These images include additional network, Wi-Fi and cellular modem drivers and firmware. Tailscale and KVM profiles are not part of this update channel. Release files are copied unchanged from the central builder; there is no separate board build here.

## Downloads

Open [Releases](https://github.com/VyARM-Community/radxa-e52c/releases) and choose:

- `.img.xz` for initial installation or recovery;
- `.iso` for a supported in-place system-image update;
- the adjacent `.sha256` file to verify your download.

The first channel-enabled release is being prepared. Until it is published, there is no usable latest-update feed.

## Initial installation

1. Download the E52C **network** image and its checksum.
2. Verify it in the download directory: `sha256sum -c <image>.img.xz.sha256`.
3. Flash the compressed image to the target boot medium with balenaEtcher, or decompress it and write it with a disk-imaging tool. This erases the selected medium.
4. Connect Ethernet to a network with DHCP, insert the boot medium, and power on.
5. Find the DHCP address in your router and connect with `ssh vyos@<address>`.

Default login: **vyos / vyos**. Change the password immediately:

```text
configure
set system login user vyos authentication plaintext-password 'YOUR-NEW-PASSWORD'
commit
save
exit
```

First-boot setup configures a detected wired interface for DHCP and enables SSH. Use a trusted setup network. The serial console is `ttyS2`, 1500000 baud.

## Setup helpers

Image-provided helpers live in `/usr/local/share/vyos-arm64-firstboot/`. Convenience links for the recurring helpers appear in the `vyos` home directory. Run them as `vyos`, not with `sudo`:

```text
./set-locales.sh
./ap-dhcp-wan-setup.sh
./modem-connect.sh
```

- `set-locales.sh`: timezone, keyboard, wireless country, DNS and NTP setup.
- `ap-dhcp-wan-setup.sh`: guided AP/network setup; requires supported wireless hardware.
- `modem-connect.sh`: supported modem setup, using native WWAN where possible and helpers where required. A manual setup replaces the previous modem setup.
- `dhcp-wan-ssh-setup.sh`: initial wired DHCP/SSH setup; normally invoked automatically once. If needed, call `/usr/local/share/vyos-arm64-firstboot/dhcp-wan-ssh-setup.sh --help`.

## Updates

Only use an **E52C network** ISO. Earlier E52C installations without the native extlinux lifecycle hooks must first be migrated using a fresh `.img.xz`; do not assume their old ISO installer is compatible. Keep the old boot medium and a configuration backup for recovery.

The board channel uses the native VyOS update-check mechanism. Once its first release is available, configure it on an existing compatible installation with:

```text
configure
set system update-check url 'https://github.com/VyARM-Community/radxa-e52c/releases/latest/download/image-version.json'
commit
save
exit
add system image latest
```

New channel-enabled installations receive this URL during their first-boot setup. No automatic installation or scheduled update check is enabled. Preserved configurations remain authoritative after image updates; set the URL explicitly when migrating an existing configuration.

Alternatively use the full HTTPS URL of the release's ISO:

```text
add system image https://github.com/VyARM-Community/radxa-e52c/releases/download/<tag>/<image>.iso
```

Follow the installer prompts and retain the previous image. Reboot when ready. The board uses vendor U-Boot/extlinux with VyOS image lifecycle hooks. To select a retained image, use `set system image default-boot <image-name>` in operational mode. Image renaming is not supported on this boot provider.

## Validation and limitations

Check each release's hardware-test status and known limitations. Successful CI checks do not replace testing that exact image on hardware. Driver inclusion does not guarantee support for every modem or peripheral.

The published official VyOS Rolling is a reference. Packages may differ because these ARM64 images are built later against the rolling package repository. Board kernel, firmware and selected profile additions are intentional differences.

## Sources and attribution

Build sources, patches, provenance and reports are maintained in the [central builder](https://github.com/frogro/vyos-arm64-board-builder). Each release identifies its source build. Components retain their respective licenses; consult the source references and installed `/usr/share/doc/*/copyright` files.

This is an independent community project, not an official or endorsed release of VyOS, Armbian or Radxa.
