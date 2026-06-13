# kit-template

A single [Copier](https://copier.readthedocs.io/) template for **both** project archetypes —
Python-only services/libraries **and** reflex.dev / AppKit web apps — chosen by a
`project_type` question. Replaces the separate `python-kit` / `projectkit` GitHub templates
with one **updateable** template (`copier update`).

```bash
uv tool install copier
copier copy --trust gh:jenreh/kit-template my-project
cd my-project && task init
```

`--trust` is required (post-gen tasks run `uv sync` and wire `.claude/skills`).

## Questions

| Question | Default | Notes |
|---|---|---|
| `project_name` | — | kebab-case distribution name |
| `project_type` | `python` | `python` (CLI/lib/service) or `reflex` (reflex.dev/AppKit web) |
| `package_name` | from `project_name` | python only; snake_case (reflex always uses `app/`) |
| `project_description` | auto | |
| `author_name` / `github_owner` | jenreh | |
| `python_version` | 3.14 | sets `requires-python` + ruff target |
| `include_db` | false | adds SQL deps **+ Alembic** (pure `env.py`). reflex always includes the DB layer (appkit `env.py`) |
| `db_name` | `{{ project_name }}-db` | asked for reflex / python+db |
| `use_devcontainer` | true | any project type (db service included only when there's a DB) |
| `include_docker` | reflex→true | Dockerfile+compose (reflex) / docker task |
| `include_terraform` | false | `terraform/` + azure task |
| `include_docs` | python→true | VitePress `docs/` scaffold |
| `include_graph` | false | GraphDB / runic hooks (see note) |
| `include_skills` | true | vendor `jenreh/agent-skills` into `.agents/skills` |

## How it works

- **Shared base** is always generated (pyproject, ruff/mypy/pytest, pre-commit, gitleaks,
  AGENTS/CLAUDE, terraform, `qa`/`release` tasks, `.agents/skills`).
- **`project_type: reflex`** adds the reflex layer: `app/`, `alembic/`, `rxconfig.py`,
  `docker-compose.yml`, `configuration/`, `assets/`, `components/`, `.devcontainer/`,
  `Dockerfile`, reflex deps, `Taskfile.reflex.yml`.
- **Feature flags** are wired two ways:
  - **Files** via Copier's templated `_exclude` (a pattern renders to `""` — matches
    nothing — when its feature is on).
  - **Taskfile includes** are all marked `optional: true` in `Taskfile.dist.yml`, so go-task
    silently skips any `tasks/Taskfile.*.yml` the template didn't ship. You only get the task
    namespaces (`db:`, `docker:`, `dev:`/`prod:`, `reflex:`) for components you selected.
- The python package lives in a whole-segment conditional directory that vanishes for reflex;
  reflex's `app/` is excluded for python.

## Notes

- `Taskfile*.yml` are **not** Jinja-rendered (go-task `{{.VAR}}` would collide); `PROJECT` is
  derived from `pyproject.toml` at runtime.
- **`include_graph`**: adds `runic-py` (graph schema migrations + OGM for Cypher DBs) to
  dependencies and vendors the runic skills. (Note: `include_graph` relaxes the `typer` pin to
  `>=0.26.0`, which `runic-py` requires.)
- **Alembic** works for both archetypes: `alembic/env.py` is rendered as the appkit-coupled
  variant for reflex, or a pure SQLAlchemy variant (reads `$DATABASE_URL` / `alembic.ini`) for
  Python — so `task db:*` is usable in both.
- `task test` for **reflex** needs Postgres + `.env` secrets (the app boots against a DB).

## Maintainers

Refresh the bundled skills snapshot with `scripts/sync-skills.sh`, then commit + tag a new
version (generated projects pull it via `copier update`).
