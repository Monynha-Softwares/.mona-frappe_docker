## Repo overview

This repository packages Frappe/ERPNext for development and production using Docker Compose. Major items:

- `compose.yaml` and `overrides/` — main service definitions and production/dev override files.
- `images/` — Dockerfile/Containerfile templates used to build custom images.
- `docs/` — guidance for running, building, and troubleshooting (start here: `docs/getting-started.md`).
- `tests/` — integration tests that exercise compose flows and bench commands.

AI agent quick goals: help contributors iterate on container setup, tests that use `Compose` helper, and docs.

## Key workflows and commands

- Dev sandbox: `docker compose -f pwd.yml up -d` (uses `pwd.yml` for a single-machine quick run).
- Local dev (compose): `docker compose -f pwd.yml up -d` or use `compose.yaml` + overrides listed in tests (`overrides/compose.proxy.yaml`, `compose.mariadb.yaml`, `compose.redis.yaml`).
- Build multi-arch images: `docker buildx bake --no-cache --set "*.platform=linux/arm64"` then add `platform: linux/arm64` to services in `pwd.yml`.
- Tests: `pytest` (see `requirements-test.txt`) — tests use `example.env` as a base; CI injects `CI` env var and `tests/compose.ci.yaml`.

Examples to copy/paste when scripting actions from the repo root:

```powershell
docker compose -f pwd.yml up -d
docker compose -f pwd.yml logs -f create-site
pytest -q
```

## Project-specific patterns and conventions

- Compose wrapper: tests use `tests.utils.Compose` which builds command tuples with `-f compose.yaml` plus several `overrides/*` files. When modifying compose files, make sure tests keep the same override ordering.
- Bench commands are executed inside the `backend` container using `compose.bench(...)` (see `tests/conftest.py`). When adding bench helpers, keep CLI usage consistent (e.g., `bench new-site`, `bench backup`).
- Environment file: `example.env` is copied and mutated in tests (`tests/conftest.py` writes SITES and version vars). Prefer reading/writing env vars via that file in tests and CI.
- Tests rely on writing files into containers using `compose cp` then running with `compose.exec`; follow that pattern for integration-style checks.

## Important files to inspect for context

- `compose.yaml`, `pwd.yml` — service layout and default ports
- `overrides/*.yaml` — common runtime variants (https, postgres, traefik, redis, mariadb)
- `images/bench/Dockerfile` (and siblings) — how images are built and what is included in runtime images
- `tests/conftest.py`, `tests/utils.py`, `tests/test_frappe_docker.py` — testing patterns and examples of compose/bench invocations
- `docs/` — living documentation; prefer linking to docs pages instead of duplicating content

## Integration & external dependencies

- This project depends on Docker Engine and docker-compose (v2+), and in CI it runs containers (minio, restic, etc.). Ensure access to the Docker daemon when running tests locally.
- Tests start a `minio` container for S3-compatible tests; they install Python deps inside the `backend` container when required (`boto3` in tests).

## How to make small, safe changes

- For compose or image changes: add/update an `overrides/*.yaml` and document the purpose in `docs/`.
- For tests: follow the `Compose` helper and fixture patterns in `tests/conftest.py`; avoid global side-effects outside `tmp_path` or `example.env` copy.
- For docs: update the appropriate `docs/*.md` file and reference it from `README.md`.

## When to ask for human review

- Any change touching production `compose` flows (`pwd.yml`) or images under `images/`.
- Changes that alter the default `example.env` variables or site creation flow.

---

If any section is unclear or you want examples added (sample bench commands, common compose flags, or CI details), tell me which area to expand and I will iterate.
