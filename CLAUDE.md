# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a GitHub profile repository. The primary artifact is `README.md`, which renders on the owner's GitHub profile page.

## Key files

- `README.md` — profile content, including badges and embedded stats cards.
- `.github/workflows/github-readme-stats.yml` — scheduled workflow that regenerates SVG profile cards and commits updated files back to the repository.

## Development workflow

There is no application code, package manifest, or local test/build toolchain in this repository. Typical work is limited to editing Markdown and GitHub Actions workflow YAML.

A minimal `.devcontainer/` setup is available for containerized editing. It builds from `.devcontainer/Dockerfile`, installs Node.js through `nvm`, installs Claude Code globally with npm, persists Claude state in `/home/vscode/.claude`, and can forward `ANTHROPIC_API_KEY` from the host environment.

Common commands:

- View the profile locally by opening `README.md` in a Markdown preview.
- Validate workflow syntax by reviewing `.github/workflows/github-readme-stats.yml` carefully; there is no repo-local lint command configured.
- If you need to inspect generated assets, check whether a `profile/` directory exists after the workflow has run; it is not currently tracked in this checkout.

## Automation architecture

The repository's only automation is the `Update README cards` GitHub Actions workflow:

- Runs daily on cron (`0 0 * * *`) and also supports manual dispatch.
- Checks out the repository.
- Uses `readme-tools/github-readme-stats-action@v1` three times to generate SVG cards for overall stats, top languages, and a pinned repository card.
- Writes generated SVGs under `profile/`.
- Commits and pushes those generated SVG updates back to `main`.

## Content structure

`README.md` is a simple profile document with:

- short self-introduction bullets
- Wakapi badge/image embeds for coding activity
- a Wakatime-style stats card embed

When editing the profile, preserve external image URLs unless the task is specifically to replace or remove them.

## Notes for future edits

- The workflow currently performs `git config` inside GitHub Actions to author automated commits; that behavior belongs to CI, not local development.
- Because this is a profile repo, most changes are user-visible on GitHub immediately after push.
- Do not assume a missing `profile/` directory is an error in local checkout; those generated files may only appear after the workflow runs or if they are later committed.
