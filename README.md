# OpenEmux Flatpak repository

The Flatpak distribution channel for
[OpenEmux](https://github.com/guilhermefeitosa66/OpenEmux) (OpenEmux issue
[#67](https://github.com/guilhermefeitosa66/OpenEmux/issues/67)). It lives in
its own repository so the ostree objects never bloat the app's repo.

The actual Flatpak repo (ostree) is published from the `gh-pages` branch via
GitHub Pages.

## Install

```bash
# One-time: the OpenEmux remote, and RetroArch (which plays the games)
flatpak remote-add --if-not-exists --no-gpg-verify openemux \
  https://guilhermefeitosa66.github.io/openemux-flatpak/repo
flatpak install -y flathub org.libretro.RetroArch

flatpak install -y openemux io.github.guilhermefeitosa66.OpenEmux
```

Updates then arrive with everything else:

```bash
flatpak update
```

Alternatively, every [OpenEmux release](https://github.com/guilhermefeitosa66/OpenEmux/releases)
ships a single-file `OpenEmux-<version>.flatpak` bundle:
`flatpak install ./OpenEmux-<version>.flatpak` (no remote needed, no
auto-update).

The repo is unsigned (hence `--no-gpg-verify`): it is served read-only over
HTTPS from GitHub Pages, which is also the trust anchor for the source code
itself.

## Architectures

`x86_64` and `aarch64`, from one remote. One ostree repo serves several
architectures, so an ARM machine finds its build behind the same URL and
`flatpak update` picks it up with everything else -- nothing to add and nothing
to choose.

The `aarch64` build uses whatever RetroArch the host provides
(`org.libretro.RetroArch` is on Flathub for ARM). The Linux `.deb`, `.rpm` and
AppImage bundle a RetroArch on x86_64 and cannot on ARM, because libretro
publishes no ARM binary -- the Flatpak is unaffected, since it has always
launched the host's RetroArch rather than one of its own.

## Publishing (maintainers)

`.github/workflows/publish.yml` builds a given OpenEmux git ref with
`flatpak-builder` and pushes the updated ostree repo to `gh-pages`. Run it
from the Actions tab (default ref: the latest release tag) after each release.

It builds once per architecture, one after the other rather than in parallel:
both legs push to the same `gh-pages` branch, and serialising them is what lets
the second build on top of the first instead of racing it. A failure in one
leg does not withdraw the other, so an ARM build that breaks leaves the x86_64
users' update alone.
