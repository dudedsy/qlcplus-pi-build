# qlcplus-pi-build

Builds [QLC+](https://github.com/mcallegari/qlcplus) `.deb` packages for
Raspberry Pi OS Trixie (64-bit) on GitHub's free arm64 runners. It uses the
upstream Debian packaging and `create-deb.sh` unchanged. By default nothing is
patched; optional patches live in `patches/` (see `patches/README.md`).

## Build

Actions tab → **Build QLC+ for Pi** → **Run workflow**. Inputs:

- `qlc_tag`: upstream tag, e.g. `QLC+_5.3.0`
- `flavour`: `v5` (QML UI, binary `qlcplus5`) or `v4` (widget UI, binary `qlcplus`)
- `qlc_repo`: source repo, default `mcallegari/qlcplus` (change only to test a fork)
- `patches`: optional, space-separated file names from `patches/` to apply,
  e.g. `universe-tick-direct.patch`. Patched builds get their own version and prerelease.
- `publish_release`: attach the packages to a GitHub Release

Jobs: build in a `debian:trixie` arm64 container, then a smoke test that
installs the packages into a clean Trixie container, checks every library and
plugin resolves, and runs `--version`, then the release.

## Install on the Pi

```
sha256sum -c --ignore-missing SHA256SUMS
cp qlcplus*.deb /tmp/
sudo apt install --no-install-recommends \
  /tmp/qlcplus-data_5.3.0_all.deb /tmp/qlcplus5_5.3.0_arm64.deb \
  qt6-svg-plugins qt6-wayland fonts-dejavu-core
```

- `qt6-svg-plugins` is required: without it every QLC+ 5 icon fails to load
  ("Unsupported image format") and buttons render blank. The upstream packaging
  doesn't depend on it, and the smoke test can't catch it because `--version`
  never loads the UI.
- `qt6-wayland` is needed to run under a Wayland compositor such as labwc.
- Installing from `/tmp` avoids apt's harmless "unsandboxed as root" notice.
- On an 800x480 screen, lower QLC+'s UI scale (app menu → UI Settings →
  Scaling factor) or the top bar runs off the right edge.

## Notes

- `qlcplus-data` from this build replaces Debian's `qlcplus-data`. Don't also
  install Debian's `qlcplus` (v4) package; it depends on its own matching
  `qlcplus-data` version.
- The repo must be public for the arm64 runners to be free.
