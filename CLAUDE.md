# CLAUDE.md

`hotcodepush-team/release-action`, the GitHub Action that runs the HotCodePush CLI's `release create` from a workflow.
The repo is public and MIT. Nothing internal goes here: no private plans, no customer, no credential.

## One file

`action.yml` is the whole action: a composite action with two `bash` steps, no `src/`, no build, no dependencies.

1. **Mask secrets** masks the token and the private key line by line, since `::add-mask::` masks one line per command.
2. **Release** builds the `release create --json --yes` arguments from the inputs, runs the CLI with `npx`, writes the outputs and the job summary from the JSON with `jq`, and turns the CLI's error JSON into an `::error` annotation and its exit code.

Rules for changing it:

- Every input is the CLI flag without `--`, described in the CLI's words. Check a new or renamed flag against the CLI's `src/commands/release/create.ts`.
- Pass inputs only through the step's `env:` and quote `"$VAR"`. Never write `${{ inputs.* }}` inside `run:`.
- Add a flag only when its input is set, with an `if` block, so `set -euo pipefail` stays safe. Booleans compare to `"true"`.
- `package.json` only carries the version release-please bumps. It is not part of the action.

## Releases

- release-please (`.github/workflows/release.yml`) keeps a release PR open on `main`. Merging it bumps `package.json`, writes `CHANGELOG.md`, tags and creates the GitHub release. Pre-1.0, `feat:` bumps the patch and a breaking change the minor.
- The `@v<version>` references in `README.md` carry `x-release-please-version`, so the release PR bumps them. Never edit them by hand.
- The Marketplace listing is manual: after merging the release PR, edit the release on GitHub and tick **Publish this Action to the GitHub Marketplace** (needs 2FA). The API release-please uses cannot set it.
- When the CLI releases, bump `cli-version`'s default in `action.yml` and the README's input list.

## CI

`.github/workflows/ci.yml` lints the workflows with actionlint and dry-runs a release of `test/app` against the staging API.
The dry run needs the `HOTCODEPUSH_TOKEN` secret and is skipped without it. `test/app/hotcodepush.json` must name an app the token can see.

- A test earns its place by catching a bug nothing else catches, at the lowest layer that can prove it; one test per branch, never per value, and nothing for what the types, the framework or a library guarantee.
- A new test names the behaviour or the bug it proves; if deleting the production code would not fail it, it does not get in.

## Agent workspace

- No `.mcp.json`: no server covers this repo's stack yet; the HotCodePush server joins when it exists.
- `.claude/skills/` holds the developer skills copied from `hotcodepush-team/.github` with the skills CLI and pinned in `skills-lock.json`; update them with `npx skills update`.
- `.github/copilot-instructions.md` holds the review criteria, the only Copilot-specific file.
- Commits are conventional commits; `main` is trunk, CI is the gate, and a commit that lands an issue says `Closes #<n>`.
