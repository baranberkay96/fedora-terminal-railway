# Fedora 45 Web Terminal (Browser Shell): Railway template spec

## Listing

| Field | Value |
|---|---|
| Name | **Fedora 45 Web Terminal (Browser Shell)** ("45" is the current stable release; 46 is Rawhide) |
| Description | Persistent Fedora 45 shell in your browser, password-protected. (63 chars) |
| Category | Other |
| Tags | fedora, terminal, web-terminal, linux, shell (the first five are sent), ttyd, tmux, fedora-45, dnf |
| Icon | `https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/fedora.svg` (200, image/svg+xml) |

Why: the Fedora niche is small (about 11 deploys in 30 days, marketplace data 2026-11). The only Fedora shell template runs bare `fedora:latest` with `tail -f /dev/null`: no web access and no password. This package brings our Ubuntu/Debian/Kali/Alpine web terminal design to Fedora at almost no extra maintenance cost.

## Base

Derived from `116-ubuntu-terminal` and `127-alpine-terminal` (same nginx auth proxy, `/healthz`, ttyd auth-header mode, tmux sessions, fail-closed password handling per STANDARD 8.29, build-time extra packages like 109's `KALI_EXTRA_PACKAGES`).

Fedora-specific changes:
- `dnf upgrade --refresh` plus one `dnf install --allowerasing --setopt=install_weak_deps=False` (dnf5); ttyd 1.7.7 and tini come from Fedora.
- Node.js 24 LTS from the nodejs.org tarball with SHA-256 (Fedora 45's default `nodejs` is 22), as in 116.
- GCC, G++, make and `python3-devel` are included, so native npm/pip modules build.
- nginx runs as `nginx`; temporary paths in `/run/terminal-nginx`; nginx starts with `-e stderr`.

## Pinned versions (checked 2026-11-20)

| Component | Source | Version |
|---|---|---|
| Base image | `fedora:45@sha256:a7d72ba2745b100a3351087472e5ee7b2450af7cf331e7673007e3817612839c` | Fedora 45 |
| ttyd | Fedora 45 | 1.7.7 |
| Node.js | nodejs.org tarball, SHA-256 checked | 24.22.3 LTS |
| nginx | Fedora 45 updates | 1.30 |
| tmux | Fedora 45 updates | 3.8 |
| tini | Fedora 45 | 0.19.0 |

Every build runs `dnf upgrade`, so Fedora 45 updates are picked up.

## Resource estimate

Railway prices: RAM $10/GB-month, vCPU $20/vCPU-month, volume $0.15/GB-month.

| Service | Idle (measured) | Typical | Notes |
|---|---|---|---|
| Terminal | 0.022 GB RAM, ~0 vCPU (measured, 0.5 GB limit) | 0.3 GB avg | Peaks come from what users run |
| Volume `/home` | 0.1 GB | 2-5 GB | Projects, venvs, node_modules |

Estimated monthly cost: idle about $0.40, typical about $4. Minimum plan: Free. Image about 0.4 GB compressed (estimate: 72 MB base plus the dnf set and Node.js), below the 4 GB cap.

## Services

### Terminal

| Setting | Value |
|---|---|
| Source | repo `baranberkay96/fedora-terminal-railway`, root `/services/terminal` (Dockerfile) |
| Healthcheck | `/healthz` on `PORT=8080`, timeout 120 s |
| Public domain | yes, target port 8080 |
| Volume | `/home` (home of `fedora` is `/home/fedora`) |
| Restart policy | ON_FAILURE, 10 retries |
| Replicas | 1 |

### Variables

| Variable | Default | Notes |
|---|---|---|
| `PASSWORD` | `${{secret(32)}}` | 12-256 bytes, one line. Otherwise the login is locked with a random password; the deploy still succeeds and the log says how to fix it. |
| `USERNAME` | `fedora` | Invalid names fall back to `fedora` with a warning. |
| `PORT` | `8080` | Invalid values or 7681 fall back to 8080 with a warning. |
| `TERMINAL_TMUX` | `true` | `false`: plain login shell per tab. |
| `TERMINAL_FONT_SIZE` | `15` | 8-40; anything else falls back to 15 with a warning. |
| `FEDORA_EXTRA_PACKAGES` | empty | Build argument. Invalid names and packages dnf cannot install are skipped with a warning (build log and deploy log); never fails the build. |
| `TZ` | `UTC` | Shell time zone. |
| `RAILWAY_DEPLOYMENT_DRAINING_SECONDS` | `15` | Grace period for the clean tmux shutdown. |

No required inputs.

## Improvements over market templates

1. A browser terminal at all: the market template only keeps a bare container alive for `railway ssh`.
2. Generated `secret(32)` password, nginx basic auth with a SHA-512 crypt hash, rate limit, cross-site WebSocket check, loopback-only ttyd. Fails closed without failing the deploy.
3. The whole `/home` persists (dotfiles, history, venvs, pipx, `npm -g`). Market: `/data` only.
4. tmux sessions survive closed tabs; `?arg=NAME` opens separate sessions.
5. A non-root user with sudo, a C/C++ toolchain and the usual CLI tools.
6. Pinned base image (`45` + digest) instead of `fedora:latest`; `dnf upgrade` at build; Node.js 24 LTS with checksum.
7. `/healthz` healthcheck with `PORT` set; build-time extra packages that never fail the build.

## Verification checklist

Done on the pre-publish test deploy (2026-11-20):

- [x] Build + deploy SUCCESS first try; `/healthz` 200.
- [x] `/` 401 without/with a wrong password, 200 with `PASSWORD`; WebSocket 101, 403 with a foreign Origin, 401 with a wrong password.
- [x] Shell: `whoami` = `fedora`, `sudo -n true`, Fedora Linux 45, node v24.22.3, npm 11.19.1, Python 3.15.2, GCC 16.2.1, git 2.56.1, tmux 3.8, `sudo ping`, git prompt, `TERM=tmux-256color`, home 0750, `PASSWORD` not in the environment.
- [x] `python3 -m venv` + `pip install requests`, `npm install -g cowsay`.
- [x] `PASSWORD=abc`: SUCCESS, lock warning, every password 401.
- [x] 0.5 GB limit: 0.022 GB idle.
- [ ] Persistence after redeploy: the check after restoring the password did not finish (workspace billing pause); same entrypoint/volume code as 127 and 128, where it passed.
- [ ] Publish + post-publish test (blocked: workspace billing).

## Open questions

- Fedora 45 reaches end of life about 13 months after its release; the template then needs a base bump to Fedora 46 or 47 (and a listing rename).
