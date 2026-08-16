# sodium-native-nodejs-mobile

[NodeJS Mobile](https://github.com/nodejs-mobile/nodejs-mobile) prebuilds for [`sodium-native`](https://github.com/holepunchto/sodium-native)

## Working locally

### Requirements

- Node 18
- Android NDK (CI uses version 27.2.12479018)
  - (optional) exported `ANDROID_NDK_HOME` environment variable

### General steps

Should be clear enough to follow the [reusable workflow steps](https://github.com/digidem/nodejs-mobile-bare-prebuilds/blob/main/.github/workflows/prebuild.yml) but in summary:

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
   npm install
   ```
4. Install [patched `cmake-napi`](https://github.com/digidem/cmake-napi-nodejs-mobile):
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

## Patches

Files in `patches/` follow the [`patch-package`](https://github.com/ds300/patch-package) naming convention — `sodium-native+<version>.patch` — and the build workflow applies the one matching the version being built with `patch -p1`, right after unpacking the npm tarball and before installing dependencies. To reproduce a build locally, apply the matching patch from inside the unzipped `package` directory between steps 2 and 3 above. Note that the workflow fails if `patches/` holds patches for other versions but none for the version being built, so bumping the version means refreshing the patch too.

`sodium-native+5.1.0.patch` adds `extensions/snm-exports.c`, five tiny wrapper functions (`snm_sodium_init`, `snm_crypto_pwhash`, and the three pwhash parameter getters) that let host app code call libsodium's password hashing directly via `dlopen`/`dlsym`. They are needed because libsodium is compiled with hidden symbol visibility, so `crypto_pwhash` itself is not exported from the built addon; the wrappers live alongside sodium-native's own `sn__extension_*` helpers, which are exported the same way. CoMapeo uses this to derive a key natively with the exact same libsodium binary that the embedded Node.js runtime uses.

## Creating a release

1. Navigate to the [Generate Prebuilds workflow](https://github.com/digidem/sodium-native-nodejs-mobile/actions/workflows/prebuilds.yml)
2. Manually dispatch the worflow with the version you want to build, ensuring that "Publish Release" is checked.

## Contributing

We welcome contributions to this repository. If you have an idea for a new feature or have found a bug, please open an issue or submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
