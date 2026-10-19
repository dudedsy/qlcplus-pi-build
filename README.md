# qlcplus-pi-build

Builds [QLC+](https://github.com/mcallegari/qlcplus) `.deb` packages for
Raspberry Pi OS Trixie (64-bit) on GitHub's free arm64 runners. It uses the
upstream Debian packaging and `create-deb.sh` unchanged; nothing is patched.

## Build

Actions tab → **Build QLC+ for Pi** → **Run workflow**. Inputs:

- `qlc_tag`: upstream tag, e.g. `QLC+_5.3.0`
- `flavour`: `v5` (QML UI, binary `qlcplus5`) or `v4` (widget UI, binary `qlcplus`)
- `publish_release`: attach the packages to a GitHub Release

Jobs: build in a `debian:trixie` arm64 container, then a smoke test that
installs the packages into a clean Trixie container, checks every library and
plugin resolves, and runs `--version`, then the release.

## Install on the Pi

```
sudo apt install ./qlcplus-data_5.3.0_all.deb ./qlcplus5_5.3.0_arm64.deb
```

Check `SHA256SUMS` from the release first.

## Notes

- `qlcplus-data` from this build replaces Debian's `qlcplus-data`. Don't also
  install Debian's `qlcplus` (v4) package; it depends on its own matching
  `qlcplus-data` version.
- The repo must be public for the arm64 runners to be free.
