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

- Conventional Commits. release-please (`.github/workflows/release.yml`) keeps a release PR open on `main`. Merging it bumps `package.json`, writes `CHANGELOG.md`, tags and creates the GitHub release. Pre-1.0, `feat:` bumps the patch and a breaking change the minor.
- The `@v<version>` references in `README.md` carry `x-release-please-version`, so the release PR bumps them. Never edit them by hand.
- The Marketplace listing is manual: after merging the release PR, edit the release on GitHub and tick **Publish this Action to the GitHub Marketplace** (needs 2FA). The API release-please uses cannot set it.
- When the CLI releases, bump `cli-version`'s default in `action.yml` and the README's input list.

## CI

`.github/workflows/ci.yml` lints the workflows with actionlint and dry-runs a release of `test/app` against the staging API.
The dry run needs the `HOTCODEPUSH_TOKEN` secret and is skipped without it. `test/app/hotcodepush.json` must name an app the token can see.
