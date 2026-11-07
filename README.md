# Xiaomi 17 (pudding) — local manifests

Start here to build **Android 17 (LineageOS 24.0, later PixelOS)** for the
**Xiaomi 17**, codename **`pudding`** (Qualcomm SM8850 / Snapdragon 8 Elite Gen 5).

> **Status: early bring-up.** Nothing is bootable yet. Manifests and build instructions are
> added here as each stage starts working.

## Repositories

On GitHub in [`xiaomi17-pudding-dev`](https://github.com/xiaomi17-pudding-dev), unless noted:

| Repository | Contents |
|---|---|
| `local_manifests_pudding` | This repo: `repo` manifests and build instructions |
| `device_xiaomi_sm8850-common` | Common device tree for the Xiaomi 17 series (SM8850) |
| `device_xiaomi_pudding` | Device tree for the Xiaomi 17 |
| [`vendor_xiaomi_sm8850-common`](https://gitlab.com/xiaomi17-pudding-dev/vendor_xiaomi_sm8850-common) (GitLab) | Proprietary files from the stock vendor partition |
| [`vendor_xiaomi_pudding`](https://gitlab.com/xiaomi17-pudding-dev/vendor_xiaomi_pudding) (GitLab) | Proprietary files from the stock odm partition (the GitHub repo of the same name only points here) |
| `device_xiaomi_pudding-kernel` | Prebuilt stock kernel, DTBs and module lists |
| `android_kernel_xiaomi_sm8850` | Xiaomi kernel source (branch `popsicle-w-oss`, unmodified) |
| `android_kernel_xiaomi_sm8850-devicetrees` | Xiaomi kernel devicetrees (branch `popsicle-w-oss`, unmodified) |

Each repo carries its own README with the stock firmware version it is based on. The
proprietary files are hosted on GitLab because of their size (Git LFS).

## Stages

| Stage | Status | Manifest |
|---|---|---|
| LineageOS 24.0 (`lineage_pudding`, branch `lineage-24.0`) | in progress, not yet booted | _coming_ |
| PixelOS | not started | _coming_ |

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
