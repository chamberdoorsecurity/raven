# Raven

Pentest engagement framework. Runs your tools, keeps everything in one engagement database, writes the report.

```bash
docker run --rm -p 127.0.0.1:8080:8080 ghcr.io/chamberdoorsecurity/raven:latest
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
docker run --rm -p 127.0.0.1:8080:8080 ghcr.io/chamberdoorsecurity/raven:latest
```

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

## Source

Closed source. This repo is docs and issues. The Docker image ships a compiled binary, not Python source.

## Links

- Docs: <https://chamberdoorsecurity.com/docs/>
- Changelog: <https://chamberdoorsecurity.com/changelog>
- Bugs: [issues](https://github.com/chamberdoorsecurity/raven/issues)

Authorized testing only. You need written permission for anything you point this at.

(c) Chamber Door Security, LLC
