# Project Conventions

## Root = Config Only

Only config files belong at the project root (README.md, LICENSE, .gitignore, package.json, pyproject.toml, Cargo.toml, tsconfig.json, docker-compose.yml, Makefile, .env.example, dotfiles, lock files). A cluttered root makes it hard to find what matters and signals a disorganized project.

## Directory Structure

- `src/` or `lib/` - production code
- `tests/` - all test files (TDD.md governs)
- `docs/` - documentation
- `scripts/` - reusable utilities
- `tmp/` - temporary files (gitignored)

## Git Discipline

- **Commit after every completed task group.** Small, frequent commits create rollback points and keep diffs reviewable.
- **Never commit `tmp/` or `.claude/context/`.** Temporary artifacts pollute history and bloat the repo.
- **Use `./tmp/` never `~/tmp`.** Project-scoped temp directories prevent cross-project contamination and clean up naturally.
