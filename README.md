# aileron-actions

Reusable composite GitHub Actions for publishing Aileron connectors and
their per-action tarballs. Consolidates the ~12KB `release.yml` that
every connector repo would otherwise duplicate down to ~30 lines of
workflow per repo.

See [github.com/ALRubinger/aileron](https://github.com/ALRubinger/aileron)
for the runtime that consumes these published artifacts.

## Actions

| Action | Purpose |
|---|---|
| [`setup-go-wasm`](./actions/setup-go-wasm/) | Install the Go toolchain pinned to the connector's `go.mod`. |
| [`build-wasm-connector`](./actions/build-wasm-connector/) | Substitute placeholder versions and compile `connector/main.go` to a `wasip1` WASM binary. |
| [`sign-and-publish`](./actions/sign-and-publish/) | Hash, ed25519-sign, pack the connector tarball, and publish the `vX.Y.Z` release. |
| [`publish-action-subpaths`](./actions/publish-action-subpaths/) | Substitute the connector-hash placeholder, sign, pack, and publish each per-action release at `actions/<name>/vX.Y.Z`. |

Each action's README documents its inputs, outputs, and contract with
the connector source layout.

## Minimal connector release.yml

```yaml
name: Release
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

A connector with custom release-time work (for example, substituting a
provider-issued OAuth `client_secret` from a repo secret) keeps that
work as an inline step alongside the composite actions — anything
provider-specific stays out of `aileron-actions`.

## Pinning

Consumers can pin by tag or commit SHA:

```yaml
# Tag (tracks the major version, accepts patches)
- uses: ALRubinger/aileron-actions/actions/setup-go-wasm@v1

# Commit SHA (immutable; recommended for supply-chain trust)
- uses: ALRubinger/aileron-actions/actions/setup-go-wasm@<commit-sha>
```

For production connector pipelines, **prefer SHA pinning**. A composite
action can shell out and read repo secrets; pinning to a SHA prevents a
silent retag from changing what runs against your signing key.

## Required secrets in the consuming repo

| Secret | Purpose |
|---|---|
| `AILERON_SIGNING_KEY` | Base64-encoded ed25519 private key (PEM). Used by both `sign-and-publish` and `publish-action-subpaths`. |
| `GITHUB_TOKEN` | Provided automatically by GitHub Actions; consumers pass it through and grant `contents: write` on the job. |

## Connector source layout assumed

```
connector/
  main.go
  manifest.toml      # version = "0.0.0-dev" placeholder
  go.mod
actions/
  <name>/
    action.md        # hash = "sha256:bound-at-release" placeholder
```

`build-wasm-connector` swaps `0.0.0-dev` for the tag-derived version.
`publish-action-subpaths` swaps `sha256:bound-at-release` for the real
connector content hash computed by `sign-and-publish`.

## License

Apache-2.0. See [LICENSE](./LICENSE).
