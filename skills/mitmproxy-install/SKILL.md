---
name: mitmproxy-install
description: Install, isolate, verify, upgrade, or repair mitmproxy environments across macOS, Linux, Windows, WSL, Docker, uv tool installs, project-local .venv environments, and mitmproxy source checkouts. Use when a user asks how to install mitmproxy, keep it scoped to one project, add Python addon dependencies, choose between native/PyPI/container setup, diagnose PATH or interpreter mismatches, or prepare a contributor development environment. Do not own proxy-mode/network interception setup after installation.
---

# mitmproxy installation and environment setup

Use this skill to get the **right mitmproxy environment** installed and verified with the smallest appropriate scope.

Do not jump directly to an install command. First determine what is being isolated: the application, a reusable CLI tool, one Python project, an upstream source checkout, or an entire container.

## Workflow

1. Inspect what already exists.
2. Classify the requested environment profile.
3. Detect the operating system and relevant environment manager when tools permit.
4. Choose the default supported path for that profile.
5. Install or repair without unnecessarily changing unrelated environments.
6. Verify executable path, version, and environment ownership.
7. Report what was installed and hand off any capture/network configuration.

Do not ask for OS, shell, Python path, or existing mitmproxy version when these can be detected safely.

## 1. Inspect before changing

Check for an existing installation and its provenance.

On macOS/Linux/WSL:

```sh
command -v mitmproxy || true
command -v mitmdump || true
command -v mitmweb || true
mitmproxy --version 2>/dev/null || true
command -v uv || true
```

On Windows PowerShell:

```powershell
Get-Command mitmproxy -ErrorAction SilentlyContinue
Get-Command mitmdump -ErrorAction SilentlyContinue
Get-Command mitmweb -ErrorAction SilentlyContinue
mitmproxy --version
Get-Command uv -ErrorAction SilentlyContinue
```

If the user is inside a Python project, also inspect `pyproject.toml`, lockfiles, and any existing `.venv` before prescribing a new environment.

If multiple installations exist, resolve which executable wins on `PATH` before upgrading or reinstalling.

## 2. Choose the environment profile

Use these defaults:

| Intent | Default profile |
| --- | --- |
| "Install mitmproxy on this machine" | native end-user install |
| "I want the commands globally but isolated from system Python" | `uv tool` |
| "This project/addon needs mitmproxy as a dependency" | project-scoped environment |
| "I want to contribute to mitmproxy itself" | source-development checkout |
| "Keep everything disposable/contained" | Docker |
| "I'm on Windows but working inside WSL" | Linux path inside WSL |

Read [references/platform-installation.md](references/platform-installation.md) when selecting native OS-specific commands or Docker/WSL behavior.

Read [references/environment-profiles.md](references/environment-profiles.md) when the user mentions `.venv`, Python dependencies, addons requiring packages, `uv`, `pip`, a source checkout, or environment conflicts.

## 3. Native end-user installation

Prefer the current upstream-supported platform route instead of a generic Python package install:

- macOS: Homebrew cask is the default.
- Linux: upstream standalone binaries are the default.
- Windows: upstream installer is the default; Microsoft Store is a supported alternative when automatic updates are desired.
- WSL: treat it as Linux inside the selected distribution.

Do not silently choose a Linux distro package merely because one exists. Upstream notes that downstream packages are community-maintained and may lag.

## 4. Isolated user-level tool

When the user wants `mitmproxy`, `mitmdump`, and `mitmweb` available as commands without coupling them to system Python, prefer:

```sh
uv tool install mitmproxy
```

When an addon needs an extra Python package in the same isolated tool environment, follow the upstream-supported `uv tool install --with <package> mitmproxy` pattern.

After changes, verify both the version and command location.

## 5. Project-scoped environment

Use this when mitmproxy belongs to one project, addon workspace, test harness, or automation repository.

First preserve the project's existing dependency manager. If it is already an `uv` project, prefer declaring mitmproxy as a project dependency rather than creating a second unmanaged installation.

If the project intentionally uses a local virtual environment without dependency metadata, a `.venv` is an appropriate boundary. Keep installation inside it and verify that the active/interpreted Python and mitmproxy executable come from that environment.

Explicitly tell the user that upstream recommends `uv tool` for PyPI-based CLI installation; ordinary virtualenv/pip approaches are compatible Python workflows but are not the project's primary supported installation path.

Never use `sudo pip install` or otherwise mutate system Python to solve a project-local requirement.

## 6. Source-development checkout

For contributing to `mitmproxy/mitmproxy`, use the repository's own development model rather than installing a released package into the checkout.

The current contributor workflow is based on `uv`; after cloning, `uv run mitmproxy --version` should bootstrap/use the checkout environment.

This skill owns environment preparation and verification only. Hand final upstream tests, lint, coverage, issue/PR preparation, and maintainer coordination to `mitmproxy-contribute`.

## 7. Verify

A successful setup is not "the install command exited zero." Verify:

- `mitmproxy --version` runs;
- `mitmdump --version` runs when expected;
- `mitmweb --version` runs when expected;
- the resolved executable path matches the intended profile;
- project-scoped installs resolve inside the intended environment;
- required addon Python dependencies import from the same environment;
- no unintended older installation is shadowing the new one.

For a `.venv`, verify using the environment's interpreter/executable directly when shell activation is ambiguous.

## 8. Handoffs

After installation:

- proxy mode selection, certificate/trust setup for a target, and proving traffic reaches mitmproxy -> `mitmproxy-capture`;
- low-level local redirect, WireGuard, TUN/eBPF, or platform capture internals -> `mitmproxy-networking`;
- addon implementation -> `mitmproxy-addons`;
- upstream patch quality gates and PR preparation -> `mitmproxy-contribute`.

Do not absorb those workflows into this skill merely because they happen immediately after installation.

## Gotchas

- "Global" can mean globally discoverable commands without system-Python mutation. `uv tool` is often the right answer.
- A `.venv` is a project boundary, not a machine-wide CLI installation.
- Installing mitmproxy does not automatically make HTTPS interception trusted on another device.
- Capture modes that need routing/elevated privileges are separate from installing the binaries.
- Multiple mitmproxy installs can make an upgrade appear ineffective because the shell still resolves an older executable.
- Windows host Python and WSL Python are separate environments; do not mix their paths or packages.
- Linux distro packages may differ from the current upstream release.
