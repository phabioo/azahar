# Azahar for Uzume

This fork builds the Azahar libretro core for [Uzume](https://github.com/phabioo/uzume). It is upstream Azahar plus the small libretro patches below, which Uzume plugins need. It is not an official Azahar build; report problems with it to the Uzume project, not to Azahar.

## Patches

- **Memory map follows the game** (`src/citra_libretro/citra_libretro.cpp`): the core sent `RETRO_ENVIRONMENT_SET_MEMORY_MAPS` once at load, before the game maps its heap, so frontends saw only VRAM. The core now keeps the map tied to the application process, sends it again after a frame when the exposed VMAs changed, and splits VMAs that are not naturally aligned powers of two instead of skipping them. Needed by the SoulLink Core plugin, which reads game memory. Reported upstream as an issue (no code, per Azahar's `AI-POLICY.md`).
- **Cheats from the frontend** (same file): `retro_cheat_set` and `retro_cheat_reset` were empty. They now hand Gateway codes to Azahar's own cheat engine (`src/core/cheats`). Lines may be separated by `+`, newlines or spaces; a line may be one 16-digit token. Cheats from the cheats folder of the title stay as they are. Needed by the Uzume cheats plugin.

## Branches and builds

- `master` mirrors upstream Azahar and carries no patches.
- The patches live on a branch that is rebased onto upstream `master` when Azahar moves on.
- `.github/workflows/uzume-core.yml` builds the core for `windows-x64` and `linux-x64` on every push to that branch and publishes a release `uzume-core-<date>-<commit>` with:
  - `azahar-uzume-import.zip`: the layout of `uzume-server import-cores` (`<platform>/azahar_libretro.<suffix>.zip` plus `<platform>/.index-extended` with build date and CRC32),
  - the single core zips and `SHA256SUMS.txt`.

## Install on an Uzume server

1. Download `azahar-uzume-import.zip` from the latest release and unzip it into an empty folder.
2. Import it: `uzume-server import-cores <folder>` (systemd: `sudo packaging/linux/install-server.sh import-cores <folder>`, Docker: `docker exec uzume-server uzume-server import-cores /data/<folder>` with the folder in the volume).
3. In the app, Server › Systems & Cores › Nintendo 3DS: make Azahar the default core if it is not, and turn its automatic update off (pin it). Otherwise the next buildbot build replaces the patched core.

## Updating the fork

```
git fetch upstream master
git rebase upstream/master   # on the patch branch
git push --force-with-lease
```

The push builds and publishes a new release. Drop a patch once upstream Azahar ships the same fix.
