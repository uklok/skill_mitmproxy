# Python and development environment profiles

Read this file for `.venv`, `uv`, PyPI, addon dependencies, source checkouts, or conflicting Python installations.

Primary references:

- <https://docs.mitmproxy.org/stable/overview/installation/>
- <https://github.com/mitmproxy/mitmproxy/blob/main/CONTRIBUTING.md>
- <https://github.com/mitmproxy/mitmproxy/blob/main/AGENTS.md>

## Profile A: isolated user-level CLI

Upstream recommends `uv` when installing mitmproxy from PyPI:

```sh
uv tool install mitmproxy
```

This creates an isolated tool environment while exposing the command-line tools to the user.

For an addon that requires another Python package, upstream documents:

```sh
uv tool install --with <package-name> mitmproxy
```

Use this profile when the user wants an application/tool rather than a dependency of one specific repository.

## Profile B: project-scoped dependency

Use a project-local environment when code in one repository imports mitmproxy, pins its version, or needs a reproducible dependency set.

### Existing uv project

If `pyproject.toml` and `uv.lock` show that the project already uses uv, preserve that model. Add mitmproxy through the project's dependency declaration rather than installing it globally.

Typical workflow:

```sh
uv add mitmproxy
uv run mitmproxy --version
```

Do not run `uv add` blindly: inspect whether the project uses dependency groups or another convention for development-only tooling.

### Local .venv without project metadata

When the repository intentionally uses a simple local virtual environment, create/reuse `.venv` and keep mitmproxy inside it.

POSIX with uv:

```sh
uv venv .venv
uv pip install --python .venv/bin/python mitmproxy
.venv/bin/mitmproxy --version
```

Windows PowerShell with uv:

```powershell
uv venv .venv
uv pip install --python .venv\Scripts\python.exe mitmproxy
.venv\Scripts\mitmproxy.exe --version
```

If the project already uses `venv` + `pip`, preserve that convention instead of migrating solely for stylistic consistency.

Upstream's installation docs state that common Python approaches such as pip/virtualenv should generally work, but the project does not have capacity to support all of them. Present this profile as a compatible project-isolation technique, not as the preferred native mitmproxy installation.

## Profile C: upstream mitmproxy development

For contributions to `mitmproxy/mitmproxy`:

```sh
git clone https://github.com/mitmproxy/mitmproxy.git
cd mitmproxy
uv run mitmproxy --version
```

The checkout's dependency/development environment is managed through uv.

Current repository guidance includes:

```sh
uv run tox
uv run tox -e lint
```

For focused pytest work, use the repository-prescribed `uv run pytest ...` form. New source files may also require the repository's individual-coverage tox environment.

Do not replace this with a separately installed PyPI version: doing so risks testing the released package rather than the checkout.

## Diagnosing path/interpreter mismatches

When behavior does not match the just-installed version, inspect provenance.

POSIX:

```sh
command -v -a mitmproxy 2>/dev/null || which -a mitmproxy
mitmproxy --version
python -c 'import sys; print(sys.executable)'
```

PowerShell:

```powershell
Get-Command mitmproxy -All
mitmproxy --version
python -c "import sys; print(sys.executable)"
```

For Python-package workflows, also inspect where the imported package resolves:

```sh
python -c "import mitmproxy, pathlib; print(pathlib.Path(mitmproxy.__file__).resolve())"
```

Interpret these paths before reinstalling. Common causes are:

- a global executable preceding `.venv` on PATH;
- shell command hashing/cache after an install;
- Windows vs WSL environment confusion;
- an activated environment different from the project being modified;
- a released PyPI package shadowing an editable/source checkout.

## Environment boundary rule

Choose one ownership model per use case and make it observable:

- native package manager owns the native application;
- uv tool owns the isolated user CLI;
- project metadata/`.venv` owns project dependencies;
- repository checkout + uv owns upstream development;
- container image owns the containerized runtime.

Avoid stacking multiple owners unless the user is intentionally testing multiple versions.
