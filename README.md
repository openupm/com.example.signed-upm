# Signed UPM Example

This repository demonstrates a minimal Unity Package Manager package that is
packed and signed in GitHub Actions, then published as a GitHub Release asset.
It is intentionally small so package authors can copy the workflow into their
own repositories without carrying unrelated project structure.

Unity 6.3 checks digital signatures on tarball packages. The `upm pack`
command creates a `.tgz` archive from the package folder and signs it with a
Unity organization through service account credentials. The signed tarball can
then be distributed directly, uploaded to a release, or submitted to a registry
workflow that consumes release assets.

## Package Layout

- `package/package.json` is the Unity package manifest.
- `package/package.json.meta` is the Unity meta file for the manifest.
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

The workflow installs Unity UPM CLI, runs `upm pack ./package`, verifies that
the archive contains `package/package.json` for `com.example.signed-upm@1.0.0`,
and attaches the signed tarball to the matching GitHub Release.

## OpenUPM

To publish a signed GitHub Release asset through OpenUPM, submit the package
metadata with `trackingMode: githubRelease` and set `githubReleaseAssetName` to
the signed archive filename, for example:

```yaml
trackingMode: githubRelease
githubReleaseAssetName: com.example.signed-upm-1.0.0.tgz
```

OpenUPM will download the public release asset instead of packing from the git
checkout.
