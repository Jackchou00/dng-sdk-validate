# Adobe DNG SDK 1.7.1 — `dng_validate`

This repo vendors Adobe DNG SDK 1.7.1 and builds the `dng_validate` command-line tool for macOS (universal) and Windows (x64) via GitHub Actions. 

Download the prebuilt binaries from the Releases page.

## Usage

```bash
dng_validate photo.dng
```

or

```bash
dng_validate -v photo.dng

dng_validate -tif out photo.dng      # 生成 out.tif
dng_validate -dng out photo.dng      # 生成 out.dng

dng_validate -csP3 -16 -tif out photo.dng

# image in different stages.
dng_validate -1 s1 -2 s2 -3 s3 photo.dng
```

常用选项（完整列表直接运行不带参数的可执行文件即可看到）：

| 选项 | 含义 |
|---|---|
| `-v` | 详细模式 |
| `-d <num>` | dump 行数上限（隐含 `-v`） |
| `-tif <file>` / `-dng <file>` | 输出 TIF / DNG |
| `-1/-2/-3 <file>` | 输出第 1/2/3 阶段图像 |
| `-transparency <file>` / `-depthMap <file>` | 输出透明度蒙版 / 深度图 |
| `-16` / `-32` | 16 位 / 32 位浮点输出 |
| `-cs1`…`-cs6`、`-csP3`、`-cs2020` | 输出色彩空间（sRGB / Adobe RGB / ProPhoto / ColorMatch / Gray 1.8 / Gray 2.2 / Display P3 / Rec.2020） |
| `-profile <name>` | 指定相机配置文件 |
| `-b4` / `-s <num>` | 四色 Bayer 插值 / 多采样 CFA 取样 |
| `-size` / `-min` / `-max` | 预览图尺寸控制 |
| `-proxy <num>` | 生成 DNG 代理文件 |
| `-lossyMosaicJXL` / `-losslessJXL` | 以 JXL 压缩写 DNG |

退出码：`0` 表示校验完成，非 `0`（通常 ≥100）表示出错，具体值对应 SDK 的 `dng_error_code`。
不带参数运行会打印版本与用法，退出码为 `1`。

---

## 相对 Adobe 官方分发包的改动

本仓库对官方 1.7.1 分发包做了 **3 组构建文件（工程文件）修改**，未改动任何 SDK 源码逻辑：

1. **`dng_sdk/projects/win/dng_validate/dng_validate.vcxproj`**
   删除指向 `libwebp/client_projects/win/webp.vcxproj` 的 `ProjectReference`。
   该文件不包含在本 SDK 分发包中（解决方案里也没有它），保留会导致 MSBuild 直接报
   `MSB3202: The project file was not found`；而 `dng_validate` 本身不引用任何 webp 符号。

2. **`libjxl/client_projects/win/highway.vcxproj`**
   删除对 `hwy/contrib/sort/disabled_targets.h` 的 `<ClInclude>` 引用。
   该头文件在上游 libjxl 裁剪集成时已被移除，且没有任何源码 `#include` 它（仅 IDE 文件树引用）。

3. **`xmp/toolkit/XMPCore/build/CMake64Static_VC16/XMPCoreStatic.vcxproj`**
   与 **`xmp/toolkit/XMPFiles/build/CMake64Static_VC16/XMPFilesStatic.vcxproj`**
   显式加上 `<PlatformToolset>v143</PlatformToolset>`。
   这两个 CMake 生成的工程原本没有指定工具集，只依赖 MSBuild 的默认值
   `$(DefaultPlatformToolset)`；在 VS IDE 里没问题，但**直接在命令行跑 `msbuild`** 时
   `VisualStudioVersion` 为空，MSBuild 会回退到 VS2010（v100）并报
   `MSB8020: The build tools for Visual Studio 2010 (Platform Toolset = 'v100') cannot be found`。
   （`dng_validate`/`libjxl`/`highway`/`brotli` 因为显式写了 `ClangCL`，不受此问题影响。）
   若你用 VS2019，可在命令行覆盖：`-p:PlatformToolset=v142`。

---

## 自行构建

两个平台的构建都**不需要联网下载任何第三方依赖**（XMP SDK、libjxl、brotli、highway、skcms、
expat、zlib、uuid 均已随仓库提供）。

### macOS

```bash
# 工程没有共享 scheme，只有 target；configuration 名为 Default
xcodebuild \
  -project dng_sdk/projects/mac/dng_validate.xcodeproj \
  -target "dng_validate release" \
  -configuration Default \
  build
# 产物：dng_sdk/targets/mac/release64/dng_validate （通用二进制 arm64 + x86_64）
```

需要 Xcode 12.4 及以上（官方说明实测至 Xcode 26）。

### Windows

需要 Visual Studio 2022 **并勾选 “适用于 Windows 的 C++ Clang 工具”**
（`Microsoft.VisualStudio.Component.VC.Llvm.ClangToolset`；dng_validate / libjxl / highway / brotli
工程使用 `ClangCL` 工具集，XMP 工程用默认 MSVC 工具集）。

```bat
:: 在 VS2022 的 “Developer Command Prompt / x64 Native Tools” 里执行，
:: 或者普通命令行 + 显式指定 VS 版本（XMP 工程已在本仓库里写死 v143，
:: 若是 VS2019 请改成 -p:PlatformToolset=v142）
msbuild dng_sdk\projects\win\dng_validate.sln -m -p:Configuration="Validate Release" -p:Platform=x64
:: 产物：dng_sdk\targets\win\release64_x64\dng_validate.exe
```

### 本地冒烟测试

```bash
./dng_validate sample_files/05_PGTM2_unsigned8.dng        # 期望输出 Validation complete
./dng_validate sample_files/02_jxl_linear_raw_float.dng   # JXL 压缩 DNG，验证 libjxl 正常
```

---

## 版本信息

| 项目 | 值 |
|---|---|
| DNG SDK | 1.7.1 |
| Build | 2724（2026-09-08） |
| C++ 标准 | C++17 |
| macOS 部署目标 | 11.0 |
| Windows 运行时 | 静态 `/MT` |

---

## 许可证

本仓库分发的是 Adobe Inc. 的 DNG SDK 及其附带的第三方库，请遵守各自的许可条款：

- **DNG SDK / 文档**：见 [`LICENSE.txt`](LICENSE.txt)（允许再分发，要求保留版权声明）
- **libjpeg**：见 [`libjpeg/README`](libjpeg/README)
- **libjxl / brotli / highway**：见 `libjxl/libjxl/LICENSE`、`libjxl/libjxl/third_party/brotli/LICENSE`、
  `libjxl/libjxl/third_party/highway/LICENSE`
- **Adobe XMP SDK / expat / zlib / uuid**：见各自目录下的声明文件与源文件头部

本仓库仅提供构建脚本与预编译产物，与 Adobe Inc. 无隶属关系。
