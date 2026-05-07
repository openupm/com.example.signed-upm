# Signed UPM Example

This repository demonstrates a minimal Unity Package Manager package that is
packed and signed in GitHub Actions, then published as a GitHub Release asset.
It is intentionally small so package authors can copy the workflow into their
own repositories without carrying unrelated project structure.

Unity 6.3 provides the UPM CLI, a command-line tool for package operations such
as packing and signing. See the
[Unity UPM CLI documentation](https://docs.unity3d.com/6000.3/Documentation/Manual/upm-cli.html)
for installation and command details.

The `upm pack` command creates a `.tgz` archive from the package folder and
signs it with a Unity organization through service account credentials. A
signed UPM package contains `package/.attestation.p7m` inside the archive. The
resulting `.tgz` file can be published to a registry such as OpenUPM.

## Package Layout

- `package/package.json` is the Unity package manifest.
- `.github/workflows/ci.yml` signs the package only when a tag is pushed.

The package has no runtime code. It exists only to demonstrate release
automation for signed UPM tarballs.

## GitHub Secrets

Create a Unity service account with package signing permission for the
organization that should sign the package. Add these GitHub Actions secrets to
the repository:

- `UPM_SERVICE_ACCOUNT_KEY_ID`
- `UPM_SERVICE_ACCOUNT_KEY_SECRET`
- `UPM_ORG_ID`

For a single package, repository secrets are enough. If several repositories in
the same GitHub organization sign packages for the same Unity organization,
prefer GitHub organization secrets scoped to only the repositories that need
them. That keeps credential rotation centralized while avoiding broad access.

## Release Flow

Update `package/package.json`, commit the change, then push a version tag:

```sh
git tag 1.0.0
git push origin main 1.0.0
```

The workflow only runs for pushed git tags. For tag `1.0.0`, it creates a
GitHub Release with the same tag name, installs Unity UPM CLI, and runs
`upm pack ./package` to create a signed UPM `.tgz` file. The signed archive
contains `package/.attestation.p7m` for the package signature. The workflow
also verifies that the archive contains `package/package.json` for
`com.example.signed-upm@1.0.0`, then attaches the signed tarball to the release.

The signed package is uploaded as a GitHub Release asset, not as a GitHub
Actions workflow artifact. Workflow artifacts and logs have retention periods,
but release assets remain attached to the release until the asset or release is
deleted. Keep release assets available so OpenUPM can process older package
versions later.

## OpenUPM

To publish a signed GitHub Release asset through OpenUPM, submit package
metadata with `trackingMode: githubRelease`:

```yaml
trackingMode: githubRelease
```

When a release has only one `.tgz` or `.tar.gz` asset, OpenUPM selects it
automatically. Set `githubReleaseAssetName` only when a release has multiple
assets. The value can be either the exact signed package filename or a stable
filename prefix when the filename contains the version string.

OpenUPM downloads the public release asset instead of packing from the git
checkout.
