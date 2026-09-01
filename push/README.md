# TestSparrow Push Action

Send your `.testsparrow/` test definitions to TestSparrow from CI. A thin wrapper
around `testsparrow push` — it installs the `testsparrow` CLI and runs the push.

Direction: **user = push** (send local → SaaS); the server side is *ingest* (receive +
reconcile). Endpoint: `/v1/ingest`. Only changed files are uploaded (a full blob-SHA
manifest is negotiated first). See `docs/architecture/ingest-protocol.md`.

## Usage

```yaml
name: TestSparrow
on:
  push:
    branches: [main]        # tracked_branch → projection refresh + run
  pull_request:              # transient → run-only (projection untouched)

jobs:
  push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: testsparrow/actions/push@v0
        with:
          api-key: ${{ secrets.TESTSPARROW_API_KEY }}
          app: my-app
```

Create the API key in the TestSparrow web UI (**Org Settings → API keys**) and store it as
the repo secret `TESTSPARROW_API_KEY`.

## Inputs

| input | required | default | description |
|---|---|---|---|
| `api-key` | yes | — | TestSparrow API key (repo secret). |
| `app` | no | — | Target app slug. Omit to let the backend resolve it from the git remote. |
| `hostname` | no | `testsparrow.com` | Backend host (apex; scheme and the api./ingest. prefix are derived). A full URL is also accepted. |
| `ref` | no | pushed branch (`GITHUB_REF_NAME`) | Branch/ref to record. |
| `version` | no | `latest` | CLI version (a release tag, or `latest`). |
| `working-directory` | no | `.` | Directory that contains `.testsparrow/`. |

## Notes

- Runs on Linux and macOS runners (`ubuntu-latest` recommended). Windows support: TODO.
- Downloads a prebuilt `testsparrow` single-binary (Bun is embedded — no Bun/Node setup).
- Only the `.testsparrow/` subtree is sent; `.env` / `.secrets/` / `artifacts/` are excluded.
- For non-GitHub CI, run the CLI directly: `testsparrow push --app my-app` with
  `TESTSPARROW_API_KEY` in the environment.

## Publishing

Authored in the monorepo at `packages/github-actions/push/` and mirror-published to the
aggregation repo `testsparrow/actions` (each action as a subdir, so this one resolves as
`uses: testsparrow/actions/push@v0`) by the `actions-release` workflow — push an
`actions-v<X.Y.Z>` tag to release. It moves both the `vX.Y.Z` and floating `v<major>` tags on
the aggregation repo.

**Note:** the action downloads the `testsparrow` binary from `testsparrow/testsparrow`
releases (`testsparrow-<target>` assets from `cli-release`). Cut a CLI release first, else the
install step can't find a binary.
