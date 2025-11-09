# my copy-paste Pre-commit Hooks

This repository ships a pre-commit configuration that enforces consistent style, validates file formats, checks commit messages, keeps my Python dependencies tidy, and runs tests before code lands on a branch.

1) General file hygiene (pre-commit-hooks)
   - trailing-whitespace, end-of-file-fixer, fix-byte-order-marker
   - Broken/unsafe bits: check-merge-conflict, check-symlinks, destroyed-symlinks,
   - check-case-conflict, check-illegal-windows-names, check-executables-have-shebangs,
   - check-vcs-permalinks, forbid-submodules, check-added-large-files
   - Debug & formats: debug-statements, check-yaml --allow-multiple-documents,
   - check-json, check-toml, check-xml
   - Security: detect-private-key (ignores creds/ and README.md)
   - Python: check-ast, name-tests-test (encourages tests/test_*.py)
   - Branch safety: no-commit-to-branch (blocks commits to main/master)

2) Commit-message & author style (gitlint)
   - Title length <= 80
   - Merge commits ignored
   - Commit title must match: `(feat|fix|try|maintain)!?(\(.*\))?: .+` or a standard merge message.
   - Author email must match: `*@fullstacks.eu` or `daniel@drackthor.me`
   - Runs against staged changes and reads the commit message file directly.

3) Python lint & format (ruff)
   - ruff-check (with --fix) for linting and quick autofixes
   - ruff-format for code formatting
   - Targets python and pyi files (skips notebooks)

4) Dependency lock consistency (uv-pre-commit)
   - uv-lock keeps the uv lockfile up to date and consistent

5) Test gate (pytest)
   - Local hook that runs uv run pytest before allowing a commit
   - Blocks commits on failing tests

## Requirements

- Python 3.8+ (project-specific, but ruff/uv like recent versions)
- pre-commit installed
- uv installed (for uv-lock and running tests)
- pytest available in the project’s environment

## Suggested installs

```shell
# pre-commit (pick one)
pipx install pre-commit
# or
pip install --user pre-commit
# or
brew install pre-commit

# uv (choose one)
curl -LsSf https://astral.sh/uv/install.sh | sh
# or on macOS:
brew install uv
```

The config pins:
- pre-commit-hooks at v6.0.0
- gitlint at v0.19.1
- ruff-pre-commit at v0.14.4
- uv-pre-commit at 0.9.8

## Quick start

From the repo root:

1) Install the git hook: `pre-commit install && pre-commit install --hook-type commit-msg`
2) (Optional) Run once over the whole repo: `pre-commit run --all-files`

Now, on every commit, the hooks will run on staged files and block the commit if something needs fixing. 
Many issues are auto-fixed; simply restage and commit again.

## Typical workflow

```shell
# code something
git add ...
pre-commit run
# or, to run on all files, not only changed ones
pre-commit run --all-files
# commit
git commit -m "feat: my awesome feature"
# will run the hook again (of course..)
```

### Skip all hooks (use sparingly)

`git commit --no-verify -m "chore: emergency commit"`


## Commit message rules (with examples)

Valid titles:

```yaml
feat(api): add pagination to /users list
fix(auth): handle expired refresh tokens
maintain: bump dev tooling
try(db): experimental index on events table

# Breaking change indicator
feat(config)!: remove deprecated ENV flags

# Invalid titles (will fail gitlint)
update stuff
fixes
(too long — over 80 chars) feat: very very very long title that goes on and on beyond eighty chars...
```

Author email requirement:

- Your Git author email must match `*@fullstacks.eu` or be `daniel@drackthor.me`.

Branch protection: 

- no-commit-to-branch prevents committing directly to main/master.
- If your default branch is named differently, configure Git accordingly

## Customizing the hooks

```yaml
# Example: add args to block commits to additional branches:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v6.0.0
  hooks:
    - id: no-commit-to-branch
      args: [--branch, main, --branch, master, --branch, release]

# Example: narrow detect-private-key excludes
- id: detect-private-key
  exclude: |
    (?x)(
        ^creds/|
        ^README.md|
        ^docs/
    )

# Example: run pytest on only changed test files (advanced; current config runs full suite)
- repo: local
  hooks:
    - id: pytest
      entry: uv run pytest
      language: system
      types: [python]
      pass_filenames: false  # set to true to pass paths of staged files; adjust your test command accordingly
```
