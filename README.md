EN | [中文](README_CN.md)

# voxl-releases

Compiled builds of [voxl](https://github.com/tembo/voxl) for Linux x86_64 and Windows x86_64.
Download a binary here instead of installing Rust and building from source.

voxl is a fast remote-desktop streamer: it serves an X11 display (Linux) or a Windows desktop as H.264
over a single WebSocket, and you watch it in a browser — no client to
install. Upstream project (MIT): https://github.com/tembo/voxl

## Download

Go to [Releases](https://github.com/popeico/voxl-releases/releases) and
download the asset for your version:

- `voxl-linux-x86_64` — the binary (rename it to `voxl` if you like)
- `voxl-linux-x86_64.sha256` — checksum file
- `voxl-linux-x86_64-musl` — dynamically linked binary for musl-based
  systems (Alpine, etc.); needs the runtime libraries (see below)
- `voxl-linux-x86_64-musl.sha256` — checksum file for the musl binary
- `voxl-windows-x86_64.zip` — portable Windows build: `voxl.exe` + a
  short README, no installer, unzip anywhere and run
- `voxl-windows-x86_64.zip.sha256` — checksum file for the zip
- `build-info.txt` / `build-info-musl.txt` / `build-info-windows.txt` — upstream commit, version,
  build date, sha256 for each variant

Verify the download:

```sh
sha256sum -c voxl-linux-x86_64.sha256
chmod +x voxl-linux-x86_64
./voxl-linux-x86_64 --version
```

The musl binary verifies the same way (`sha256sum -c
voxl-linux-x86_64-musl.sha256`, then run it — after installing its
runtime libraries, below).

On Windows, verify the zip with certutil and compare the hash with
`voxl-windows-x86_64.zip.sha256`:

```powershell
certutil -hashfile voxl-windows-x86_64.zip SHA256
```

## Runtime requirements

Linux x86_64 with an X11 display to stream. The glibc binary links
the video / audio libraries dynamically, so install them first
(Debian / Ubuntu):

```sh
sudo apt-get install -y libx264-dev libyuv-dev libopus-dev libpulse-dev \
  libxcb1-dev libxcb-shm0-dev libxcb-randr0-dev libxcb-xfixes0-dev \
  libxcb-shape0-dev
```

The `voxl-linux-x86_64-musl` binary is dynamically linked against
musl libc and needs the same libraries installed on the target
system (Alpine):

```sh
apk add opus pulseaudio x264-libs libyuv libxcb
```

Why not fully static? Alpine ships no static libopus or libpulse —
there is no opus-static / pulseaudio-static subpackage and no .a in
opus-dev / pulseaudio-dev (verified against the v3.22 APKINDEX).
A fully static musl binary would need opus and pulseaudio (plus
their transitive deps) built from source for musl; that is not done
here. `file` / `ldd` output for the musl binary is recorded in
`build-info-musl.txt`.

The Windows build is portable: unzip `voxl-windows-x86_64.zip`
anywhere, no installer, no admin rights. x264, libyuv and Opus are
statically linked, so there is nothing else to install beyond the C
runtime that ships with Windows 10+.

No Rust toolchain, no Node, no build step — those are only needed to
compile it, and the build already happened here.

## Quick start

On the machine whose display you want to stream (display `:1` here):

```sh
DISPLAY=:1 ./voxl-linux-x86_64 --port=6901
```

It prints a link with a sign-in token — open it in a browser. Add
`?stats` to the URL for an FPS / bitrate overlay. See upstream
[README](https://github.com/tembo/voxl#readme) for all flags
(`--title`, `--framerate`, `--enable-basic-auth`, clipboard, recording).

On Windows, after unzipping:

```powershell
.\voxl.exe --port=6901
```

No `DISPLAY` needed — it streams the Windows desktop.

Serving through a public hostname (reverse proxy / tunnel) needs both
flags, or the browser WebSocket is refused:

```sh
DISPLAY=:1 ./voxl-linux-x86_64 --port=6901 \
  --allowed-hosts=<your-host> --allowed-origins=https://<your-host>
```

## Troubleshoot

- **Stream never starts / WS refused in the browser:** you are missing
  `--allowed-origins` (see above). `--allowed-hosts` alone is not enough
  behind a public hostname.
- **Encoder dies on start (`height not divisible by 2`):** your display
  resolution has an odd width or height. Use an even resolution, e.g.
  1280x720 or 1920x1080.
- **`pulse connect failed` repeats in the log:** no PulseAudio server in
  the session, so there is no audio. Video is unaffected.
- **`error while loading shared libraries`:** a runtime library from the
  list above is missing — install it and retry.
- **Windows SmartScreen blocks the first launch:** the binary is
  unsigned. Click "More info", then "Run anyway".

## About these builds

- Source: [tembo/voxl](https://github.com/tembo/voxl) (MIT License).
  Each release notes the exact upstream commit built.
- Build definitions live in the private repo `popeico/build-factory`
  (`voxl/` source mirror + `.github/workflows/voxl.yml`). This repo holds
  release files only — no source mirror.
- Upstream ships no prebuilt binary (only the `.crate` source package),
  which is why these builds exist. If upstream starts publishing
  prebuilds, these builds stop.

## Install with mise

Releases are signed with [packslip](https://packslip.dev), so
[mise](https://mise.jdx.dev) installs them with no extra setup:

```sh
mise use packslip:github.com/popeico/voxl-releases@0.3.5
```

That writes this line to `mise.toml`:

```toml
"packslip:github.com/popeico/voxl-releases" = "0.3.5"
```

The signer fingerprint for this release is
`ps1_7fev5yvmlonnngel5jnurbtgzu` (verify with
`packslip pin packslip.sigstore.json` on the release bundle).

On musl hosts (Alpine) mise picks the musl build automatically. On
Windows hosts it picks the portable zip and installs it as `voxl.exe`.
The glibc build needs the runtime libraries listed above, including
`libx264.so.165` (x264 0.165), which Ubuntu 24.04 LTS does not ship —
on 24.04 use the musl build or a newer distro.
