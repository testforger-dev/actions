# TestSparrow Image Push Action

Build a custom image and push it to TestSparrow from CI (BYO client-build-push, §13). Wraps
`testsparrow image publish` — the runner's docker builds the image, then it's uploaded via a
presigned URL and TestSparrow copies it into Artifact Registry. TestSparrow never builds it.

Image push hits the **Public API** (`api.testsparrow.com`, Bearer) — a different surface from
`push` (which is git ingest on `ingest.testsparrow.com`).

## Usage

Images change rarely, so run this on Dockerfile changes / manually — **not** on every commit.

```yaml
name: TestSparrow Image
on:
  push:
    paths: ['Dockerfile', '.testsparrow/**/Dockerfile']
  workflow_dispatch:

jobs:
  image:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: testsparrow/actions/image-push@v0
        with:
          api-key: ${{ secrets.TESTSPARROW_API_KEY }}
          app: my-app
          tag: my-app:ci-${{ github.sha }}
```

## Inputs

| input | required | default | description |
|---|---|---|---|
| `api-key` | yes | — | TestSparrow API key (repo secret). |
| `app` | yes | — | Target app slug. |
| `tag` | yes | — | Image tag to build and push. |
| `dockerfile` | no | `./Dockerfile` | Path to the Dockerfile (`-f`). |
| `context` | no | `.` | Build context directory. |
| `api-url` | no | `https://api.testsparrow.com` | Public API base URL. |
| `version` | no | `latest` | CLI version (a release tag, or `latest`). |
| `working-directory` | no | `.` | Directory to run from. |

## Notes

- Needs docker on the runner (`ubuntu-latest` has it).
- BYO uploads the full image each push (no upload-side layer dedup) — cheap on the GCS side
  (ingress is free, PUT ops are negligible), but it costs your runner's upload bandwidth.
  The recurring cost is Artifact Registry storage (bounded by the per-plan image quota).
- For non-GitHub CI, run the CLI directly: `testsparrow image publish --app my-app --tag <t>`
  with `TESTSPARROW_API_KEY` in the environment.
