# bi-python-project-template

![CI](https://github.com/OWNER/bi-python-project-template/actions/workflows/ci.yml/badge.svg)

> _One-line project description._

## Quickstart

```sh
make setup    # create venv, install deps, set up pre-commit
make check    # format + lint + typecheck + test
```

## Project layout

```
packages/     # reusable library code (src layout, uv_build backend)
apps/         # runnable apps / CLI entry points
notebooks/    # Jupyter exploration
docs/         # methodology, metrics, assumptions, limitations
data/         # raw / interim / processed / external (gitignored)
outputs/      # figures, map layers, marts (gitignored)
```

## Key docs

- [docs/METHODS.md](docs/METHODS.md)
- [docs/METRIC_DEFINITIONS.md](docs/METRIC_DEFINITIONS.md)
- [docs/ASSUMPTIONS.md](docs/ASSUMPTIONS.md)
- [docs/LIMITATIONS.md](docs/LIMITATIONS.md)
- [docs/DATA_SOURCES.md](docs/DATA_SOURCES.md)

## About the scaffolding

This project structure comes from a personal starter template. It's set up following the
actual tooling docs and adjusted to how I work:

- [uv workspaces](https://docs.astral.sh/uv/concepts/workspaces/) for the monorepo layout
- [uv dependency groups](https://docs.astral.sh/uv/concepts/dependencies/#dependency-groups)
  to separate dev / notebook / production deps
- [uv build backend](https://docs.astral.sh/uv/concepts/build-backend/) (`uv_build`)
  so packages are installable without setuptools
- [ruff](https://docs.astral.sh/ruff/) for linting + formatting in one tool
- [mypy strict](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-strict)
  for type safety

See [DEVELOP.md](DEVELOP.md) for how to add packages, manage dependencies, and work with uv.

## License

GPLv3 (see LICENSE).
