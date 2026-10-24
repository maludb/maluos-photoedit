# MaluOS PhotoEdit status

> **Updated:** 2026-10-24

MaluOS PhotoEdit is the MaluOS photo editor, forked from
[storytold/photocraft](https://github.com/storytold/photocraft) on 2026-10-24 at upstream
`1c3db2e5c` (v0.15.1 + 406 commits, 5,773 commits since 2026-09-30, ~680k lines of Rust in 26
crates, 11,000+ tests). Remotes: `origin` = github.com/maludb/maluos-photoedit, `upstream` =
github.com/storytold/photocraft. Upstream code is untouched so far; this directory is the only
addition. The fork follows the MaluOS Slides pattern
([maluos-slides/docs/maluos-slides/plan.md](https://github.com/maludb/maluos-slides/blob/main/docs/maluos-slides/plan.md)):
Phase 1 proves build and install, Phase 2 renames and trims to Linux/MaluOS through a vendor
branch and a scripted rename.

## Phase 1: build and install on MaluOS (Ubuntu 26.04)

| Step | State | Notes |
|---|---|---|
| 1.1 repository | done 2026-10-24 | `maludb/maluos-photoedit` created (public), upstream history kept, pushed to `origin main` |
| 1.2 release build | done 2026-10-24 | `cargo build --release --locked -p photocraft -p photocraft-cli --features heif -j 6` under the memory-capped cgroup, 14m36s cold, craft-fonts embedded; rustc 1.99.1 (upstream requires 1.95); binaries 112 MB + 74 MB |
| 1.3 packaging | done 2026-10-24 | `packaging/linux/package.sh --skip-build --formats "deb tar"` with nfpm 2.48.0; `photocraft-0.15.1-linux-x86_64.deb` (84 MB) |
| 1.4 install | done 2026-10-24 | `apt-get install ./dist/release/photocraft-0.15.1-linux-x86_64.deb` on the dev VM; binaries in `/usr/bin`, desktop entry, MIME types (24 image formats), metainfo registered; copied to the dev VM's Apache folder (`/var/www/html/maluos/`) with a `.sha256` |
| 1.5 smoke test | done 2026-10-24 | `photocraft-cli convert` PNG → JPEG and PSD; `photocraft --control 7893 --control-token-file … --automation-write-root …` under Xvfb: token auth, `ui.inspect`, `engine.commands` (940 commands), `ui.screenshot` of the opened JPEG; GPU compositor falls back to CPU under Xvfb (expected, no GPU); settings in `~/.config/photocraft/` |

### Build recipe (dev VM)

System packages beyond the Rust toolchain (from upstream's CI): `libxkbcommon-dev`,
`libwayland-dev`, `libx11-dev`, `libxrandr-dev`, `libxi-dev`, `libgl1-mesa-dev`, `libgtk-3-dev`,
plus `pkg-config` and `cmake`; for packaging `nfpm` (in `~/.local/bin`), `desktop-file-utils`,
`appstream`; for the headless GUI test `xvfb`.

Fonts: upstream's release pins `storytold/craft-fonts` at `abb83316` (a different commit from
MaluOS Slides' pin), checked out as a worktree in `~/craft-fonts-photoedit`; there is no
`.cargo/config.toml` default, so pass `CRAFT_FONTS_DIR` explicitly.

```sh
systemd-run --user --scope -p MemoryHigh=4500M -p MemoryMax=6G -- \
  env CRAFT_FONTS_DIR=$HOME/craft-fonts-photoedit \
  cargo build --release --locked -p photocraft -p photocraft-cli --features heif -j 6
PATH=$HOME/.local/bin:$PATH CRAFT_FONTS_DIR=$HOME/craft-fonts-photoedit GITHUB_REPOSITORY=maludb/maluos-photoedit \
  packaging/linux/package.sh --skip-build --formats "deb tar"
sudo apt-get install ./dist/release/photocraft-0.15.1-linux-x86_64.deb
```

The binaries report "0.15.1 (dev build)" because the build ran outside `package.sh`, which
exports the commit and date; the release workflow will set them.

### Still upstream-branded

Everything says PhotoCraft: crate names (`photocraft-*`), binaries, app id
`ai.storyteller.photocraft`, env vars `PHOTOCRAFT_*`, the native format `.pcraft`, the ArtCraft
brand files under `docs/brand/`, and macOS, Windows and FreeBSD packaging.

## Next

Phase 2 for PhotoEdit, mirroring Slides: `cargo xtask ci` baseline (nextest profile `ci`,
`--features heif`); port `scripts/rename-upstream.sh` with a PhotoCraft mapping (crates
`maluos-photoedit-*`, binaries `maluos-photoedit` and `maluos-photoedit-cli`, app id
`org.maluos.photoedit`, env `MALUOS_PHOTOEDIT_*`, keep `.pcraft` and its MIME type, keep upstream
credits and test fixtures); drop non-Linux packaging, upstream workflows and dependabot; the
fork's own Ubuntu workflows; MaluOS identity, vendored deb in the maluos repo, boot test. Upstream
lands ~350 commits a day, so the vendor-branch sync must be frequent.
