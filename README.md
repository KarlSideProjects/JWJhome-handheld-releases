# Handheld Hardware Monitor


Version **0.0.30** provides compiled Display packages for the supported R36 and TrimUI installations. Python application roles, including the installer, are compiled with Nuitka; first-party JavaScript is minified and mangled. This public repository contains release packages, installation instructions and version/license information.

Karl’s software is free for noncommercial use. Commercial use, including internal company use, requires Karl’s prior written permission. See [LICENSE.md](LICENSE.md) and [third-party notices](THIRD-PARTY.md). Previously granted GPL rights remain effective.

Before installing, confirm the release is marked stable on its page. Pre-release packages are for validation.

[Download both packages](https://github.com/KarlSideProjects/JWJhome-handheld-releases/releases/tag/v0.0.30). [Version metadata](version.json) identifies the compiled source, archive hashes and installer manifest hashes. Installation uses the complete compiled package from this public release.

## Compatibility and preparation

* **R36:** the tested Debian 13 ARM64 installation with glibc 2.41; run as `ark`. Exit the monitor with **Select+B** before installing. This artifact does not support older glibc firmware.
* **TrimUI Smart Pro S:** the tested ARM64 Buildroot system with glibc 2.33; run as `root`. The existing compatible Chromium/SDL runtime and CJK font under `/mnt/UDISK/hwmonitor/runtime` are prerequisites and remain installed separately.

Use a terminal with curl, gzip, tar and sha256sum. If curl is unavailable on the handheld, download and verify the matching archive on your computer, transfer it to the device, then extract and run the same bundled installer. Existing MQTT and language settings are retained. Installation restarts the display; choose a time when that interruption is acceptable.

## Install from a computer

The release contains exactly seven assets: both device archives, their checksums, `version.json`, and these two computer entrypoints. The version metadata binds the archive, manifest and entrypoint SHA-256 values to source `12b30d05f6f38ebb149e14ce52bbdcd03b488f02`. Confirm the stable release and the hashes through a trusted source before running either script. Download the script to a file; do not pipe a download into a shell.

Bash requires Python 3, `ssh` and `scp`. Replace `handheld.example` with your device. Use `--device trimui` for TrimUI.

```sh
curl --fail --location --proto '=https' --proto-redir '=https' https://github.com/KarlSideProjects/JWJhome-handheld-releases/releases/download/v0.0.30/install-handheld.sh --output install-handheld.sh &&
printf '%s  install-handheld.sh\n' '0c89fc8bf63776281be86ff469a798421e0bf2cab1b25937268fd9351da13d81' | sha256sum --check - &&
bash ./install-handheld.sh --device r36 --host handheld.example --metadata https://github.com/KarlSideProjects/JWJhome-handheld-releases/releases/download/v0.0.30/version.json --source 12b30d05f6f38ebb149e14ce52bbdcd03b488f02 --version 0.0.30 --state "$HOME/handheld-install-0.0.30"
```

Windows PowerShell 5.1 or 7 requires native OpenSSH `ssh` and `scp`. Download [install-handheld.ps1](https://github.com/KarlSideProjects/JWJhome-handheld-releases/releases/download/v0.0.30/install-handheld.ps1) to a new directory, then verify it before execution:

```powershell
if ((Get-FileHash -LiteralPath .\install-handheld.ps1 -Algorithm SHA256 -ErrorAction Stop).Hash.ToLowerInvariant() -ceq '6c7aef23f710b847681f82ca2a6d0ae8c1e9c2d7a90a6f63df2dd3a25deba46f') {
    powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\install-handheld.ps1 -Device r36 -HostName handheld.example -Metadata https://github.com/KarlSideProjects/JWJhome-handheld-releases/releases/download/v0.0.30/version.json -Source 12b30d05f6f38ebb149e14ce52bbdcd03b488f02 -Version 0.0.30 -State "$env:USERPROFILE\handheld-install-0.0.30"
} else { throw 'Installer checksum mismatch' }
```

The Windows command runs only the hash-verified file in a child PowerShell process. Its execution-policy setting lasts only for that process; it does not change the computer or user policy.

Use a new private state directory for each attempt. Verify the device host key and allow OpenSSH to prompt for authentication. Settings remain private; use `--config` or `-Config` for an existing private MQTT configuration. An unknown result requires investigation before another activation attempt. These instructions do not establish compatibility with untested firmware or replace the device acceptance checks.

## R36 installation

```sh
curl --fail --location https://github.com/KarlSideProjects/JWJhome-handheld-releases/releases/download/v0.0.30/display-r36-arm64-0.0.30.tar.gz --output display-r36-arm64-0.0.30.tar.gz &&
printf '%s  display-r36-arm64-0.0.30.tar.gz\n' '710cbb65f46764d1c5198812f5560f1ff9b345ca65ee5fc6d4a6abe0bb9b6f2e' | sha256sum --check - &&
bundle=$(mktemp -d) &&
gzip -dc display-r36-arm64-0.0.30.tar.gz | tar -xf - -C "$bundle" &&
cd "$bundle" &&
sh ./install-r36.sh --manifest-sha256 f26433074185da5424957dfc5a4b358098e1912a46fc7b563294da60467dc4b0
```

## TrimUI installation

```sh
curl --fail --location https://github.com/KarlSideProjects/JWJhome-handheld-releases/releases/download/v0.0.30/display-trimui-arm64-0.0.30.tar.gz --output display-trimui-arm64-0.0.30.tar.gz &&
printf '%s  display-trimui-arm64-0.0.30.tar.gz\n' '75f97da246b88a679f8355ff1f8af146675f3fed5e51c172e0ae881b48697c9c' | sha256sum --check - &&
bundle=$(mktemp -d) &&
gzip -dc display-trimui-arm64-0.0.30.tar.gz | tar -xf - -C "$bundle" &&
cd "$bundle" &&
sh ./install-trimui.sh --manifest-sha256 4dae1467709fcf8920823c412e2b7486bbe7ceb7bb8124a3929d12ac8f322523
```

For a new MQTT configuration, after verifying and extracting the archive, run `./app/mqtt_setup.bin --output /absolute/private/path/new-config.json` in an interactive terminal. The output file must not exist. Add `--config /absolute/private/path/new-config.json` to the installer command. Keep this file private and outside this repository.

The installer verifies its supplied manifest and every registered file before running the compiled installer. It preserves a private backup and reports incomplete recovery rather than silently claiming success. If it reports an unknown display owner or an unsupported running-process stop, exit the application normally before retrying. Dependency and firmware changes are outside application rollback.
