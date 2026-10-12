由 GitHub Actions 使用 Adobe 官方工程文件自动构建。

## 包含内容

| 平台 | 文件 | 说明 |
|---|---|---|
| macOS | `dng_validate-macos-universal.zip` | 通用二进制（arm64 + x86_64），部署目标 macOS 11.0+ |
| Windows | `dng_validate-windows-x64.zip` | x64，静态链接 C 运行时（`/MT`），无需 VC++ Redistributable |
| 校验 | `SHA256SUMS.txt` | 上述 zip 的 SHA-256 |

每个 zip 内含可执行文件、`LICENSE.txt`、`README.md` 与 `VERSION.txt`。

## 用法

```bash
dng_validate photo.dng                  # 校验并打印元数据
dng_validate -v photo.dng               # 详细输出
dng_validate -tif out photo.dng         # 转 TIFF
dng_validate -dng out photo.dng         # 转 DNG
dng_validate -csP3 -16 -tif out photo.dng
```

不带参数运行会打印完整选项列表。退出码 `0` 表示校验完成。

## 版本

- DNG SDK **1.7.1**，build **2724**
- macOS：Xcode 构建，`ARCHS_STANDARD_64_BIT` + `ONLY_ACTIVE_ARCH=NO` → 通用二进制
- Windows：MSVC 2022 + ClangCL（dng_validate/libjxl/highway/brotli）+ 默认 MSVC 工具集（XMP）

## 内容来源

本 Release 由仓库源码构建，源码为 Adobe DNG SDK 1.7.1 官方分发包，仅做了 2 处工程文件修复
（移除指向不存在的 `libwebp` 工程的悬空引用、移除已被裁剪的 `disabled_targets.h` 文件树引用），
未改动任何 SDK 源码逻辑。详见仓库 README。
