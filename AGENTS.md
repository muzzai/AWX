# Repository Guidelines

## Project Structure & Module Organization
AWX splits backend, frontend, and deployment helpers so changes stay scoped. Python API and task code live under `awx/`, Django apps under `awx/main/`, and Celery tasks in `awx/main/tasks/`. The React UI resides in `awx/ui/client/` with feature folders under `src/`. Installers and container assets live in `tools/docker-compose/` and `installer/`. Place new modules near their runtime touchpoint and add a brief `README.md` whenever you create a new top-level directory.

## Build, Test, and Development Commands
Run `make docker-compose` to start the Postgres, Redis, web, and task containers, and `make docker-compose-build` after dependency changes. `make test` executes backend pytest suites inside the stack; use `make awx-shell COMMAND="manage.py check"` for quick smoke checks. In `awx/ui/client/`, run `npm ci && npm run start` for live reload, `npm run lint` for formatting, and finish with `pre-commit run --all-files` before pushing.

## Coding Style & Naming Conventions
Python follows Black (88 columns) and isort ordering; use `python -m black awx` and `python -m isort awx` prior to commits. Prefer explicit module paths such as `awx.main.models.<Model>`. React components adopt PascalCase filenames, hooks use `useCamelCase`, and colocated SCSS modules mirror the component name. YAML and Ansible content uses two-space indentation with documented vars at the top.

## Testing Guidelines
Mirror the app layout under `awx/main/tests/` and name files `test_<feature>.py`. Cover new migrations, Celery tasks, and RBAC edge cases with both success and failure assertions. UI tests live in `awx/ui/client/src/**/__tests__` using Jest with Testing Library; keep snapshot coverage for purely presentational components. Run `npm test` for the UI suite and `pytest --cov=awx --cov-report=term-missing` when gauging overall backend impact.

## Commit & Pull Request Guidelines
Write imperative subjects under 72 characters and reference issue keys when they exist (e.g., `Fix job template launch failure`). Bundle related changes into commits that pass linting locally. Pull requests should include a concise summary, verification steps, and screenshots or CLI transcripts for user-facing updates. Request review from the relevant domain owner (backend, UI, installer) and wait for CI green before merging.

## Security & Configuration Tips
Never commit `.env`, secrets, or controller credentials. Keep developer overrides in `awx/settings/local_settings.py` and document required env vars in `docs/configuration.md`. Validate RBAC-sensitive flows with least-privilege accounts, and run `npm audit` plus `pip install --require-hashes -r requirements/requirements.txt` on dependency updates.
