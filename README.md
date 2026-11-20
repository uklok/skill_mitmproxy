# mitmproxy Agent Skills

Community-oriented [Agent Skills](https://agentskills.io/) for learning, operating, extending, debugging, and contributing to the [mitmproxy](https://github.com/mitmproxy) ecosystem.

This repository is intentionally organized around **distinct user intents and engineering responsibilities**, not around a single monolithic "mitmproxy" skill. The goal is high-quality skill discovery: an agent should load the smallest relevant body of guidance, know where its responsibility ends, and hand off cleanly to a sibling skill when the task crosses a boundary.

> **Status:** bootstrap / architecture phase. The repository is being built incrementally through reviewed pull requests. Skill content is not considered stable until its trigger behavior and workflows have been evaluated.

## Why this exists

mitmproxy is more than one executable and more than one repository. A user may be trying to:

- install mitmproxy on a particular operating system;
- run it in an isolated Python environment for one project;
- choose a capture mode and get traffic flowing;
- inspect, filter, record, replay, or modify flows;
- build a Python addon;
- diagnose protocol or TLS behavior;
- work on platform-specific networking in Rust;
- change mitmweb;
- reproduce an issue and prepare an upstream contribution.

Those tasks overlap, but they do not have the same prerequisites, source of truth, validation steps, or failure modes. Treating them as one giant skill would reduce discovery quality and make maintenance harder.

## Design principles

1. **One discoverable responsibility per skill.** Trigger descriptions should distinguish neighboring workflows rather than competing for the same prompt.
2. **Progressive disclosure.** Keep each `SKILL.md` focused; move deep architecture, command matrices, and edge cases into referenced resources.
3. **Source before folklore.** Prefer current upstream documentation and repository evidence. Mark inferred guidance as such.
4. **Separate use from development.** Installing released mitmproxy, using it inside a project environment, and developing mitmproxy from source are different setup profiles.
5. **Separate OS-neutral and OS-specific steps.** Shared concepts belong once; macOS, Linux, Windows, and WSL differences should remain explicit.
6. **Validate before contributing.** Skills that change code must include the upstream project's tests, linting, coverage expectations, and contribution routing.
7. **Safe interception by default.** Examples should use systems and traffic the user is authorized to inspect. Certificates, flow dumps, credentials, cookies, and captured payloads should be treated as sensitive.
8. **Clean handoffs.** A skill should state when another skill owns the next step instead of duplicating its instructions.

## Planned skill map

| Skill | Primary responsibility | Typical handoffs |
| --- | --- | --- |
| `mitmproxy-install` | Select and execute the right installation/environment strategy for macOS, Linux, Windows, WSL, Docker, isolated tools, project-scoped Python environments, or source development | `capture`, `addons`, `contribute` |
| `mitmproxy-discover` | Locate the correct repository, subsystem, source path, docs, and ownership boundary before changing anything | any specialist skill |
| `mitmproxy-capture` | Choose and configure a proxy/capture topology and verify that intended traffic reaches mitmproxy | `flows`, `protocols`, `networking` |
| `mitmproxy-flows` | Inspect, filter, modify, save, replay, and transform captured flows | `addons`, `protocols` |
| `mitmproxy-addons` | Build and test Python addons, hooks, commands, options, and scripted transformations | `protocols`, `contribute` |
| `mitmproxy-protocols` | Investigate application/transport protocol behavior such as HTTP, HTTP/2, HTTP/3, WebSocket, DNS, TCP, UDP, TLS, and QUIC | `networking`, `contribute` |
| `mitmproxy-networking` | Work on Rust-backed capture and platform networking, including WireGuard, local redirect, TUN/eBPF, macOS, Windows, and Linux components | `protocols`, `contribute` |
| `mitmproxy-web` | Work on mitmweb frontend behavior and its integration with the shared mitmproxy core | `flows`, `contribute` |
| `mitmproxy-contribute` | Reproduce/triage issues, prepare tests and patches, run project quality gates, and shape upstream-ready PRs | specialist skill for implementation |

Names are intentionally responsibility-oriented. They may change if trigger evaluations show ambiguity.

## Installation and environment model

The `mitmproxy-install` skill will reason about **what the user is trying to isolate**, not just which command happens to install a package.

### 1. Native end-user installation

Use the upstream-recommended platform path when the user wants a normal machine-wide application installation:

- **macOS:** Homebrew cask is the upstream-recommended route; standalone binaries are also available.
- **Linux:** upstream recommends standalone binaries. Distribution packages may exist, but are community-maintained and can lag behind current releases.
- **Windows:** upstream provides an installer, Microsoft Store package, and standalone binaries. WSL follows the Linux path.

The skill should verify the resulting `mitmproxy`, `mitmdump`, and `mitmweb` executables and avoid silently substituting a distro package for an upstream-supported installation.

### 2. Isolated CLI/tool installation

When the goal is to keep mitmproxy out of the system Python while exposing its commands globally, upstream recommends `uv tool install mitmproxy`.

Addon dependencies can be attached to that isolated tool environment with `uv tool install --with <package> mitmproxy`.

This is different from a project-local virtual environment: the tool is isolated, but its commands remain available as user-level tools.

### 3. Project-scoped Python environment (`.venv`)

Use this profile when mitmproxy and related Python dependencies belong to one project, test harness, or addon workspace.

The skill should:

1. establish or reuse the project's environment manager;
2. create a local `.venv` when appropriate;
3. install/pin mitmproxy and addon dependencies inside that environment;
4. verify which Python and mitmproxy executables are active;
5. record reproducible dependency state using the project's chosen mechanism;
6. avoid mutating the system Python;
7. clearly distinguish this compatible Python workflow from the installation methods for which the mitmproxy project provides primary support.

For `uv`-managed projects, prefer an explicit project environment and reproducible dependency declaration over ad-hoc global `pip install` commands. If an existing project already uses another environment manager, preserve that project's convention unless there is a concrete reason to migrate it.

### 4. mitmproxy source-development environment

This is for contributors working on `mitmproxy/mitmproxy`, not for ordinary use.

The upstream repository uses `uv`. Its contributor workflow runs commands from the checkout with `uv run`, and project guidance requires tests/lint through the repository's established `tox` environments.

The install skill owns preparing and verifying the environment; `mitmproxy-contribute` owns contribution policy and final quality gates.

### 5. Containers

Official Docker images are an independent execution profile. The skill should use them when isolation, disposable environments, CI, or host dependency minimization is the user's primary constraint, while making network exposure, bind mounts, certificates, and persisted flow/config data explicit.

### Environment questions the skill should resolve

Before prescribing commands, determine as much as possible from context:

- operating system and architecture;
- normal user installation vs project dependency vs upstream development;
- whether commands should be globally available or local to a project;
- whether extra Python packages are required by addons;
- whether Docker/WSL is already the user's execution boundary;
- whether elevated privileges are actually required by the selected capture mode;
- version/pinning requirements;
- shell and PATH constraints;
- whether the environment already contains a compatible mitmproxy installation.

Do not ask the user for facts that can be detected safely from the working environment.

## Upstream architecture map

Primary active repositories currently include:

- [mitmproxy/mitmproxy](https://github.com/mitmproxy/mitmproxy) — Python core, proxy stack, addons, content views, I/O, tools, web application, examples, and tests.
- [mitmproxy/mitmproxy_rs](https://github.com/mitmproxy/mitmproxy_rs) — Rust networking and platform components, Python bindings, WireGuard/local redirect infrastructure, content views, and highlighting.
- [mitmproxy/pdoc](https://github.com/mitmproxy/pdoc) — API documentation tooling.
- [mitmproxy/www](https://github.com/mitmproxy/www) — project website infrastructure.
- [mitmproxy/android-unpinner](https://github.com/mitmproxy/android-unpinner) — specialized Android certificate-pinning research/tooling.

Archived repositories such as `netlib`, `pathod`, and the standalone `mitmweb` repository may be useful as historical evidence, but should not be treated as current implementation authority without verification.

## Repository shape

The target shape is:

```text
.
├── README.md
├── AGENTS.md
├── skills/
│   ├── mitmproxy-install/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── mitmproxy-discover/
│   │   └── SKILL.md
│   ├── mitmproxy-capture/
│   │   └── SKILL.md
│   ├── mitmproxy-flows/
│   │   └── SKILL.md
│   ├── mitmproxy-addons/
│   │   └── SKILL.md
│   ├── mitmproxy-protocols/
│   │   └── SKILL.md
│   ├── mitmproxy-networking/
│   │   └── SKILL.md
│   ├── mitmproxy-web/
│   │   └── SKILL.md
│   └── mitmproxy-contribute/
│       └── SKILL.md
├── references/
│   ├── architecture.md
│   ├── upstream-map.md
│   └── contribution-policy.md
└── evals/
    ├── trigger-cases.yaml
    └── workflow-cases.md
```

The final structure may be adjusted as evaluations reveal better ownership boundaries.

## Skill quality gates

Every skill should be pressure-tested before being called stable.

### Discovery

- positive prompts that should trigger the skill;
- negative prompts that belong to a sibling skill;
- ambiguous prompts that test routing and handoff behavior;
- terminology variants a user is likely to use without knowing mitmproxy's internal names.

### Workflow

At minimum:

- one successful workflow;
- one failure/recovery path;
- one sibling-skill handoff;
- verification that commands and referenced source paths are current.

### Contribution-facing skills

When a workflow modifies upstream source, follow the relevant repository's own contributor instructions. For the main mitmproxy repository, current guidance includes using `uv`, running tests through the project's established commands, running lint checks, adding matching tests, and discussing sufficiently large changes with maintainers before investing heavily in implementation.

## Contribution workflow for this repository

The initial README is the repository bootstrap commit. After it lands:

1. create a feature branch from `main`;
2. make one coherent architectural or skill change per branch;
3. include or update evaluations with behavior changes;
4. open a pull request against `main`;
5. review discovery overlap, source accuracy, and handoff behavior before merge.

Do **not** use direct-to-`main` development for normal work after bootstrap.

Tracking starts in [Issue #1](https://github.com/uklok/skill_mitmproxy/issues/1).

## Source-of-truth hierarchy

When guidance disagrees, prefer:

1. current source code and repository-local contributor instructions for implementation behavior;
2. current official mitmproxy documentation for supported user workflows;
3. current Agent Skills specification and creation guidance for skill structure/discovery;
4. community discussion and historical repositories as contextual evidence, clearly labeled.

Useful starting points:

- [mitmproxy documentation](https://docs.mitmproxy.org/stable/)
- [mitmproxy installation](https://docs.mitmproxy.org/stable/overview/installation/)
- [mitmproxy source](https://github.com/mitmproxy/mitmproxy)
- [mitmproxy contributor guide](https://github.com/mitmproxy/mitmproxy/blob/main/CONTRIBUTING.md)
- [mitmproxy repository agent guidance](https://github.com/mitmproxy/mitmproxy/blob/main/AGENTS.md)
- [mitmproxy_rs](https://github.com/mitmproxy/mitmproxy_rs)
- [Agent Skills specification](https://agentskills.io/specification)
- [Agent Skills best practices](https://agentskills.io/skill-creation/best-practices)
- [Anthropic skill creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)

## License

A repository license has not yet been selected. Do not assume that this repository's skill content inherits the license of any referenced upstream project. Source excerpts and bundled material must respect their original licenses.
