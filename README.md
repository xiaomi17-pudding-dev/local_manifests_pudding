# Xiaomi 17 (pudding) — local manifests

Start here to build **Android 17 (LineageOS 24.0)** for the
**Xiaomi 17**, codename **`pudding`** (Qualcomm SM8850 / Snapdragon 8 Elite Gen 5).

> **Status:** LineageOS 24.0 runs with SELinux enforcing; see the LineageOS section for what has
> been tested on the device. Manifests and build instructions are added here as each stage
> starts working.

## Repositories

On GitHub in [`xiaomi17-pudding-dev`](https://github.com/xiaomi17-pudding-dev), unless noted:

| Repository | Contents |
|---|---|
| `local_manifests_pudding` | This repo: `repo` manifests and build instructions |
| `twrp_device_xiaomi_pudding` | TWRP device tree for the Xiaomi 17, by antocorvo3000 (original history kept) |
| `device_xiaomi_sm8850-common` | Common device tree for the Xiaomi 17 series (SM8850) |
| `device_xiaomi_pudding` | Device tree for the Xiaomi 17 |
| `device_xiaomi_miuicamera-pudding` | MIUI camera app of the stock firmware, inherited by the device tree |
| [`vendor_xiaomi_sm8850-common`](https://gitlab.com/xiaomi17-pudding-dev/vendor_xiaomi_sm8850-common) (GitLab) | Proprietary files from the stock vendor partition |
| [`vendor_xiaomi_pudding`](https://gitlab.com/xiaomi17-pudding-dev/vendor_xiaomi_pudding) (GitLab) | Proprietary files from the stock odm partition (the GitHub repo of the same name only points here) |
| [`vendor_xiaomi_miuicamera-pudding`](https://gitlab.com/xiaomi17-pudding-dev/vendor_xiaomi_miuicamera-pudding) (GitLab) | Proprietary files of the MIUI camera app |
| `device_xiaomi_pudding-kernel` | Prebuilt stock kernel, DTBs and module lists |
| `android_hardware_xiaomi` | Fork of LineageOS `hardware/xiaomi`; branch `pudding/fingerprint-modules` adds the ultrasonic fingerprint modules |
| `android_kernel_xiaomi_sm8850` | Xiaomi kernel source (branch `popsicle-w-oss`, unmodified) |
| `android_kernel_xiaomi_sm8850-devicetrees` | Xiaomi kernel devicetrees (branch `popsicle-w-oss`, unmodified) |

Each repo carries its own README with the stock firmware version it is based on. The
proprietary files are hosted on GitLab because of their size (Git LFS).

## Stages

| Stage | Status | Manifest |
|---|---|---|
| TWRP 3.7.1_16 (`twrp_sm8750_thales`, recovery) | built, in testing | [`twrp/`](twrp) |
| LineageOS 24.0 (`lineage_pudding`, branch `lineage-24.0`) | runs, in testing | [`lineage.xml`](lineage.xml) |
| Other AOSP-based ROMs | not started | _coming_ |

## Building (once a stage is available)

```bash
# 1. Source: the base manifest named in the stage's section below
repo init -u <base manifest> -b <branch> --git-lfs
# 2. Add this repo's manifest
mkdir -p .repo/local_manifests
curl -o .repo/local_manifests/pudding.xml \
  https://raw.githubusercontent.com/xiaomi17-pudding-dev/local_manifests_pudding/<branch>/<stage>.xml
# 3. Sync and build
repo sync -c --no-tags -j$(nproc)
source build/envsetup.sh
# lunch / build target per stage
```

### TWRP

```bash
repo init -u https://github.com/TWRP-Test/platform_manifest_twrp_aosp.git -b twrp-16.0 --depth=1
mkdir -p .repo/local_manifests
for f in pins restore-minimal pudding; do
  curl -o .repo/local_manifests/$f.xml \
    https://raw.githubusercontent.com/xiaomi17-pudding-dev/local_manifests_pudding/lineage-24.0/twrp/$f.xml
done
repo sync -c --no-tags -j$(nproc)
patch -p1 -d system/vold < device/xiaomi/sm8750_thales/patches/0001-vold-fix-synthetic-password-gcm.patch
export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh
lunch twrp_sm8750_thales-bp2a-eng
m recoveryimage
```

`pins.xml` pins the TWRP-Test projects to their state of 2026-07-12, `restore-minimal.xml`
adds back AOSP projects the recovery build needs, `pudding.xml` adds the device tree.
Flash `out/target/product/sm8750_thales/recovery.img` to `recovery_a` / `recovery_b`.

### LineageOS 24.0

```bash
repo init -u https://github.com/LineageOS/android.git -b lineage-24.0 --git-lfs
mkdir -p .repo/local_manifests
curl -o .repo/local_manifests/pudding.xml \
  https://raw.githubusercontent.com/xiaomi17-pudding-dev/local_manifests_pudding/lineage-24.0/lineage.xml
repo sync -c --no-tags -j$(nproc)
source build/envsetup.sh
lunch lineage_pudding-cp2a-userdebug
m bacon
```

The OTA package is `out/target/product/pudding/lineage-24.0-<date>-UNOFFICIAL-pudding.zip`,
the recovery image `out/target/product/pudding/recovery.img`. The phone must run the stock
firmware the blobs come from (HyperOS OS3.0.335.0.XPCEUXM, see the device tree READMEs).

First installation: flash the recovery image to the active slot's recovery partition
(`fastboot getvar current-slot`, then `fastboot flash recovery_<slot> recovery.img`), boot
into recovery, Format data, then Apply update → ADB sideload the OTA package.

Tested on the device (release build, SELinux enforcing):

| Feature | Status |
|---|---|
| Boot, lock screen PIN | works |
| Fingerprint (under-display, screen on) | works |
| Face unlock | works |
| Camera (MIUI camera: photo and video) | works |
| Widevine DRM | works, L1 |
| Wi-Fi | works |
| Bluetooth | works |
| Audio | works |
| Charging | works |

Not tested yet: calls and mobile data, GPS, NFC and the sensors. Known limitations: no fingerprint unlock with the screen off yet, and only
units with the Qualcomm ultrasonic fingerprint sensor are supported. The build has no Google
apps; banking apps that require Play Integrity device integrity will not work on an unlocked
bootloader.

## Principles

- Stay as close to upstream (LineageOS, AOSP, Xiaomi, Qualcomm) as possible; every carried change
  is a separate, documented commit.
- Blobs and prebuilts come from one stated stock firmware version.
- Upstream code is never squashed; Xiaomi's kernel branches are kept unmodified.

## Warnings

Unlocking the bootloader and flashing custom software can brick your device and void your
warranty. Back up `persist`, `modemst1`, `modemst2` and `fsg` before flashing anything, and
never relock the bootloader on modified software. You do this at your own risk.

## Credits

Thanks to the Xiaomi 17 / SM8850 community for prior work on device trees and kernels;
specific credits are kept in the commits and READMEs where their work is used.
