# Adobe DNG SDK 1.7.1 — `dng_validate`

> I need a prebuilt `dng_validate` command-line tool for macOS and Windows, but Adobe only provides the source code of DNG SDK. This repo provides binaries built from the source code using GitHub Actions.

This repo vendors Adobe DNG SDK 1.7.1 and builds the `dng_validate` command-line tool for macOS (universal) and Windows (x64) via GitHub Actions. 

Download the prebuilt binaries from the Releases page.

## Usage

```bash
dng_validate photo.dng
```

or

```bash
dng_validate -v photo.dng

dng_validate -tif out photo.dng      # generate out.tif
dng_validate -dng out photo.dng      # generate out.dng

dng_validate -csP3 -16 -tif out photo.dng

# image in different stages
dng_validate -1 s1 -2 s2 -3 s3 photo.dng
```

## Modification

Please note that this repository contains modifications to the official Adobe DNG SDK 1.7.1 distribution files for successful build of `dng_validate` on Windows.

1. `dng_sdk/projects/win/dng_validate/dng_validate.vcxproj`: Remove the `ProjectReference` pointing to `libwebp/client_projects/win/webp.vcxproj`.

2. `libjxl/client_projects/win/highway.vcxproj`: Remove the `<ClInclude>` reference to `hwy/contrib/sort/disabled_targets.h`.

3. `xmp/toolkit/XMPCore/build/CMake64Static_VC16/XMPCoreStatic.vcxproj` and `xmp/toolkit/XMPFiles/build/CMake64Static_VC16/XMPFilesStatic.vcxproj`: Add `<PlatformToolset>v143</PlatformToolset>`.

These modifications should have no effect on the functionality of `dng_validate` and are only necessary for building the tool on Windows.

## License

- DNG SDK: [`LICENSE.txt`](LICENSE.txt)
- libjpeg: [`libjpeg/README`](libjpeg/README)
- libjxl / brotli / highway: `libjxl/libjxl/LICENSE`, `libjxl/libjxl/third_party/brotli/LICENSE`,
  `libjxl/libjxl/third_party/highway/LICENSE`
