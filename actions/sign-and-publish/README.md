# sign-and-publish

Compute the connector content hash, sign the payload with the publisher's
ed25519 key, pack a flat tarball, and publish a GitHub release at
`vX.Y.Z`. The release body cross-links to every per-action release in the
cohort (which [`publish-action-subpaths`](../publish-action-subpaths/)
publishes from the same workflow run).

The content hash is `sha256(connector.wasm || manifest.toml)` — the
canonical-hash input used by the install pipeline. Producers and
consumers must concatenate in the same order; this action enforces the
producer side.

The tarball is a flat archive with exactly three files at the root:
`connector.wasm`, `manifest.toml`, `signature.sig`. No directory prefix.
The asset is uploaded as `aileron.tar.gz`.

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `signing-key` | yes |  | Base64-encoded ed25519 private key (PEM). Pass `${{ secrets.AILERON_SIGNING_KEY }}`. |
| `github-token` | yes |  | Token with `contents: write` on the publishing repo. |
| `connector-wasm` | no | `connector.wasm` | Path to the built WASM binary. |
| `manifest-path` | no | `connector/manifest.toml` | Path to the connector manifest. |
| `actions-dir` | no | `actions` | Directory holding per-action subdirectories (used to build cohort release notes). |
| `version` | no | derived from `GITHUB_REF` | Version being released. |

## Outputs

| Name | Description |
|---|---|
| `connector-hash` | `sha256:<hex>` content hash. Pass into `publish-action-subpaths` so per-action manifests bind to the same connector. |
| `tag` | Connector release tag (e.g. `v1.2.3`). |

## Required permissions

```yaml
permissions:
  contents: write
```

## Usage

```yaml
on:
  push:
    tags:
      - "v*.*.*"

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: ALRubinger/aileron-actions/actions/setup-go-wasm@v1
      - uses: ALRubinger/aileron-actions/actions/build-wasm-connector@v1
      - id: connector
        uses: ALRubinger/aileron-actions/actions/sign-and-publish@v1
        with:
          signing-key: ${{ secrets.AILERON_SIGNING_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
      - uses: ALRubinger/aileron-actions/actions/publish-action-subpaths@v1
        with:
          signing-key: ${{ secrets.AILERON_SIGNING_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          connector-hash: ${{ steps.connector.outputs.connector-hash }}
```
