# Repository Guidelines

## Project Overview

This repository is a GitHub profile README and generated-card asset repository, not a checked-in Vue or TypeScript application. `README.md` presents the `@hoshinorei` profile and embeds external Wakapi/Shields badges plus local SVG cards from `profile/`. The README mentions TypeScript and Vue 3 as the author's preferred technologies; no application source, package manifest, or app entry point is present here.

For changes, treat the README, the card-generation workflow, and the Dev Container files as the behavioral sources of truth. Do not invent application modules, npm scripts, or framework conventions that are not in this checkout.

## Architecture & Data Flow

There are two small, independent flows:

1. **Profile rendering:** GitHub renders `README.md`; external time badges load from Wakapi/Shields URLs, and local references load `profile/top-langs.svg`, `profile/stats.svg`, and `profile/wakatime.svg`.
2. **Card publication:** `.github/workflows/github-readme-stats.yml` runs daily at `16:00 UTC` (`0 16 * * *`) or through `workflow_dispatch`, checks out the repository, generates the three referenced cards with `readme-tools/github-readme-stats-action@v1`, then commits and pushes `profile/*.svg` using `GITHUB_TOKEN`.

The Dev Container has a separate lifecycle flow: `initializeCommand` runs `.devcontainer/init.sh` on the host to create and fix ownership of the persistent Claude configuration volume; `postStartCommand` runs `.devcontainer/post-start.sh` to add `hasCompletedOnboarding: true` to `.claude.json` when that key is absent.

`profile/pin-readme-tools-github-readme-stats.svg` is checked in but is neither referenced by `README.md` nor produced by the current workflow. `profile/stats.svg` currently contains a generated “Resource not accessible by integration” message; do not assume the workflow has recently succeeded.

## Key Directories

- `profile/` — checked-in SVG card outputs and the currently unused pinned-card asset.
- `.github/workflows/` — GitHub Actions automation for refreshing the README cards.
- `.devcontainer/` — Ubuntu development environment definition, feature lock, and Bash lifecycle scripts.
- Repository root — `README.md` is the user-facing document; `.gitignore` excludes local Claude settings and worktrees.

## Development Commands

No install, build, start, lint, format, typecheck, or test command is defined. There is no `package.json`, package-manager lockfile, compiler configuration, or task-runner file.

The only project-provided commands are Dev Container lifecycle commands:

```bash
bash .devcontainer/init.sh [volume-name] [uid] [gid]
bash .devcontainer/post-start.sh [json-file]
```

`init.sh` defaults to volume `claude-code-config` and UID/GID `1000`; it requires Bash and Docker access. `post-start.sh` defaults to `/home/vscode/.claude.json` and requires `jq`. In normal use these are invoked automatically by `.devcontainer/devcontainer.json`. To refresh cards, manually dispatch **Update README cards** in GitHub Actions; the workflow also runs on its configured schedule.

## Code Conventions & Common Patterns

- Shell scripts use `#!/usr/bin/env bash`, `set -eu`, quoted variable expansions, snake_case variables, and top-level sequential commands. Preserve fail-fast behavior when editing them.
- Shell filenames use kebab-case; Dev Container JSON keys use camelCase.
- `init.sh` uses `docker volume inspect || docker volume create` for idempotent volume setup, then runs Alpine to `chown` the mounted volume.
- `post-start.sh` validates that existing JSON is an object, adds the onboarding key only when absent, and uses `mktemp`/`cmp`/`mv` to avoid rewriting unchanged files. Preserve this idempotence.
- Workflow changes belong in `.github/workflows/github-readme-stats.yml`; generated SVGs may be overwritten by the next scheduled or manual run. Keep output paths aligned with the README references.
- There is no repository-level TypeScript, Vue, dependency-injection, UI-state, or asynchronous application pattern to copy.

## Important Files

- `README.md` — profile content and local/external badge references.
- `.github/workflows/github-readme-stats.yml` — card options, output paths, trigger schedule, token usage, and commit/push behavior.
- `.devcontainer/devcontainer.json` — base image, installed features, environment forwarding, mounts, and lifecycle hooks.
- `.devcontainer/init.sh` — persistent-volume creation and ownership setup.
- `.devcontainer/post-start.sh` — Claude onboarding JSON normalization.
- `.devcontainer/devcontainer-lock.json` — pinned feature versions and digests; this is not an application dependency lockfile.
- `profile/*.svg` — generated/published README assets; inspect the workflow before editing one directly.
- `.gitignore` — local-only `.claude/settings.local.json` and `.claude/worktrees/` exclusions.

## Runtime/Tooling Preferences

The configured environment is `mcr.microsoft.com/devcontainers/base:ubuntu`, running as `vscode`, with the jq, GitHub CLI, Node, and Claude Code Dev Container features. The Node feature is present, but this repository does not specify a Node.js runtime version or require a package manager. Feature resolutions are pinned in `.devcontainer/devcontainer-lock.json`.

The Dev Container forwards `ANTHROPIC_API_KEY`, `ANTHROPIC_BASE_URL`, `GH_TOKEN`, `WAKATIME_API_KEY`, and `WAKATIME_API_URL`. Treat these as local secrets/integration inputs; never hard-code or commit them. Local Claude settings and worktrees are intentionally ignored.

## Testing & QA

No test framework, test files, fixtures, mocks, coverage configuration, lint job, or validation workflow exists. The sole CI workflow generates and publishes profile cards; its job named `build` is not an application build.

For changes, use the narrowest evidence available:

- Check Markdown links and the three local SVG paths in `README.md` after README edits.
- For workflow edits, verify output paths, action options, permissions, and the `GITHUB_TOKEN` input before dispatching the workflow.
- For shell edits, run the affected script only in an environment with the required Docker/jq prerequisites and a disposable configuration target; no repository test harness exists.
- Treat generated-card freshness and external API/action availability as runtime concerns, not locally verified facts.
