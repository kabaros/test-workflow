# Sample workflow for DHIS2 apps

## Permissions required

## Simple workflow
In the simple workflow, we use semantic-release library directly:

1. Add the needed dev dependencies to the project

```
yarn add -D @semantic-release/changelog @semantic-release/git
```

1. Add a workflow to `.github/workflows`

```yaml
on:
    push:
        branches:
            - main

permissions:
    contents: read

jobs:
    release:
        runs-on: ubuntu-latest
        permissions:
            contents: write
            pull-requests: write
        steps:
            - uses: actions/checkout@v4
              with:
                  fetch-depth: 0
            - uses: actions/setup-node@v4
              with:
                  node-version: "lts/*"
                  cache: yarn
            - run: yarn install --frozen-lockfile
            - run: npx semantic-release
              env:
                  GITHUB_TOKEN: ${{ secrets.DHIS2_BOT_GITHUB_TOKEN }}
```
## Advanced workflow

This uses DHIS2 internal reusable workflow which allows a more powerful (but opinionated setup). This workflow:

- Lints the commits to ensure they follow semantic release
- Publishes to GitHub
- Also publishes to AppHub if an AppHub token is provided

This is geared towards applications developed with the DHIS2 platform tools, so it has some additional prerequisites, namely the use `@dhis2/cli-app-scripts` and `@dhis2/cli-style`, and the presence of a `d2.config.js` file.


Here is a sample setup for a workflow (you can add to `.github/workflows/release.yml` for example):


```yaml
name: test-and-release

on: push

concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    # Cancel previous runs if not on a release branch
    cancel-in-progress: ${{ !contains(fromJSON('["refs/heads/master", "refs/heads/main"]'), github.ref) }}

jobs:
    lint-commits:
        uses: dhis2/workflows-platform/.github/workflows/lint-commits.yml@v1
    # lint:
    #     uses: dhis2/workflows-platform/.github/workflows/lint.yml@v1
    # test:
    #     uses: dhis2/workflows-platform/.github/workflows/test.yml@v1
    # e2e:
    #     uses: dhis2/workflows-platform/.github/workflows/legacy-e2e.yml@v1
    #     # Skips forks and dependabot PRs
    #     if: '!github.event.push.repository.fork'
    #     secrets: inherit
    #     with:
    #         api_version: 41
    release:
        needs: [lint-commits]
        uses: dhis2/workflows-platform/.github/workflows/release.yml@v1
        # Skips forks and dependabot PRs
        if: '!github.event.push.repository.fork'
        # secrets: inherit
        with:
          publish_apphub: false
          publish_github: true
        secrets:
          DHIS2_BOT_APPHUB_TOKEN: ${{ secrets.DHIS2_BOT_APPHUB_TOKEN }}
          DHIS2_BOT_GITHUB_TOKEN: ${{ secrets.DHIS2_BOT_GITHUB_TOKEN }}

```