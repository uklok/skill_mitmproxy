# Platform installation reference

Read this file when the task is a native OS install, WSL install, or containerized install.

Source of truth: <https://docs.mitmproxy.org/stable/overview/installation/>

Re-check the upstream page before changing commands in this reference.

## macOS

Upstream's recommended native installation is:

```sh
brew install --cask mitmproxy
```

Standalone binaries are also available from the mitmproxy website.

Verification:

```sh
command -v mitmproxy
mitmproxy --version
mitmdump --version
mitmweb --version
```

If Homebrew is already managing mitmproxy, preserve that ownership for upgrades rather than layering another installation method on top.

## Linux

Upstream recommends its standalone binaries for normal Linux installation.

Some distributions package mitmproxy themselves, but upstream explicitly describes these as community-supported downstream packages that may lag current releases. Use a distro package only when the user or environment has a reason to prefer distribution lifecycle integration over upstream freshness.

Keep "install the application" separate from transparent-proxy routing. iptables/nftables/sysctl or other routing changes belong to capture/networking workflows, not this installation procedure.

## Windows

Upstream supports:

1. the mitmproxy Windows installer;
2. Microsoft Store installation, which can update automatically;
3. standalone binaries.

The installer/Store setup places `mitmproxy`, `mitmdump`, and `mitmweb` on PATH.

Upstream also recommends Windows Terminal for improved console rendering; this is optional UX guidance, not a mitmproxy runtime requirement.

PowerShell verification:

```powershell
Get-Command mitmproxy
mitmproxy --version
mitmdump --version
mitmweb --version
```

Do not assume that installing on the Windows host also installs mitmproxy in WSL.

## WSL

Treat each WSL distribution as a Linux environment.

Decide whether the user intends to run mitmproxy:

- as a Windows application on the host, or
- inside WSL.

Install and resolve paths inside the environment that will actually execute the proxy. Avoid mixing Windows Python packages, WSL Python packages, and their respective virtual environments.

## Docker

Official mitmproxy images are available from Docker Hub.

Use Docker when the user's priority is a disposable/reproducible runtime, CI execution, or minimizing host dependencies. Before presenting a final command, determine:

- which proxy/web ports must be published;
- whether config/certificate state should persist;
- whether flow files or scripts need bind mounts;
- whether the container must reach host-only services;
- whether the selected capture topology can work across the container network boundary.

Do not present a container as equivalent to a native transparent/local capture setup without verifying the networking model.

## Binary package maintenance

Upstream notes that precompiled binaries and Docker images contain frozen dependency sets. They do not self-update. Encourage normal release updates rather than assuming bundled dependencies are refreshed in place.
