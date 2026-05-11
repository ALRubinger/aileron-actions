# setup-go-wasm

Install the Go toolchain pinned to the connector's `go.mod`. Pair with
[`build-wasm-connector`](../build-wasm-connector/) to compile the
`connector/main.go` source to a `wasip1` WASM binary.

`GOOS=wasip1` and `GOARCH=wasm` are set inside `build-wasm-connector`'s
`go build` step rather than exported globally, so other Go commands in
the same job (for example `go test` against the host portion of the
connector) keep their normal target.

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `go-version-file` | no | `connector/go.mod` | Path to `go.mod` or `go.work` used to pin the Go toolchain version. |

## Outputs

None.

## Usage

```yaml
- uses: actions/checkout@v5
- uses: ALRubinger/aileron-actions/actions/setup-go-wasm@v0.0.1
```

Pin by commit SHA in production for supply-chain trust:

```yaml
- uses: ALRubinger/aileron-actions/actions/setup-go-wasm@<commit-sha>
```
