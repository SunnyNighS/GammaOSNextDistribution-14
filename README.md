# GammaOS Next: RG DS Plus / nano fork

This is an independent fork of
[GammaOS Next](https://github.com/TheGammaSqueeze/GammaOSNextDistribution-14) (Android 14 /
LineageOS 21 GSI). It concentrates on the **Anbernic RG DS Plus** and on GammaOS **nano mode**:
drastic-nano (DS), azahar-nano (3DS), the DSi-style home and the parts of the framework they
depend on.

**This is not the official GammaOS.** The upstream maintainer does not endorse it or support
it. For official releases and support, use the original project.

## Developed with Claude Code

Development in this fork is done with [Claude Code](https://claude.com/claude-code), Anthropic's
coding agent:

- It hunts bugs, writes the fixes and reviews them.
- Every change is tested on a retail RG DS Plus before it is published here.
- Each commit is one logical change, and its message explains what changed and why.

The work continues here, whether or not upstream takes it.

## Relationship to upstream

- Every change so far is also offered upstream as
  [PR #26](https://github.com/TheGammaSqueeze/GammaOSNextDistribution-14/pull/26).
- Upstream `develop` is merged in from time to time.
- Branches:
  - `develop` (default): the fork, the `rgdsplus` branch plus this README.
  - `rgdsplus`: the PR branch, exactly what is offered upstream.
- Kernel fixes for the RG DS Plus live in
  [SunnyNighS/rg_ds_kernel](https://github.com/SunnyNighS/rg_ds_kernel/commits/rgdsplus-main),
  branch `rgdsplus-main`.

## What is different

Highlights. The full list is in PR #26 and in the commit log.

- **drastic-nano:**
  - GPU 3D speedups, and DS fog done right in a post pass
  - 44.1→48 kHz resampling with fades
  - sleep, wake and lid handling that keeps the vblank lock
  - Select+Vol brightness that repeats while held and never steps against the press
- **nano home:**
  - native-resolution covers and text, and vector chrome
  - a boot progress bar
  - Menu tap returns to the last game, Menu hold opens the Control Center
  - durable Recently Played, Favorites and Collections
- **Input:**
  - held buttons survive menus, hotplug and sleep
  - `drop_input` releases what an app still holds
  - gammapad combos stay out of DS and 3DS games
- **Framework in nano mode:**
  - fewer boot-time stalls
  - display power ordering for the two panels
  - power group and wake-lock fixes
- **System:**
  - swap files of 2 GiB and up, set up at idle I/O priority
  - the RG DS Plus SELinux policy cache and IRQ placement
- **Packaging:** the tv image carries azahar-nano again.

Changes specific to one device are gated on `ro.gammaos.device=anbernicrgdsplus`, so other
devices keep the upstream behaviour.

## Building

You need:

- an x86_64 Linux host. The builds here run on an Ubuntu 24.04 based system.
- at least 300 GB of free disk space
- as much RAM as you can give it

Install the usual AOSP host packages, then check out the tree. It is a full tree, not a `repo`
manifest. About 400 prebuilts (clang, JDK, Rust and others) come through Git LFS.

```sh
sudo apt install git git-lfs gnupg flex bison build-essential zip curl zlib1g-dev \
  libc6-dev-i386 x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils \
  xsltproc unzip fontconfig python3
git lfs install
git clone --branch develop https://github.com/SunnyNighS/GammaOSNextDistribution-14.git
cd GammaOSNextDistribution-14
git lfs pull
bash buildtv.sh
```

How the build works:

- `buildtv.sh` builds the TV product `lineage_tv_arm64_bvN`, which the RG DS Plus runs.
  `build.sh` builds the non-TV image.
- The first run downloads the Android NDK r27c and CMake into `out/` and builds the azahar-nano
  core. `AZAHAR_NANO_SKIP_BUILD=1` keeps the committed libraries instead.
- The system image ends up in `out/home/build-output/lineage-21.0-<date>-UNOFFICIAL-tv_arm64_bvN.img`.

To build the kernel and the Wi-Fi module for the RG DS Plus:

```sh
git clone --branch rgdsplus-main https://github.com/SunnyNighS/rg_ds_kernel.git
cd rg_ds_kernel
rgds/build-boot-images.sh   # out/boot.img
rgds/build-wifi.sh          # out/8821cs.ko for /vendor/lib/modules
```

`rgds/README.md` in the kernel tree has the details. A new `boot.img` needs the `8821cs.ko`
built from the same tree, or Wi-Fi does not come up.

## Installing

No prebuilt images are published here. The build gives the same kind of system image as
upstream, so install it the way you install a GammaOS Next system image on your device. Back up
your saves first.

The Updater still checks the official GammaOS OTA server. Installing an official update
replaces a build of this fork.

## License

Same as upstream: Apache-2.0 for the GammaOS and AOSP code. Third-party components keep their
own licenses.
