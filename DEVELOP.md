# Developer Guide

This project uses **[uv](https://docs.astral.sh/uv/)** for all Python and dependency
management. If you haven't used uv before, the key docs are:

- [Installing uv](https://docs.astral.sh/uv/getting-started/installation/)
- [Workspaces](https://docs.astral.sh/uv/concepts/workspaces/)
- [Dependency groups](https://docs.astral.sh/uv/concepts/dependencies/#dependency-groups)
- [Build backend](https://docs.astral.sh/uv/concepts/build-backend/)

---

## Initial setup

```sh
make setup        # runs uv sync + pre-commit install
```

This creates `.venv/`, installs all workspace members, and sets up the git hooks.

## Adding a new workspace package

uv workspaces let you split code into independently-versioned packages under one repo.
Our convention: library code goes in `packages/`, runnable apps in `apps/`.

### 1. Create the package

```sh
# From the repo root:
uv init --package packages/my_new_package \
  --build-backend uv
```

This creates `packages/my_new_package/` with:
- `pyproject.toml` (using `uv_build` backend — no setuptools/hatch needed)
- `src/my_new_package/__init__.py`

The `--build-backend uv` flag tells uv to use its own
[build backend](https://docs.astral.sh/uv/concepts/build-backend/) (`uv_build`), which is
lighter than setuptools or hatch and handles the `src/` layout automatically.

### 2. Verify it's a workspace member

Check the root `pyproject.toml`:

```toml
[tool.uv.workspace]
members = ["packages/*", "apps/*"]
```

The glob `packages/*` picks up the new package automatically. If you used a nested path
like `packages/sub/my_pkg`, you'd need `packages/**` or an explicit entry.

### 3. Add dependencies to the new package

```sh
# Runtime dep (goes into the package's own pyproject.toml):
uv add --package my_new_package pandas

# If you need another workspace member as a dep:
uv add --package my_new_package my_other_package
```

uv resolves workspace-internal references via
[`tool.uv.sources`](https://docs.astral.sh/uv/concepts/workspaces/#configuring-workspaces):

```toml
[tool.uv.sources]
my_other_package = { workspace = true }
```

### 4. Sync

```sh
uv sync --all-packages --all-extras --all-groups
# or just:
make sync
```

## Dependency groups

We use [dependency groups](https://docs.astral.sh/uv/concepts/dependencies/#dependency-groups)
(PEP 735) to keep deps organized without polluting the main install:

| Group | Purpose | When installed |
|---|---|---|
| `dev` | pytest, mypy, ruff, pre-commit | always (`default-groups`) |
| `notebooks` | ipykernel, pandas, plotly, matplotlib, etc. | `uv sync --group notebooks` or `--all-groups` |

### Adding a dep to a group

```sh
# Add to the notebooks group (defined in root pyproject.toml):
uv add --group notebooks folium

# Add to dev:
uv add --group dev hypothesis
```

Groups live in the **root** `pyproject.toml` under `[dependency-groups]`.
They're installed into the shared `.venv/` — not per-package.

## Running things

All commands go through `uv run` to ensure the right venv is active:

```sh
uv run pytest                  # tests
uv run ruff check . --fix      # lint
uv run mypy                    # type check
uv run python -m my_package    # run a module
```

Or use the Makefile shortcuts:

```sh
make test       # uv run pytest -q
make lint       # uv run ruff check . --fix
make typecheck  # uv run mypy
make check      # format + lint + typecheck + test
```

## Adding a CLI entry point

In a package's `pyproject.toml`:

```toml
[project.scripts]
my-cli = "my_package.cli:main"
```

After `uv sync`, the command `my-cli` is available via `uv run my-cli` or directly in the
activated venv.

## Notebooks

Notebooks go in `notebooks/`. The `notebooks` dependency group provides the kernel and
common data-science packages. To use it:

```sh
uv sync --group notebooks
```

Then select the `.venv/bin/python` kernel in VS Code / Jupyter.

Notebooks are **not** type-checked or linted by CI — they're exploratory. Keep reusable
logic in `packages/` and import it from notebooks.

## Pre-commit & CI

Pre-commit runs ruff (lint + format) and basic file checks on every commit.
CI (`.github/workflows/ci.yml`) runs the full `make sync` → lint → typecheck → test
pipeline on push/PR.

## Project conventions

- **src layout** for all packages (`packages/foo/src/foo/`)
- **Line length**: 120 (ruff + pylint + editors all configured)
- **Type checking**: mypy strict mode — add `# type: ignore[code]` with a comment when needed
- **Quotes**: double (`"`) — enforced by ruff format
- **Tests**: mirror the package structure under `tests/` at the repo root, or `packages/foo/tests/`

## Creating a GitHub template repo from this

If you want to reuse this scaffold as a GitHub template:

1. Push this repo to GitHub (or create a new repo from the scaffold output)
2. Go to **Settings → General → Template repository** and check the box
3. Now anyone (including you) can click **"Use this template"** on GitHub to create a new
   repo with the same file structure
4. After cloning the new repo, run:
   ```sh
   # Update project name in pyproject.toml, CITATION.cff, *.code-workspace, README
   # Then:
   make setup
   ```

Alternatively, keep `scaffold.sh` in a gist or standalone repo and run it directly:

```sh
curl -fsSL https://raw.githubusercontent.com/YOU/ds-template/main/scaffold.sh | bash -s my-project
```
