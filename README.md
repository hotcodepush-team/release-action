# HotCodePush Release Action

Release an over-the-air (OTA) update to a channel from GitHub Actions with the [HotCodePush CLI](https://github.com/hotcodepush-team/cli).

[![CI](https://github.com/hotcodepush-team/release-action/actions/workflows/ci.yml/badge.svg)](https://github.com/hotcodepush-team/release-action/actions/workflows/ci.yml)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-HotCodePush%20Release-blue?logo=github)](https://github.com/marketplace/actions/hotcodepush-release-action)
[![License](https://img.shields.io/github/license/hotcodepush-team/release-action)](./LICENSE)

> [!NOTE]
> Store the `token` and the `private-key` as [encrypted secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions), never in the workflow file. Pin the action to a fixed version such as `@v0.1.0`, since no moving major tag is kept. <!-- x-release-please-version -->

## Usage

Build the app's web assets, then release them to a channel:

```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-node@v7
        with:
          node-version: 22
      - run: npm ci && npm run build
      - uses: hotcodepush-team/release-action@v0.1.0 # x-release-please-version
        with:
          token: ${{ secrets.HOTCODEPUSH_TOKEN }}
          private-key: ${{ secrets.HOTCODEPUSH_SIGNING_KEY }}
          channel: production
```

Every input:

```yaml
- uses: hotcodepush-team/release-action@v0.1.0 # x-release-please-version
  with:
    # The channel, by id or name, one per line or comma-separated; hotcodepush.json's channel by default.
    channel: ''
    # The build to upload, the framework's build output by default: on React Native and Expo one platform's prepared bundle, Capacitor's webDir or Cordova's www.
    path: ''
    # A bundle already uploaded, by number or id; without it the build is uploaded first.
    bundle: ''
    # Release what this channel serves, by id or name, instead of a bundle or a build.
    from-channel: ''
    # The bundle's version label; the project's package.json version by default.
    bundle-version: ''
    # The platforms the bundle serves, `ios,android` by default.
    platform: ''
    # The commit to record, over the detected one.
    git-sha: ''
    # The branch or tag to record, over the detected one.
    git-ref: ''
    # The commit message subject to record, over the detected one.
    git-message: ''
    # Detect no git provenance when `true`; the `git-*` inputs still apply.
    no-git: ''
    # The share of devices the release reaches, 0 to 100; 100 by default, the schedule's first percentage with `progression`.
    rollout-percentage: ''
    # Widen the rollout step by step while the release stays healthy when `true`: 10, 50 and 100 percent by default, each step after at least an hour and 50 applied updates.
    progression: ''
    # The percentages the rollout widens through, rising, the last above the starting rollout, with `progression`; `10,50,100` by default.
    progression-percentages: ''
    # The seconds a step lasts at least before the next one, with `progression`; 3600 by default.
    progression-minimum-seconds: ''
    # The applied updates each step needs before the next one, with `progression`; 50 by default.
    progression-minimum-update-count: ''
    # Devices apply the release at once and restart when `true`.
    mandatory: ''
    # The release notes; they ride the public index.
    notes: ''
    # The app versions served, a semver range such as `>=2.3.0 <3.0.0`.
    binary: ''
    # The OS versions served, a range such as `>=17`.
    os: ''
    # An attribute the app sets that the device must carry, `key=value`, one per line or comma-separated.
    attribute: ''
    # A device the release is limited to, by id, one per line or comma-separated.
    device: ''
    # What auto-pause does when a release fails too often: `pause`, `revoke`, `notify` or `none`.
    failure-action: ''
    # The failure percentage at which auto-pause acts.
    failure-threshold: ''
    # The applied updates a release needs before auto-pause judges it.
    failure-minimum-update-count: ''
    # Validate and print the audience the release would reach when `true`; nothing is signed, uploaded or published, so no private key is needed.
    dry-run: ''
    # The app, by id or name, when no hotcodepush.json is found or another app is meant.
    app: ''
    # The organization, by id or name, when you belong to several.
    organization: ''
    # The hotcodepush.json to use, when it is not the nearest one up from the working directory.
    config: ''
    # The content of the private key file to sign with, as `signing-key create` wrote it.
    private-key: ''
    # The API the token belongs to; the production API by default.
    api-url: ''
    # The directory the CLI runs in.
    # Default: `.`.
    working-directory: ''
    # The HotCodePush CLI version to run, or an npm package spec such as a tarball URL.
    # Default: `0.1.0`.
    cli-version: ''
    # The HotCodePush API token.
    # Required.
    token: ''
```

## Outputs

| Name | Description |
| --- | --- |
| `releases` | The JSON the CLI printed: the created releases, or on a dry run the audience per channel. |
| `release-id` | The id of the first release; empty on a dry run. |
| `release-number` | The number of the first release; empty on a dry run. |
| `channel-id` | The id of the first channel. |
| `bundle-id` | The id of the bundle the first release carries; empty on a dry run. |
| `bundle-number` | The number of the bundle the first release carries; empty on a dry run. |
| `rollout-percentage` | The rollout percentage of the first release; empty on a dry run. |

## Notes

- The runner needs Node.js 22 or later. The action runs the CLI with `npx`, so add `actions/setup-node` where the runner has an older Node.js.
- The action reads the CLI's JSON with `jq`, which GitHub-hosted runners have preinstalled. Install it on a self-hosted runner.
- `channel`, `attribute` and `device` take several values, one per line or comma-separated on one line. When a value contains a comma, use one line per value.
- A dry run (`dry-run: 'true'`) signs, uploads and publishes nothing, so it needs no `private-key`. Use it on pull requests to check the release and see the audience it would reach.
- The bundle's version label is the project's `package.json` version by default. Set `bundle-version: ${{ github.sha }}` to label each bundle with its commit instead.
- `cli-version` takes a CLI version or any npm package spec, such as a tarball URL.
- The action writes a job summary with a row per release, or per channel on a dry run.
- When the CLI fails, the action annotates the run with the CLI's error code, message and fix, and exits with the CLI's exit code.

## Documentation

See [GitHub Actions](https://hotcodepush.com/docs/ci/github-actions) in the HotCodePush docs.

## License

See [LICENSE](./LICENSE).
