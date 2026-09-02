# Immutable action release demo

This repository demonstrates a staged GitHub action release. A dispatch selects
a version bump, publishes an immutable release candidate (RC), validates the
action at the candidate commit, and publishes the verified asset as an
immutable stable release.

> [!IMPORTANT]
> In repository settings, go to the "Releases" section and select
> **Enable release immutability** before running the workflow. The setting
> applies only to future releases.

```mermaid
flowchart LR
  D[Dispatch major, minor, or patch] --> C[Select next vX.Y.Z-rc.N]
  C --> B[Build and attest asset]
  B --> R[Publish immutable RC]
  R --> T[Call reusable validation]
  T --> S[Create stable tag at same SHA]
  S --> P[Publish immutable stable release]
```

## Release guarantees

| Guarantee | How |
| --- | --- |
| Serialized version selection | One `release` concurrency group |
| Predictable versions | Latest stable tag plus `major`, `minor`, or `patch`; existing RC tags increment `rc.N` |
| Exact source under test | The RC tag, reusable workflow, and `$/` action resolve to the release run's commit |
| Protected tags | Tags are created once without force; a repository ruleset blocks updates and deletion |
| Immutable publication | Releases are drafted, assets attached, then published with release immutability enabled |
| Build provenance | GitHub artifact attestation binds the archive to its source workflow and commit |
| Promotion without rebuilding | The verified RC asset is downloaded and attached to the stable release |
| Least privilege | Each job declares only the permissions it needs |
| Pinned dependencies | `actions.lock` pins workflow action versions |

## Run a release

Open the [Release workflow](../../actions/workflows/release.yml), select
`patch`, `minor`, or `major`, then run it from `main`.

If the latest stable tag is `v1.4.2`, a minor release starts with
`v1.5.0-rc.1`. If validation fails, or a workflow on `main` changes before
promotion, the published prerelease remains as an audit record. Run the
workflow again with the same bump to create `v1.5.0-rc.2`. Only a candidate
that passes validation is published as `v1.5.0`.

## Why no dynamic ref is needed

The release workflow creates the RC tag at its own commit, then calls
`release-test.yml` with `uses: $/.github/workflows/release-test.yml`. The
reusable workflow and its `uses: $/` action both resolve from that same commit.
The candidate tag is passed as an input only to verify the immutable release
and asset.

GitHub Actions does not allow expressions in `uses:`, but this flow does not
need one: the caller commit is the candidate commit.

## Verify a release

```bash
gh release verify v0.1.1 --repo nodeselector/actions-release-demo
gh release download v0.1.1 --repo nodeselector/actions-release-demo
gh release verify-asset v0.1.1 actions-release-demo-v0.1.1.tar.gz \
  --repo nodeselector/actions-release-demo
gh attestation verify actions-release-demo-v0.1.1.tar.gz \
  --repo nodeselector/actions-release-demo
```

See GitHub's documentation for
[immutable releases](https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases)
and [release verification](https://docs.github.com/en/code-security/how-tos/secure-your-supply-chain/secure-your-dependencies/verify-release-integrity).

## Live proof

| Artifact | Evidence |
| --- | --- |
| Full release pipeline | [Patch release run](https://github.com/nodeselector/actions-release-demo/actions/runs/33664342318) |
| Exact-tag `$/` execution | [RC validation run](https://github.com/nodeselector/actions-release-demo/actions/runs/33664386924) |
| Immutable candidate | [`v0.1.1-rc.1`](https://github.com/nodeselector/actions-release-demo/releases/tag/v0.1.1-rc.1) |
| Immutable stable release | [`v0.1.1`](https://github.com/nodeselector/actions-release-demo/releases/tag/v0.1.1) |
| Server-enforced tag protection | [Immutable release tag ruleset](https://github.com/nodeselector/actions-release-demo/rules/22130912) |
