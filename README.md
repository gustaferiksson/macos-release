# macos-release

Reusable GitHub Actions workflow that releases a signed, notarized macOS app:
import the Developer ID certificate, build, sign, notarize, staple, zip, create
the GitHub Release, and bump the Homebrew cask in
[gustaferiksson/homebrew-tap](https://github.com/gustaferiksson/homebrew-tap).

## Use it

```yaml
name: Release
on:
  push:
    tags: [ 'v*' ]
permissions:
  contents: write
concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: false
jobs:
  release:
    uses: gustaferiksson/macos-release/.github/workflows/release.yml@main
    secrets: inherit
    with:
      name: MonoText
      app: .build/MonoText.app
      cask: monotext
      runner: macos-26
      build: |
        swift build -c release
        ./bundle.sh release
```

`permissions: contents: write` must be on the caller — a called workflow cannot
escalate its own permissions.

## Inputs

| Input | Meaning |
| --- | --- |
| `name` | Display name: the release title and the zip prefix (`MonoText-1.2.3.zip`). |
| `app` | Path to the built `.app`, relative to the repo root. |
| `cask` | Cask file name in the tap's `Casks/`, without `.rb`. |
| `runner` | Runner image label, e.g. `macos-26`. |
| `build` | Shell script producing `app`. `$VERSION`, `$BUILD` and `$SIGN_ID` are set. |
| `signed_by_build` | Set when `build` already signs the app, to skip the codesign step. |

## Secrets

Required on every calling repo, and passed with `secrets: inherit`:

| Secret | What it is |
| --- | --- |
| `APPLE_CERT_P12` | base64 of the exported *Developer ID Application* cert (`.p12`). |
| `APPLE_CERT_PASSWORD` | Password used when exporting that `.p12`. |
| `NOTARY_APPLE_ID` | Apple ID email for `notarytool`. |
| `NOTARY_PASSWORD` | App-specific password for that Apple ID. |
| `TAP_GITHUB_TOKEN` | Token with write access to `gustaferiksson/homebrew-tap`. |

The preflight step fails the run if any of them is empty, so a release is never
silently ad-hoc signed or left un-bumped.
