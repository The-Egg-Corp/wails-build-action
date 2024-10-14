# wails-build-action
> [!NOTE]
> This is a fork of [dAppServer/wails-build-action](https://github.com/marketplace/actions/wails-build-action).

GitHub Action to build a [Wails](https://wails.io) v2 project.\
By default, the action will build and upload assets to the workflow, on a tagged build it will also upload them to the release.

## Changelog
### v1.4
- Renamed the `use-bun` option to `setup-bun`.
- Fixed wrong description of the `build` option.
- Cleaned up the output name of build assets.
  - Prev: `Wails Build Linux <build-name>`. Now: `<build-name> (Linux)` 

### v1.3
- Bumped `import-codesign-certs`. **v1** -> **v3**.

### v1.2
- Added support for Bun setup. Defaults to `false`.
- Changed `nsis` option so that it *must* be specified.
- Bumped `upload-artifact`. **v3** -> **v4**.
- Bumped `setup-go`. **v4** -> **v5**.
- Bumped `setup-node`. **v3** -> **v4**.
- Bumped `node-version` from **18.x** to **20**. This should stop action warnings.

## Default Build
```yaml
- uses: The-Egg-Corp/wails-build-action@v1.4
  with:
    build-name: wailsApp
    build-platform: linux/amd64
    nsis: false # Set to true to create an installer.
```

## Build without uploading
```yaml
- uses: The-Egg-Corp/wails-build-action@v1.4
  with:
    build-name: wailsApp
    build-platform: linux/amd64
    nsis: false
    package: false
```

## GitHub Action Options
| Name                                 | Default                  | Description                                                |
|--------------------------------------|--------------------------|------------------------------------------------------------|
| `build-name`                         | (Required)               | The name of the binary                                     |
| `build`                              | `true`                   | Runs `wails build` on your source                          |
| `nsis`                               | (Required)               | Runs `wails build` with `-nsis` to create an installer     |
| `sign`                               | `false`                  | After build, signs and creates signed installers           |
| `package`                            | `true`                   | Upload workflow artifacts & publish release on tag         |
| `build-platform`                     | `darwin/universal`       | Platform to build for                                      |
| `wails-version`                      | `latest`                 | Wails version to use                                       |
| `wails-build-webview2`               | `download`               | WebView2 installer method [download, embed, browser, error]|
| `go-version`                         | `^1.22`                  | Go version to use                                          |
| `node-version`                       | `20`                     | NodeJS version to use                                      |
| `bun-setup`                          | `false`                  | Whether to setup Bun                                       |
| `bun-version`                        | `latest`                 | Bun version to use                                         |
| `deno-build`                         | ""                       | Deno compile command                                       |
| `deno-working-directory`             | `.`                      | Working directory of your [Deno](https://deno.land/) server|
| `deno-version`                       | `v1.x`                   | Deno version to use                                        |
| `sign-macos-app-id`                  | ""                       | MacOS Application Certificate ID                           |
| `sign-macos-apple-password`          | ""                       | MacOS Apple Password                                       |
| `sign-macos-app-cert`                | ""                       | MacOS Application Certificate                              |
| `sign-macos-app-cert-password`       | ""                       | MacOS Application Certificate Password                     |
| `sign-macos-installer-id`            | ""                       | MacOS Installer Certificate ID                             |
| `sign-macos-installer-cert`          | ""                       | MacOS Installer Certificate                                |
| `sign-macos-installer-cert-password` | ""                       | MacOS Installer Certificate Password                       |
| `sign-windows-cert`                  | ""                       | Windows Signing Certificate                                |
| `sign-windows-cert-password`         | ""                       | Windows Signing Certificate Password                       |

## Example Build
This example uses a manual trigger, you may want to use `on: [push, pull_request]` instead.

```yaml
name: Wails Build

on:
  # Allows workflow to be manually triggered.
  workflow_dispatch:
    inputs:
      logLevel:
        description: "Log level"
        required: true
        type: choice
        options: [info, warning, debug]
        default: "warning"
      nsis:
        description: "Create installer"
        type: boolean
        required: true
        default: false
      package: 
        description: "Upload artifacts"
        type: boolean
        required: false
        default: true

jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        build: [
          {name: "App", platform: linux/amd64, os: ubuntu-latest},
          {name: "App", platform: windows/amd64, os: windows-latest},
          {name: "App", platform: darwin/universal, os: macos-latest}
        ]

    runs-on: ${{ matrix.build.os }}
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - uses: The-Egg-Corp/wails-build-action@v1.4
        with:
          build-name: ${{ matrix.build.name }}
          build-platform: ${{ matrix.build.platform }}
```

## MacOS Code Signing
You need to make two gon configuration files, this is because we need to sign and notarize the .app before making an installer with it.

```yaml
  - uses: The-Egg-Corp/wails-build-action@v1.4
    with:
      build-name: wailsApp
      sign: true
      build-platform: darwin/universal
      sign-macos-apple-password: ${{ secrets.APPLE_PASSWORD }}
      sign-macos-app-id: ${{ secrets.MACOS_DEVELOPER_CERT_ID }}
      sign-macos-app-cert: ${{ secrets.MACOS_DEVELOPER_CERT }}
      sign-macos-app-cert-password: ${{ secrets.MACOS_DEVELOPER_CERT_PASSWORD }}
      sign-macos-installer-id: ${{ secrets.MACOS_INSTALLER_CERT_ID }}
      sign-macos-installer-cert: ${{ secrets.MACOS_INSTALLER_CERT }}
      sign-macos-installer-cert-password: ${{ secrets.MACOS_INSTALLER_CERT_PASSWORD }}
```

`build/darwin/gon-sign.json`
```json
{
  "source": ["./build/bin/wailsApp.app"],
  "bundle_id": "com.wails.app",
  "apple_id": {
    "username": "username",
    "password": "@env:APPLE_PASSWORD"
  },
  "sign": {
    "application_identity": "Developer ID Application: XXXXXXXX (XXXXXX)",
    "entitlements_file": "./build/darwin/entitlements.plist"
  },
  "dmg": {
    "output_path": "./build/bin/wailsApp.dmg",
    "volume_name": "Lethean"
  }
}
```

`build/darwin/gon-notarize.json`
```json
{
  "notarize": [{
    "path": "./build/bin/wailsApp.pkg",
    "bundle_id": "com.wails.app",
    "staple": true
  },{
    "path": "./build/bin/wailsApp.app.zip",
    "bundle_id": "com.wails.app",
    "staple": false
  }],
  "apple_id": {
    "username": "USER name",
    "password": "@env:APPLE_PASSWORD"
  }
}
```

`build/darwin/entitlements.plist`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.app-sandbox</key>
  <true/>
  <key>com.apple.security.network.client</key>
  <true/>
  <key>com.apple.security.network.server</key>
  <true/>
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
</dict>
</plist>
```
