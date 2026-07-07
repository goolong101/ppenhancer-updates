# PP Doctor ↔ ppenhancer-updates — OTA update spec

How **PP Doctor** (the Windows/macOS cabinet manager) updates a PinnerPi backglass
from this public channel. Source lives in the PRIVATE `goolong101/ppenhancer`; this
repo ships only artifacts + metadata.

## Channels
- **prod** (default for users): `manifest.json` on `main`; binaries = latest **full**
  GitHub Release.
- **test** (your own cab): `manifest-test.json` on `main`; binaries = latest
  **prerelease** Release. Give PP Doctor a channel toggle; default = prod.

## Update algorithm
1. **Read installed version** from the cab: `ssh pi@<cab> cat /home/pi/PinnerPi/VERSION`.
2. **Fetch channel manifest** (no clone needed):
   `GET https://raw.githubusercontent.com/goolong101/ppenhancer-updates/main/<manifest>`
   (`manifest.json` for prod, `manifest-test.json` for test).
3. **Compare** `manifest.version` (semver) vs installed. Newer ⇒ update available.
4. **Download each artifact** to a local staging dir:
   `download_base + "/" + asset`  (download_base = `.../releases/download/v<ver>`).
5. **Verify** each file's md5 against `artifacts[].md5`. Abort the whole update if any
   mismatch — never push a partial/corrupt set.
6. **Push to cab** (per artifact): `scp` staged file → `/home/pi/PinnerPi/<dest>`.
   `chmod +x` when `exec:true`. For `pinnerpi_sdl` update BOTH `build/pinnerpi_sdl`
   AND the root `./pinnerpi_sdl` (the daemon execs the ROOT copy — see the pinnerpi
   binary/golden rules).
7. **Apply safely**: `sudo systemctl stop pinnerpi.service` → swap files → `start`.
   (Or drop the set into `media/__updates/` and let the daemon's transfer-mode handler
   install it.) Verify `pgrep -x pinnerpi_sdl` after; if it crash-loops, __golden
   auto-restores — so **refresh __golden after a verified update**.
8. **Bump the cab's `/home/pi/PinnerPi/VERSION`** to the new version so step 1 is correct
   next time.

## manifest.json schema
```
{ "version":"0.2.0", "release_tag":"v0.2.0", "channel":"stable",
  "download_base":"https://github.com/goolong101/ppenhancer-updates/releases/download/v0.2.0",
  "artifacts":[ { "asset":"pinnerpi_sdl", "dest":"build/pinnerpi_sdl",
                  "md5":"…", "size":2852072, "exec":true }, … ] }
```
- `asset` = Release asset filename (basename). `dest` = path under `/home/pi/PinnerPi/`.
- `exec` = chmod +x on the cab. `md5`/`size` = integrity check.

## Guardrails PP Doctor must honor
- md5-verify **before** touching the cab; all-or-nothing.
- Stop the service before swapping the running binary.
- Keep root `./pinnerpi_sdl` and `build/pinnerpi_sdl` identical.
- Refresh `__golden` only AFTER the new build verifies healthy on hardware.
- The cab's local mirror is the source of truth for what it currently renders; PP Doctor's
  preview must match the cab render.
