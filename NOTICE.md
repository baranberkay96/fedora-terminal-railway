# NOTICE: Fedora 45 Web Terminal template

## Software installed in the image

| Component | Source | License | How it is used |
|---|---|---|---|
| Fedora 45 | `fedora` official image, https://hub.docker.com/_/fedora, tag `45`, index digest `sha256:a7d72ba2…` (2026-11-02), upgraded with `dnf upgrade` at build | Various free licenses (see `rpm -qi` and `/usr/share/licenses` in the image) | Base image (`FROM`) and dnf packages |
| ttyd 1.7.7 | Fedora package `ttyd` | MIT, Copyright (c) 2016-2022 Shuanglei Tao | Web terminal on loopback, installed unmodified |
| nginx 1.30 | Fedora package `nginx` | BSD-2-Clause | Basic-auth reverse proxy. The config was written for this template |
| tmux 3.8 | Fedora package `tmux` | ISC | Persistent sessions |
| tini 0.19.0 | Fedora package `tini` | MIT | Init and child subreaper |
| Node.js 24.22.3 LTS | Official tarballs from https://nodejs.org/dist/v24.22.3/, SHA-256 checked | MIT (bundled components under their own licenses, see `/usr/local/share/doc/nodejs/LICENSE`) | Installed unmodified into `/usr/local` |
| GCC, git, Python 3, ripgrep, fd, fastfetch and the other CLI tools | Fedora packages | Each package's own license (GPL, MIT, BSD, Apache and similar) | Installed unmodified from the Fedora repositories |

Changes to installed Fedora files: file capabilities are removed from all binaries (Railway's containers cannot exec binaries whose capabilities they lack; this lets `sudo ping` work), the account `fedora` (uid 1000, bash) is added, `/etc/sudoers.d/90-fedora-terminal` grants it passwordless sudo, and `/etc/skel/.bashrc` gets one line that loads this template's shell defaults. nginx's own `nginx.conf` is replaced at every start by the file this template renders. No package files are edited otherwise.

Optional packages named in `FEDORA_EXTRA_PACKAGES` are installed from the Fedora repositories at build time under their own licenses.

Product names belong to their owners. This is a community template, not affiliated with the Fedora Project, Red Hat, the ttyd author, the OpenJS Foundation or Railway. Fedora is a trademark of Red Hat, Inc.

## Upstream files and data checked (2026-11-20)

- Docker Hub tags of `library/fedora`: `45` = `latest` (index `sha256:a7d72ba2745b100a3351087472e5ee7b2450af7cf331e7673007e3817612839c`, amd64 manifest `sha256:53d138f2…`, 72 MB compressed); `46` is Rawhide.
- Fedora 45 package metadata (mdapi.fedoraproject.org): every package name above, and the paths `/usr/bin/tini`, `/usr/bin/ttyd`, `/usr/bin/fd`, `/usr/share/git-core/contrib/completion/git-prompt.sh`.
- https://nodejs.org/dist/v24.22.3/SHASUMS256.txt (24.22.3 is the newest 24.x "Krypton" LTS).
- The ttyd behaviour this template relies on (auth-header mode `-H`, loopback `-i lo`, the `-c` startup log line that prints the credential) was checked for the Debian Web Terminal template of the same author against ttyd tag `1.7.7`; Fedora ships the same release.

## Market templates reviewed

"Fedora" (bare `fedora:latest` with `tail -f /dev/null` and a `/data` volume) and two Fedora webtop desktops. Only their configuration and listing figures were used. No files or text from other templates were used.

All files in this package were written for this template, adapted from the same author's Ubuntu, Debian and Kali Linux Web Terminal templates: the Dockerfile, `entrypoint.sh`, `terminal-session.sh`, `bashrc.sh`, `tmux.conf`, `nginx.conf.template`, `motd.txt` and the docs.
