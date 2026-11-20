# Agent instructions

This repository contains Agent Skills for the mitmproxy ecosystem.

## Development boundary

- The bootstrap README is the only intentional direct-to-`main` commit. Make all normal changes on a feature branch and land them through a pull request.
- Keep a change coherent: one skill or one cross-cutting architecture concern per PR unless the pieces cannot be reviewed independently.
- Do not copy large blocks of upstream documentation into this repository. Prefer concise procedures plus links/source references.
- Do not treat archived mitmproxy repositories as current implementation authority without explicit verification.

## Skill format

Follow the current Agent Skills specification:

- each skill lives at `skills/<skill-name>/SKILL.md`;
- the frontmatter `name` must match the parent directory;
- names use lowercase letters, numbers, and hyphens only;
- descriptions must explain both what the skill does and when it should trigger;
- keep `SKILL.md` focused and under 500 lines;
- put conditional/deep material in `references/` and tell the agent exactly when to read it;
- keep references one level deep from `SKILL.md`;
- add scripts only when repeated deterministic logic has emerged from real execution.

## Research hierarchy

For implementation behavior, prefer in order:

1. current source and repository-local instructions in the relevant mitmproxy repository;
2. current official mitmproxy documentation;
3. current Agent Skills specification/best practices for skill packaging;
4. community discussions and historical sources as contextual evidence only.

State when guidance is inferred rather than directly supported upstream.

## Skill boundaries

The planned responsibility map is maintained in `README.md`. Avoid duplicating sibling ownership.

When editing a skill:

1. identify the user intent it owns;
2. identify neighboring intents that must *not* trigger it;
3. keep handoffs explicit;
4. add/update positive, negative, and ambiguous trigger cases;
5. include a success path, failure/recovery behavior, and a verification step.

## mitmproxy-specific contribution facts

For work against `mitmproxy/mitmproxy`:

- use `uv` as required by its repository guidance;
- run Python tests through the repository's documented `uv run ...` commands;
- account for lint and coverage requirements;
- accompany patches with appropriate tests;
- discuss sufficiently large upstream changes with maintainers before investing heavily.

Always re-check upstream instructions before relying on these summaries; they can change.

## Installation guidance

Installation guidance must distinguish:

- native end-user installation;
- isolated user-level tool installation;
- project-scoped Python environments such as `.venv`;
- mitmproxy source-development checkouts;
- containers;
- WSL.

Separate OS-neutral decision logic from macOS/Linux/Windows specifics. Preserve an existing project environment manager when one already exists. Never mutate system Python merely for convenience.

## Security and privacy

mitmproxy handles intercepted traffic. Treat generated CA material, flow files, cookies, authorization headers, credentials, request/response bodies, and user traffic as sensitive.

Examples should use traffic and systems the user is authorized to inspect. Do not make elevated privileges the default when a non-privileged capture setup satisfies the task.
