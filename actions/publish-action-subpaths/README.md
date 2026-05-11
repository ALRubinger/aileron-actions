# publish-action-subpaths

For each subdirectory under `actions/`, substitute the
`sha256:bound-at-release` placeholder in `action.md` with the real
connector hash, sign the substituted manifest with the publisher's
ed25519 key, pack a flat tarball, and publish a GitHub release at
`actions/<name>/vX.Y.Z` (prerelease).

Each action release links back to the connector release (and to every
sibling action in the cohort) so a reader landing on any artifact can
find the others.

Marked `--prerelease` so the connector release stays as GitHub's
"Latest" — per-action releases are sibling artifacts in the same cohort,
not a successor release stream.

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `signing-key` | yes |  | Base64-encoded ed25519 private key (PEM). Pass `${{ secrets.AILERON_SIGNING_KEY }}`. |
| `github-token` | yes |  | Token with `contents: write` on the publishing repo. |
| `connector-hash` | yes |  | `sha256:<hex>` content hash from `sign-and-publish`. Every action manifest binds to this hash. |
| `actions-dir` | no | `actions` | Directory holding per-action subdirectories. |
| `version` | no | derived from `GITHUB_REF` | Version being released. |

## Outputs

None.

## Action manifest contract

Each `actions/<name>/action.md` must:

- Carry `hash = "sha256:bound-at-release"` as a placeholder. This action
  rewrites it to the real connector hash before signing.
- Be the only file under `actions/<name>/` that the release process
  reads. Anything else in the subdirectory is ignored.

After substitution and signing, each per-action tarball contains exactly
two files at the root: `action.md` and `signature.sig`.

## Usage

```yaml
- id: connector
  uses: ALRubinger/aileron-actions/actions/sign-and-publish@v0.0.1
  with:
    signing-key: ${{ secrets.AILERON_SIGNING_KEY }}
    github-token: ${{ secrets.GITHUB_TOKEN }}

- uses: ALRubinger/aileron-actions/actions/publish-action-subpaths@v0.0.1
  with:
    signing-key: ${{ secrets.AILERON_SIGNING_KEY }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    connector-hash: ${{ steps.connector.outputs.connector-hash }}
```
