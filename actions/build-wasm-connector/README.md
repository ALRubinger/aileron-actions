# build-wasm-connector

Substitute the placeholder version into the connector manifest and every
action manifest, then compile `connector/main.go` to a `wasip1` WASM binary.

The committed source carries `version = "0.0.0-dev"` so the publisher
never hand-edits a version field; this action substitutes the real
version (derived from the pushed `v*.*.*` tag) into the working tree
before the rest of the pipeline hashes, signs, or packs anything.

`GOOS=wasip1` and `GOARCH=wasm` are scoped to the `go build` step only —
other Go commands in the same job (for example `go test` against the
host portion of the connector) keep their normal target.

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `connector-dir` | no | `connector` | Directory containing `main.go`, `manifest.toml`, and `go.mod`. |
| `actions-dir` | no | `actions` | Directory holding per-action subdirectories with `action.md` files. |
| `version` | no | derived from `GITHUB_REF` | Version to substitute for `0.0.0-dev`. Defaults to the pushed tag with the leading `v` stripped. |
| `output-path` | no | `connector.wasm` | Path (relative to the workspace root) to write the built WASM binary. |

## Outputs

| Name | Description |
|---|---|
| `version` | The version that was substituted into the manifests. Pass downstream to `sign-and-publish` / `publish-action-subpaths` when you override the default. |
| `connector-wasm` | Path to the built WASM binary. |

## Usage

```yaml
on:
  push:
    tags:
      - "v*.*.*"

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: ALRubinger/aileron-actions/actions/setup-go-wasm@v0.0.1
      - uses: ALRubinger/aileron-actions/actions/build-wasm-connector@v0.0.1
```

## Manifest template contract

The committed `connector/manifest.toml` must contain literal `0.0.0-dev`
in its version field. Each `actions/*/action.md` must contain the same
placeholder. The action fails the build if any placeholder remains after
substitution.
