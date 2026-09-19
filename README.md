# ish-AOK-rootfs

Manifest of downloadable root filesystems offered by [iSH-AOK](https://github.com/emkey1/ish-AOK)'s
"New Filesystem" picker. This repo is a git submodule of iSH-AOK
(`deps/rootfs-manifest`); the app bundles `manifest.json` at build time and
merges it with the small set of filesystems shipped inside the IPA itself.

This repo exists so anyone can propose a new rootfs without touching the
iSH-AOK application source.

The `archives/` directory in this repo holds the archive for every
`"tier": "official"` entry (and some community ones), served via
`raw.githubusercontent.com`. A `downloadURL` doesn't have to point here,
though — manifest entries are free to reference an archive hosted anywhere
stable.

(The original copies of these archives also remain attached to
[ish-AOK's `rootfs-assets` release](https://github.com/emkey1/ish-AOK/releases/tag/rootfs-assets)
for backwards compatibility with anything still linking there directly.)

## Contributing a new rootfs

1. Build (or otherwise obtain) a `.tar.xz` (preferred; `.tar.gz`/`.tar.zst`/`.tar.bz2` also work)
   root filesystem archive for a Linux guest architecture iSH-AOK supports:
   `i386`, `amd64` (x86_64), `arm64` (aarch64), or `riscv64`.
2. Either open a PR adding your archive under `archives/` directly (keep it
   under 100 MB — GitHub's hard per-file limit for a plain `git push`), or
   host it yourself somewhere stable and directly downloadable over HTTPS
   (avoid links that require auth, redirect through an HTML interstitial,
   or expire) and just reference that URL.
3. Add an entry to `manifest.json` (see schema below) and open a pull
   request. Set `"tier": "community"` — only iSH-AOK maintainers promote
   entries to `"official"`, which implies the image is covered by the
   project's regression suite.
4. In your PR description, note what you tested (does `/bin/login` work,
   does the package manager work, any known issues).

Once merged here, a maintainer bumps the `deps/rootfs-manifest` submodule
pointer in iSH-AOK and the new choice ships in the next app release.

## `manifest.json` schema

A JSON array of objects. All fields are required strings unless noted.

| Field | Description |
|---|---|
| `identifier` | Stable, unique, machine-readable id (letters/digits only, no spaces). Never reuse or change an existing identifier — it's persisted in exported root metadata. |
| `displayName` | Shown in the picker row when architecture isn't grouped (fallback / accessibility label). Keep it short; put "(Experimental)" style caveats here, not in `familyDisplayName`. |
| `archiveName` | Base filename (no extension) the downloaded archive is renamed to on-device. Should be unique per identifier. |
| `importName` | Becomes the on-device root directory name and mount "source" string. Must match iSH-AOK's `RootNameIsValid` rules: letters, digits, `.`, `-`, `_` only, no spaces, can't start with `.`. |
| `initialWindow` | Currently always `"session-shell"`. |
| `guestABI` | One of `i386`, `amd64`, `arm64`, `riscv64` — must match the archive's actual userland architecture. |
| `downloadURL` | Direct HTTPS URL to the `.tar.*` archive. Must return the file directly (no HTML interstitial). |
| `downloadSize` | Human-readable approximate download size, e.g. `"~32 MB"`. Shown to the user before they download. |
| `family` | Groups architecture variants of the same distro/release into one picker row (e.g. all three Alpine 3.23.3 entries share `"alpine3233"`). Use a new family value per distro *and* per release/version you want listed separately. |
| `familyDisplayName` | Shown for the grouped row; architecture is chosen as a sub-choice. |
| `tier` | `"official"` or `"community"`. New PRs should use `"community"`. |
| `series` | *Optional.* Stable id for a line of images republished over time from the same source (e.g. `"pscal"`). Requires `version`. |
| `version` | *Optional.* This build's version within its `series`, e.g. `"2026.09.19"`. Requires `series`. |

Keep `identifier`/`archiveName`/`importName` free of ambiguity with existing
entries in this file — the app dedupes by `identifier`.

## Versioned images (`series` / `version`)

Most entries here are a distribution's own release and are replaced wholesale
when that release changes. Some images are built by this project instead, from
sources that keep moving — the PSCAL + SmallCLUE rootfs is the first — so a new
build is published every so often with the same contents but newer code.

Those entries carry a `series` (which line of images this is) and a `version`
(which build). Every version ever published stays listed here: this file is the
record of what exists, and the archives stay downloadable. iSH-AOK offers only
the **two most recent versions of each series** in its picker, so the list does
not grow a row on every rebuild. Versions are compared numerically (`2026.9.19`
sorts before `2026.10.1`, not after), and two architectures of the same version
count as one version.

A superseded version disappears from the picker but is not withdrawn: it is
still installable by identifier via
`ish-cli roots install source=catalog id=<identifier>`.

**Never delete an archive from `archives/`.** The app bundles a *copy* of
`manifest.json` at build time, so every iSH-AOK already installed keeps asking
for whatever URLs its own copy names — deleting a file 404s those installs with
"Couldn't download the filesystem image", and they cannot be fixed from here.
Dropping an entry from this file is fine and only affects future app builds;
the file it pointed at has to stay. If a published build turns out to be
defective, replace its bytes with a fixed image rather than removing it.

Give each version its own `family` as well, so the two offered builds appear as
two separate rows rather than being folded into one.
