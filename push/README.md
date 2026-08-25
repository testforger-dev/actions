# TestForger Push Action

Send your `.testforger/` test definitions to TestForger from CI. A thin wrapper
around `testforger push` — it installs the `testforger` CLI and runs the push.

Direction: **user = push** (send local → SaaS); the server side is *ingest* (receive +
reconcile). Endpoint: `/v1/ingest`. Only changed files are uploaded (a full blob-SHA
manifest is negotiated first). See `docs/architecture/ingest-protocol.md`.

## Usage

```yaml
name: TestForger
on:
  push:
    branches: [main]        # tracked_branch → projection refresh + run
  pull_request:              # transient → run-only (projection untouched)

jobs:
  push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: testforger-dev/actions/push@v0
        with:
          api-key: ${{ secrets.TESTFORGER_API_KEY }}
          app: my-app
```

Create the API key in the TestForger web UI (**Org Settings → API keys**) and store it as
the repo secret `TESTFORGER_API_KEY`.

## Inputs

| input | required | default | description |
|---|---|---|---|
| `api-key` | yes | — | TestForger API key (repo secret). |
| `app` | no | — | Target app slug. Omit to let the backend resolve it from the git remote. |
| `api-url` | no | `https://api.testforger.com` | Backend base URL. |
| `ref` | no | pushed branch (`GITHUB_REF_NAME`) | Branch/ref to record. |
| `version` | no | `latest` | CLI version (a release tag, or `latest`). |
| `working-directory` | no | `.` | Directory that contains `.testforger/`. |

## Notes

- Runs on Linux and macOS runners (`ubuntu-latest` recommended). Windows support: TODO.
- Downloads a prebuilt `testforger` single-binary (Bun is embedded — no Bun/Node setup).
- Only the `.testforger/` subtree is sent; `.env` / `.secrets/` / `artifacts/` are excluded.
- For non-GitHub CI, run the CLI directly: `testforger push --app my-app` with
  `TESTFORGER_API_KEY` in the environment.

## Publishing

Authored in the monorepo at `packages/github-actions/push/` and mirror-published to the
aggregation repo `testforger-dev/actions` (each action as a subdir, so this one resolves as
`uses: testforger-dev/actions/push@v0`) by the `actions-release` workflow — push an
`actions-v<X.Y.Z>` tag to release. It moves both the `vX.Y.Z` and floating `v<major>` tags on
the aggregation repo.

**Note:** the action downloads the `testforger` binary from `testforger-dev/testforger`
releases (`testforger-<target>` assets from `cli-release`). Cut a CLI release first, else the
install step can't find a binary.
