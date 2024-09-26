# wails-build-action
> [!NOTE]
> This is a fork of [dAppServer/wails-build-action@v2](https://github.com/marketplace/actions/wails-build-action) and is currently not on the marketplace.

GitHub action to build a [Wails](https://wails.io) v2 project.
By default, the action will build and upload the results to github, on a tagged build it will also upload to the release.

### Changelog
- Added support for Bun setup. Defaults to `false`.
- Changed `nsis` to be required.
- Added `node-uses` option to customize which action to use for Node setup.
- Bumped `upload-artifact`. **v3** -> **@v4**.
- Bumped `setup-go`. **v4** -> **v5**.
- Bumped `setup-node`. **v3** -> **v4**.
- Bumped `node-version` from **18.x** to **20**. This should stop action warnings.

# Default build
```yaml
- uses: dAppServer/wails-build-action@v2.2
  with:
    build-name: wailsApp
    build-platform: linux/amd64
```

## Build with No uploading
```yaml
- uses: dAppServer/wails-build-action@v2.2
  with:
    build-name: wailsApp
    build-platform: linux/amd64
    package: false
```

## GitHub Action Options

| Name                                 | Default                  | Description                                                |
|--------------------------------------|--------------------------|------------------------------------------------------------|
| `build-name`                         | none, required input     | The name of the binary                                     |
| `build`                              | `true`                   | Runs `wails build` on your source                          |
| `nsis`                               | `false`                  | Runs `wails build` with `-nsis` to create an installer     |
| `sign`                               | `false`                  | After build, signs and creates signed installers           |
| `package`                            | `true`                   | Upload workflow artifacts & publish release on tag         |
| `build-platform`                     | `darwin/universal`       | Platform to build for                                      |
| `wails-version`                      | `latest`                 | Wails version to use                                       |
| `wails-build-webview2`               | `download`               | WebView2 installer method [download, embed, browser, error]|
| `go-version`                         | `^1.22`                  | Go version to use                                          |
| `node-uses`                          | `@actions/setup-node@v4` | Action to use for Node setup                               |
| `node-version`                       | `20`                     | NodeJS version to use                                      |
| `bun-setup`                          | `false`                  | Whether to setup Bun                                       |
| `bun-version`                        | `20`                     | Bun version to use                                         |
| `deno-build`                         | ``                       | Deno compile command                                       |
| `deno-working-directory`             | `.`                      | Working directory of your [Deno](https://deno.land/) server|
| `deno-version`                       | `v1.20.x`                | Deno version to use                                        |
| `sign-macos-app-id`                  | ''                       | ID of the app signing cert                                 |
| `sign-macos-apple-password`          | ''                       | MacOS Apple password                                       |
| `sign-macos-app-cert`                | ''                       | MacOS Application Certificate                              |
| `sign-macos-app-cert-password`       | ''                       | MacOS Application Certificate Password                     |
| `sign-macos-installer-id`            | ''                       | MacOS Installer Certificate ID                             |
| `sign-macos-installer-cert`          | ''                       | MacOS Installer Certificate                                |
| `sign-macos-installer-cert-password` | ''                       | MacOS Installer Certificate Password                       |
| `sign-windows-cert`                  | ''                       | Windows Signing Certificate                                |
| `sign-windows-cert-password`         | ''                       | Windows Signing Certificate Password                       |


## Example Build

```yaml
name: Wails build

on: [push, pull_request]

jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        build: [
          {name: 'App', platform: linux/amd64, os: ubuntu-latest},
          {name: 'App', platform: windows/amd64, os: windows-latest},
          {name: 'App', platform: darwin/universal, os: macos-latest}
        ]
    runs-on: ${{ matrix.build.os }}
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: recursive
      - uses: dAppServer/wails-build-action@v2.2
        with:
          build-name: ${{ matrix.build.name }}
          build-platform: ${{ matrix.build.platform }}
```

## MacOS Code Signing
You need to make two gon configuration files, this is because we need to sign and notarize the .app before making an installer with it.

```yaml
  - uses: dAppServer/wails-build-action@v2.1
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
  "source" : ["./build/bin/wailsApp.app"],
  "bundle_id" : "com.wails.app",
  "apple_id": {
    "username": "username",
    "password": "@env:APPLE_PASSWORD"
  },
  "sign" :{
    "application_identity" : "Developer ID Application: XXXXXXXX (XXXXXX)",
    "entitlements_file": "./build/darwin/entitlements.plist"
  },
  "dmg" :{
    "output_path":  "./build/bin/wailsApp.dmg",
    "volume_name":  "Lethean"
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
