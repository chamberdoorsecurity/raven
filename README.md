# Raven

Pentest engagement framework. Runs your tools, keeps everything in one engagement database, writes the report.

```bash
docker run --rm -p 127.0.0.1:8080:8080 \
  --cap-add=NET_RAW --cap-add=NET_ADMIN \
  ghcr.io/chamberdoorsecurity/raven:latest
```

Then <http://localhost:8080>. No account needed.

## What it does

- Runs nmap, nuclei, netexec, httpx, ffuf, sqlmap, gowitness, masscan. Arbitrary binaries are never blocked; wrap your own in a YAML module.
- Parses output into a per-engagement SQLite DB: hosts, ports, services, credentials, evidence, findings.
- Dedupes findings across hosts. Evidence carries SHA-256, custodian and acquisition time.
- Enforces scope and testing windows on every launch.
- Generates Markdown, HTML and PDF reports from the DB. No AI in that path.
- Image is Kali with the toolchain already installed.

## Install

Docker, any platform:

```bash
docker run --rm -p 127.0.0.1:8080:8080 \
  --cap-add=NET_RAW --cap-add=NET_ADMIN \
  ghcr.io/chamberdoorsecurity/raven:latest
```

Both caps are needed to run nmap at all: its binary carries file capabilities (`cap_net_admin`, `cap_net_raw`), and the kernel refuses to exec it unless they are in the container's bounding set. Docker grants `NET_RAW` by default but not `NET_ADMIN`.

On Linux, `--network host` instead of `-p` lets it scan your local network directly. That also exposes the dashboard on your interfaces, and there is no auth in single-user mode, so on a client site keep the loopback publish.

apt (Kali, Ubuntu 24.04; needs glibc 2.39+):

```bash
curl -fsSL https://chamberdoorsecurity.com/apt.sh | sudo sh
```

Adds the repo and signing key, prints the install command, installs nothing.

Binaries for linux amd64/arm64 and macOS arm64: <https://chamberdoorsecurity.com#download>. No Intel Mac build, use Docker.

## AI

Everything above works unpaired. AI features (writeups, attack-path suggestions, chat) need a paired account and draw credits.

## Verifying downloads

Each artifact has a published SHA-256; the download page prints the check command. The apt repo is GPG-signed.

Pin a digest instead of a tag:

```bash
docker image inspect ghcr.io/chamberdoorsecurity/raven:latest --format '{{index .RepoDigests 0}}'
```

## What is in this repo

- `workflows/` — the 17 shipped workflow templates (OWASP WSTG, PTES, external perimeter, web app, OSINT, and others). Fork one, edit the steps, drop it in your engagement's `workflows/` dir.
- `modules/examples/` — example YAML modules showing the format for wrapping your own tools.

Raven itself is closed source. The Docker image ships a compiled binary, not Python source, so GitHub's auto-generated "Source code" archives on each release contain these templates, not the product.

## License

The contents of this repository, meaning the workflow templates, the example
modules and this README, are MIT licensed. Fork them, edit them, ship them in
your own engagements, publish your own. That is what they are here for.

The MIT grant covers this repository and nothing else. Raven itself, the
binaries and packages attached to releases here, and the container images are
proprietary and are not licensed by it. Their terms are at
<https://chamberdoorsecurity.com>.

## Links

- Docs: <https://chamberdoorsecurity.com/docs/>
- Changelog: <https://chamberdoorsecurity.com/changelog>
- Bugs: [issues](https://github.com/chamberdoorsecurity/raven/issues)

Authorized testing only. You need written permission for anything you point this at.

(c) Chamber Door Security, LLC
