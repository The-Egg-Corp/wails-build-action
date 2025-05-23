## v2.0
- Added `name` option to specify the clean app name. Extensions should be included in `build-name`.
  - This is now **required** as it is used instead of `build-name` in cases like outputting the artifact.
- Included `libwebkit-4.1-dev` in Linux dependency install step.
  - This should fix "**libwebkit not found**" issue when building with newer Linux versions like `ubuntu-latest`.
- Changed default value of `build-platform` option. `darwin/universal` -> `windows-latest`.

## v1.4
- Renamed the `use-bun` option to `setup-bun`.
- Fixed wrong description of the `build` option.
- Cleaned up the output name of build assets.
  - Prev: `Wails Build Linux <build-name>`. Now: `<build-name> (Linux)` 

## v1.3
- Bumped `import-codesign-certs`. **v1** -> **v3**.

## v1.2
- Added support for Bun setup. Defaults to `false`.
- Changed `nsis` option so that it *must* be specified.
- Bumped `upload-artifact`. **v3** -> **v4**.
- Bumped `setup-go`. **v4** -> **v5**.
- Bumped `setup-node`. **v3** -> **v4**.
- Bumped `node-version` from **18.x** to **20**. This should stop action warnings.