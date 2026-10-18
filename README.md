# local_manifests

Local `repo` manifest for the **Nothing Phone (3a) Lite (Galaxian, MT6878)** device
tree. It declares every project that lives in the `Nothing-Phone-3a-Lite` org, so a
build host only needs the upstream ROM manifest plus this repo.

## Usage

```sh
# from the ROM source root, once:
git clone https://github.com/Nothing-Phone-3a-Lite/local_manifests .repo/local_manifests

# then
repo sync -j8 --force-sync
```

`repo` scans `.repo/local_manifests/` automatically, so any `*.xml` in here is picked
up alongside the main manifest. The remote is declared as `n3al` rather than `github`
so it cannot collide with a remote already defined upstream.

## Projects

| Path | Repo | Revision | Kind |
|---|---|---|---|
| `device/nothing/Galaxian` | `device_nothing_Galaxian` | `lineage-24.0-repos` | device tree (entry commented, see below) |
| `device/nothing/Galaxian-kernel` | `device_nothing_Galaxian-kernel` | `origin-kernel` | hard |
| `vendor/nothing/Galaxian` | `proprietary_vendor_nothing_Galaxian` | `lineage-24.0` | hard |
| `device/mediatek/sepolicy_vndr` | `device_mediatek_sepolicy_vndr` | `lineage-24.0` | hard |
| `hardware/mediatek` | `hardware_mediatek` | `lineage-24.0` | hard |
| `hardware/dolby` | `hardware_dolby` | `17` | hard |
| `packages/apps/ParanoidGlyph` | `packages_apps_ParanoidGlyph` | `main` | optional |
| `packages/apps/GlyphAdapter` | `packages_apps_GlyphAdapter` | `main` | optional |
| `packages/apps/GalaxianParts` | `packages_apps_GalaxianParts` | `main` | optional |
| `packages/apps/NTGallery` | `packages_apps_NTGallery` | `main` | optional |
| `packages/apps/NTCamera` | `packages_apps_NTCamera` | `main` | optional |
| `patches` | `patches` | `master` | helper |

**Hard** means the build fails outright if the project is missing — the device tree
`include`s or `inherit-product`s it unguarded. **Optional** means the device tree
guards it with `wildcard` / `inherit-product-if-exists`, so a missing project only
loses that feature (the glyph apps, the Nothing Gallery, or the camera overrides).

Every project above carries an explicit `revision` because only the five
`packages_apps_*` repos have a `main` branch; the rest would otherwise inherit the
remote default and fail with `unknown revision`.

## Branch pairing

The device tree branch matters:

- `lineage-24.0-repos` — **use this one.** The app-free line: it has no in-tree copy
  of the apps and instead consumes the `packages/apps/*` repos above.
- `lineage-24.0` — do **not** pair this with this manifest. It still carries the apps
  at `app/parts`, `glyph/*`, `nt-fwk/` and `blob-patches/`, which is a different
  layout from the repos, so the synced projects would be ignored.

On the vendor side there is nothing to pair: `lineage-24.0-ntcam` used to be the only
branch with `NTCamera.apk`, `NTGallery.apk` and the camera libs, but `lineage-24.0`
has since been fast-forwarded onto it (both at `bd9e949`), so a single vendor
revision works for both device-tree lines.

## The device tree entry is commented out

`nothing-galaxian.xml` deliberately ships the `device/nothing/Galaxian` project
commented out:

```xml
<!--
<project path="device/nothing/Galaxian" name="device_nothing_Galaxian" remote="n3al" revision="lineage-24.0-repos" />
-->
```

A project path may only be declared **once** across the main manifest and every local
manifest — a duplicate is a hard error and aborts the whole sync. Most build hosts
already declare `device/nothing/Galaxian` somewhere (pointing at an older fork, which
still resolves because GitHub redirects transferred repos). To centralise it here,
delete that declaration first, then uncomment this one.

## Applying the kernel patches

The `patches` project is not a build project — nothing includes or inherits it — but
the ROM will not boot, flash, or record 60 FPS video without it. Each top-level
directory maps to a project path with `/` written as `__`
(`frameworks__base` → `frameworks/base`), and the applier runs `git am`:

```sh
python3 patches/apply.py patches "$PWD"
```

It needs a clean, already-synced tree, and stops on conflict with instructions for
resuming or aborting the `git am` by hand.

## Provenance

| Repo | Origin |
|---|---|
| `device_mediatek_sepolicy_vndr` | standalone mirror of `Andreyka445/device_mediatek_sepolicy_vndr` (itself forked from `nothing-galaga`); source kept as remote `upstream` |
| `hardware_mediatek` | fork of `LineageOS/android_hardware_mediatek`, upstream link kept |
| `hardware_dolby` | originally `samakshkambxj/hardware_dolby` |
| `device_nothing_Galaxian-kernel` | originally `samakshkambxj/android_device_nothing_Galaxian-kernel`, renamed to match `KERNEL_PATH` |
| `proprietary_vendor_nothing_Galaxian` | originally `samakshkambxj/vendor_nothing_galaxian`, renamed to the LineageOS blob-repo convention |
