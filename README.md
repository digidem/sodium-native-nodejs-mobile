# sodium-native-nodejs-mobile

[NodeJS Mobile](https://github.com/nodejs-mobile/nodejs-mobile) prebuilds for
[`sodium-native`](https://github.com/holepunchto/sodium-native)

## Working locally

### Requirements

- Node 24 (see [`.nvmrc`](.nvmrc))
- Android NDK (CI uses version 27.2.12479018)
  - (optional) exported `ANDROID_NDK_HOME` environment variable
- Xcode, for the iOS targets

### General steps

Should be clear enough to follow the [prebuild action steps][prebuild-action]
but in summary:

1. Download the npm tarball package and unzip e.g.
   ```
   npm pack sodium-native@latest | xargs tar -zxvf
   ```
2. Navigate to unzipped directory:
   ```
   cd package
   ```
3. Install dependencies:
   ```
   npm install --ignore-scripts
   ```
4. Install
   [patched `cmake-napi`](https://github.com/digidem/cmake-napi-nodejs-mobile):
   ```
   npm install cmake-napi@github:digidem/cmake-napi-nodejs-mobile
   ```
5. Install [bare-make](https://github.com/holepunchto/bare-make) globally:
   ```
   npm install -g bare-make@latest
   ```
6. Generate, build and install:
   ```
   bare-make generate --platform android --arch arm64
   bare-make build
   bare-make install
   ```

Note that step 6 is the short version. CI passes extra flags per platform that
matter for the artifacts actually shipping — a 16 KB max page size, an
`android-24` target so the linker doesn't emit RELR relocations that Android <
9 can't load, the NDK's libc++ headers (`bare-make` leaves `ANDROID_STL=none`),
and `APPLE_CLANG=ON` on iOS. The [prebuild action][prebuild-action] is the
source of truth for these; each has a comment explaining why.

## Creating a release

1. Navigate to the [Build, test, and release prebuilds workflow][workflow]
2. Manually dispatch the workflow with the version you want to build, ensuring
   that "Publish release" is checked.

The build runs every target, then tests the result on an Android emulator (at
API 30, and again at API 24 — the oldest level the prebuilds must load on) and
on an iOS simulator, before anything is published. Unchecking "Publish release"
runs the same build and tests without creating a GitHub Release, which is
useful for verifying a version before shipping it.

## Contributing

We welcome contributions to this repository. If you have an idea for a new
feature or have found a bug, please open an issue or submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file
for more details.

`sodium-native` itself is licensed under MIT by Holepunch; this
repository only packages prebuilds of it.

[prebuild-action]:
  https://github.com/digidem/nodejs-mobile-bare-prebuilds/blob/main/.github/actions/prebuild/action.yml
[workflow]:
  https://github.com/digidem/sodium-native-nodejs-mobile/actions/workflows/prebuilds.yml
