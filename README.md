# docker-gnucash

[GnuCash](https://www.gnucash.org/) running as a full desktop application inside your browser — no local install required.

This is a fork of [linuxserver/docker-calibre](https://github.com/linuxserver/docker-calibre), retargeted from Calibre to GnuCash. It builds on [`ghcr.io/linuxserver/baseimage-selkies`](https://github.com/linuxserver/docker-baseimage-selkies), which streams a real Linux desktop to your browser over WebRTC/websockets (via [Selkies](https://github.com/selkies-project/selkies)), so GnuCash runs exactly as it would on a desktop — just accessed at a URL instead of a window.

This is a personal build, not an official LinuxServer.io or GnuCash project.

## Quick start

There's no published image yet — build it locally:

```bash
git clone https://github.com/theblaz/docker-gnucash.git
cd docker-gnucash
docker build -t docker-gnucash .
```

### docker-compose

```yaml
---
services:
  gnucash:
    image: docker-gnucash
    container_name: gnucash
    security_opt:
      - seccomp:unconfined #optional, see Security note below
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - PASSWORD= #optional
      - CLI_ARGS= #optional, e.g. path to a .gnucash file to open on startup
    volumes:
      - /path/to/gnucash/config:/config
    ports:
      - 8080:8080
      - 8181:8181
    shm_size: "1gb"
    restart: unless-stopped
```

### docker cli

```bash
docker run -d \
  --name=gnucash \
  --security-opt seccomp=unconfined `#optional` \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e PASSWORD= `#optional` \
  -e CLI_ARGS= `#optional` \
  -p 8080:8080 \
  -p 8181:8181 \
  -v /path/to/gnucash/config:/config \
  --shm-size="1gb" \
  --restart unless-stopped \
  docker-gnucash
```

Once running, open `https://yourhost:8181/` — GnuCash will already be open and maximized on the desktop.

## Parameters

| Parameter | Function |
| :----: | --- |
| `-p 8080:8080` | desktop GUI (HTTP, only useful behind a reverse proxy) |
| `-p 8181:8181` | desktop GUI (HTTPS) |
| `-e PUID=1000` | UserID — see [User / Group Identifiers](#user--group-identifiers) |
| `-e PGID=1000` | GroupID — see [User / Group Identifiers](#user--group-identifiers) |
| `-e TZ=Etc/UTC` | timezone, see this [list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List) |
| `-e PASSWORD=` | optional HTTP basic auth password for the `abc` user |
| `-e CLI_ARGS=` | optional args passed straight to `gnucash`, e.g. a file path to open a book on startup |
| `-v /config` | persistent home directory — GnuCash preferences live in `/config/.config/gnucash`; **save your book files somewhere under `/config`** (e.g. `/config/Documents`) so they survive container recreation |
| `--shm-size=` | recommended for all desktop/selkies-based images |
| `--security-opt seccomp=unconfined` | some GUI syscalls aren't in Docker's default seccomp profile; see [Security note](#security-note) before using this |

This image is built on `baseimage-selkies`, which supports a large set of additional environment variables (GPU acceleration, Wayland tuning, UI hardening, watermarking, etc.) — see the [baseimage-selkies README](https://github.com/linuxserver/docker-baseimage-selkies) for the full list. `TITLE` defaults to `GnuCash` in this image.

## Security note

> [!WARNING]
> This container provides privileged access to the host system. Do not expose it to the Internet unless you have secured it properly.

- HTTPS is required for full functionality — connect via port `8181`, not `8080`.
- There is no authentication by default. `PASSWORD` enables basic HTTP auth, which is only suitable for a trusted local network. For internet exposure, put this behind a reverse proxy (e.g. [SWAG](https://github.com/linuxserver/docker-swag)) with real authentication.
- The web desktop includes a terminal with passwordless `sudo`. Anyone with GUI access has root in the container.
- `--security-opt seccomp=unconfined` disables a Docker security layer. Only use it if GnuCash fails to start without it (older kernels/libseccomp).

## Known limitation: in-app Help

GnuCash's Help menu opens `yelp`, which renders pages via WebKitGTK. WebKitGTK sandboxes its renderer process using `bubblewrap`, which needs to mount a nested `/proc`. Most Docker hosts strip the capability (`CAP_SYS_ADMIN`) required for that by default, so **Help currently does not open** in this image. Granting `--cap-add=SYS_ADMIN` fixes it but weakens container isolation, so it isn't done here — the docs (`gnucash-docs`) are still installed and readable directly at `/usr/share/help/C/gnucash-guide` and `/usr/share/help/C/gnucash-manual` inside the container if needed, or read online at [gnucash.org/docs.phtml](https://www.gnucash.org/docs.phtml).

## User / Group Identifiers

When using volumes (`-v`), permission mismatches between the host and container are avoided by setting `PUID`/`PGID` to match a host user:

```bash
id your_user
```

```text
uid=1000(your_user) gid=1000(your_user) groups=1000(your_user)
```

## Building locally

```bash
git clone https://github.com/theblaz/docker-gnucash.git
cd docker-gnucash
docker build --no-cache --pull -t docker-gnucash .
```

An `aarch64` variant is provided as `Dockerfile.aarch64` (inherited from the upstream Calibre fork; not verified against GnuCash).

## Credits

- Based on [linuxserver/docker-calibre](https://github.com/linuxserver/docker-calibre) and [linuxserver/docker-baseimage-selkies](https://github.com/linuxserver/docker-baseimage-selkies).
- [GnuCash](https://www.gnucash.org/) is developed by the GnuCash project.
- Not affiliated with or endorsed by LinuxServer.io or the GnuCash project.

## License

Inherited from the upstream fork: [GPLv3](LICENSE).
