# aic8800 for OpenWrt — standalone apk packages

AICSemi **AIC8800** Wi-Fi driver, packaged for OpenWrt and built **outside** any
OpenWrt tree: the CI here pulls the official SDK, builds the packages against it
and publishes them as `.apk` files on a release.

Built for **OpenWrt 25.12.5, rockchip/armv8** (`aarch64_generic`).

## Why it is not in the firmware tree

The driver is an out-of-tree vendor blob-loader that upstream OpenWrt would not
take, and it has no business in a device tree either. Keeping it here means the
firmware branches stay buildable from a clean OpenWrt checkout, and the driver
can be rebuilt on its own schedule.

Using the *official* SDK is the other half of that: the SDK carries the kernel
config the released images were built with, so the modules come out with the
published vermagic and install on a stock image. The build asserts that — if the
ABI ever drifts, the run fails rather than shipping a module that cannot load.

## What it is for

The SDIO packages drive the **AIC8800D80** found on boards like the FriendlyElec
NanoPi R28S (module `SKI.WB800D80S.1`), which has no mainline driver. On those
builds the device tree already describes the SDIO host, so the card is detected
at boot — there is simply nothing bound to it until these packages are
installed.

## Install

```
apk add --allow-untrusted ./aic8800-sdio-firmware-*.apk ./kmod-aic8800-sdio-*.apk
apk add wpad-basic-mbedtls        # if the image ships no supplicant
reboot
```

Bluetooth over SDIO is compiled in; apk pulls `kmod-bluetooth` in with it.

## Building a release

Push a tag:

```
git tag aic8800-25.12.5-r1 && git push origin aic8800-25.12.5-r1
```

The workflow downloads the SDK, adds `package/` here as a feed, builds
`kmod-aic8800-sdio` plus `aic8800-sdio-firmware`, checks the kernel ABI, and
publishes the apk files.

## Source and credit

* Driver: [radxa-pkg/aic8800](https://github.com/radxa-pkg/aic8800), pinned at
  `89f865b8`.
* OpenWrt packaging (`package/aic8800/Makefile` and its `patches/`): taken from
  ImmortalWrt's `package/kernel/aic8800`. Those patches were written against
  backports **6.18.39**, which is exactly the version OpenWrt 25.12 ships — the
  cfg80211 API the driver compiles against is the same on both sides, which is
  why they transplant cleanly, byte for byte.

  The Makefile carries one addition: ImmortalWrt builds against a 6.18 kernel,
  and 6.12 promotes `-Wmissing-prototypes` and `-Wexpansion-to-defined` to
  errors, which the vendor source trips in ten places. `AIC_KCFLAGS` demotes
  those two back to warnings. Nothing about types or implicit declarations is
  touched — a warning that can hide a real defect stays fatal.

Other prior art worth knowing about, none of it used directly here:
[firtel-t/aic8800-sdio-openwrt](https://github.com/firtel-t/aic8800-sdio-openwrt)
(same idea for the Radxa Zero 3W, driving the radxa debian patches straight from
upstream) and
[nickbash11/aic8800-usb_openwrt](https://github.com/nickbash11/aic8800-usb_openwrt)
(the USB variant).
