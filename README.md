# ppenhancer-updates

Public OTA update channel for the **PinnerPi backglass** (`ppenhancer`) — Raspberry Pi Zero 2W.
Source is private (`goolong101/ppenhancer`); this repo ships only build artifacts + metadata.

- `VERSION` — current stable version string (semver).
- `manifest.json` — per-artifact `md5` + `size` for the current version. The cabinet
  compares its installed files against this to decide what to pull.
- Binaries are distributed via **GitHub Releases** (tag = version); this repo holds the
  version/manifest so a cabinet can check "am I current?" cheaply.

The Pi already supports offline updates via USB transfer mode (`media/__updates/`); this
channel is the network/OTA path.
