# Gyroflow 编译指南

本文档提供各平台编译 Gyroflow 的完整流程，包括依赖安装、常见问题及解决方案。

## 目录

- [Windows](#windows)
  - [环境准备](#windows-环境准备)
  - [依赖安装](#windows-依赖安装)
  - [环境变量配置](#windows-环境变量配置)
  - [编译运行](#windows-编译运行)
  - [常见问题](#windows-常见问题)
- [Linux](#linux)
- [Android](#android)
  - [环境准备](#android-环境准备)
  - [环境变量配置](#android-环境变量配置)
  - [编译配置](#android-编译配置)
  - [编译运行](#android-编译运行)
  - [常见问题](#android-常见问题)

---

## Windows

### Windows 环境准备

#### 1. 安装 Rust

```powershell
# 使用 rustup 安装（推荐）
winget install Rustlang.Rustup

# 或从 https://rustup.rs/ 下载安装
```

下载完双击可执行文件按提示安装rust

#### 2. 安装必备工具（已安装可跳过）

```powershell
# Git
winget install Git.Git

# 7-Zip（用于解压 FFmpeg）
winget install 7zip.7zip

# just 命令运行器（可选，但有兼容性问题）
cargo install just
```

---

### Windows 依赖安装

#### just脚本自动安装

使用powershell cd到gyroflow工程根目录执行：

```powershell
just install-deps
```

> **⚠️ 重要提示：在powershell中执行`just install-deps`命令时会导致如下错误**：
> ```
> 表达式或语句中包含意外的标记"}"。
> error: Recipe `install-deps` failed with exit code 1
> ```
> 
> **原因分析**：
> - `just` 工具在 Windows 上将 recipe 生成为临时 `.ps1` 脚本执行
> - 脚本中的多行 PowerShell 命令在特定环境下换行符处理有问题
> - 这不是代码问题，而是 just 工具本身的兼容性问题
> 
> **解决方案**：
> - **方法1（推荐）**：手动安装依赖（见下文 "Windows 依赖安装" 章节），使用 `windows.just` 中已配置好的本地路径
> - **方法2**：不使用 just，直接用 `cargo` 命令编译
> - **方法3**：修复 `_scripts/windows.just` 中的 `install-deps` recipe，使用单行 PowerShell 命令

#### 手动安装

##### Python

直接在python官网下载最新版安装程序并安装即可

##### Qt

Gyroflow 需要 Qt 6.4+。

**使用 aqtinstall 安装：**

```powershell
pip install aqtinstall===3.2.0

# 安装 Qt 6.7.3 (或 6.4.3)
aqt install-qt windows desktop 6.7.3 win64_msvc2019_64 -m qtshadertools

# 默认安装到 C:\Users\<用户名>\AppData\Local\Qt
# 或指定目录：
aqt install-qt windows desktop 6.7.3 win64_msvc2019_64 -O D:\ProgramData\Qt -m qtshadertools
```

> **注意**：Qt 安装路径不能包含空格。

---

##### FFmpeg

Gyroflow 需要特定版本：`ffmpeg-8.1-windows-desktop-vs2026-gpl-lite`

**下载地址：**
https://sourceforge.net/projects/avbuild/files/windows-desktop/

或从项目官方渠道获取预编译版本。

**解压到目录**，例如：
```
D:\ProgramData\FFmpeg\ffmpeg-8.1-windows-desktop-vs2026-gpl-lite
```

目录结构应包含：
```
ffmpeg-8.1-windows-desktop-vs2026-gpl-lite/
├── bin/
│   ├── avcodec-*.dll
│   ├── avformat-*.dll
│   └── ...
├── lib/
│   ├── avcodec.lib
│   ├── avformat.lib
│   └── ...
├── include/
│   ├── libavcodec/
│   ├── libavformat/
│   └── ...
└── lib/pkgconfig/
    └── *.pc files
```

---

##### OpenCV

**使用 vcpkg 安装：**

```powershell
# 克隆 vcpkg（如果尚未安装）
cd E:\code\research\3rdparty  # 或其他目录
git clone --depth 1 https://github.com/Microsoft/vcpkg.git

# 引导 vcpkg
.\vcpkg\bootstrap-vcpkg.bat

# 安装 OpenCV（Release 版本）
.\vcpkg\vcpkg install "opencv4[core,calib3d,opencl,thread,intrinsics]:x64-windows-release" --cmake-args=-DVCPKG_BUILD_TYPE=release

# 安装 OpenCL（如果需要）
.\vcpkg\vcpkg install "opencl:x64-windows-release" --cmake-args=-DVCPKG_BUILD_TYPE=release
```

> **注意**：使用 `x64-windows-release` triplet，而非默认的 `x64-windows`，以避免 Debug/Release 混用问题。

---

##### LLVM

LLVM 是 `bindgen` 的必需依赖，用于生成 C/C++ 库的 Rust 绑定。

**下载安装：**

```powershell
cd E:\code\research\3rdparty\gyroflow\ext

# 下载 LLVM 19.1.7
wget "https://github.com/llvm/llvm-project/releases/download/llvmorg-19.1.7/LLVM-19.1.7-win64.exe" -outfile "llvm.exe"

# 解压（不需要完整安装）
7z x -y llvm.exe -ollvm

# 清理
del llvm.exe
```

LLVM 路径：`E:\code\research\3rdparty\gyroflow\ext\llvm\bin`

也可以在[github下载页面](https://github.com/llvm/llvm-project/releases/)下载最新的exe(如LLVM-22.1.2-win64.exe)，然后手动安装

---

### Windows 环境变量配置

编译需要正确设置以下环境变量：

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `QMAKE` | Qt qmake 可执行文件路径 | `D:/ProgramData/Qt/6.7.3/msvc2019_64/bin/qmake.exe` |
| `LIBCLANG_PATH` | libclang.dll 所在目录 | `C:/Program Files/llvm/bin` |
| `FFMPEG_DIR` | FFmpeg 根目录 | `D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite` |
| `OPENCV_LINK_PATHS` | OpenCV 库文件路径 | 见下方 |
| `OPENCV_INCLUDE_PATHS` | OpenCV 头文件路径 | 见下方 |
| `OPENCV_LINK_LIBS` | 要链接的 OpenCV 库 | `opencv_core4,opencv_calib3d4,...` |
| `VCPKG_ROOT` | vcpkg 安装根目录 | `E:/code/research/3rdparty/vcpkg` |

#### OpenCV 环境变量详细值

```powershell
$VcpkgTriplet = "x64-windows-release"
$env:OPENCV_LINK_PATHS = "E:/code/research/3rdparty/vcpkg/installed/$VcpkgTriplet/lib,E:/code/research/3rdparty/vcpkg/installed/$VcpkgTriplet/lib/manual-link/opencv4_thirdparty"
$env:OPENCV_INCLUDE_PATHS = "E:/code/research/3rdparty/vcpkg/installed/$VcpkgTriplet/include/opencv4"
$env:OPENCV_LINK_LIBS = "opencv_core4,opencv_calib3d4,opencv_features2d4,opencv_imgproc4,opencv_video4,opencv_flann4,opencv_imgcodecs4,opencv_objdetect4,opencv_stitching4"
```

#### PATH 扩展

确保以下目录在 PATH 中：

```powershell
$env:PATH = "D:/ProgramData/Qt/6.7.3/msvc2019_64/bin;" + `
            "D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite/bin;" + `
            "D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite/bin/x64;" + `
            "E:/code/research/3rdparty/vcpkg/installed/x64-windows-release/bin;" + `
            "C:/Program Files/llvm/bin;" + `
            $env:PATH
```

> **注意**：以上路径根据实际安装的路径调整

---

### Windows 编译运行

#### 方式一：使用 run-local.ps1（适用于快速测试）

如果你不想修改 `windows.just`，可以创建 `run-local.ps1` 脚本设置所有环境变量并运行编译：

```powershell
# 完整脚本见下文 "run-local.ps1 参考" 部分
.\run-local.ps1
```

> **注意**：这种方式每次运行前都需要执行脚本设置环境变量，不如方式二（修改 windows.just）方便。

---

#### 方式二：修改 windows.just 后使用 just 编译（⭐推荐）

由于项目默认的 `common.just` 中依赖路径指向 `ext/` 目录（项目相对路径），而你手动安装的依赖可能在其他位置（如 `D:/ProgramData/Qt`、`D:/ProgramData/FFmpeg` 等），所以需要修改 `_scripts/windows.just` 配置本地路径。

**步骤 1：修改 `_scripts/windows.just`**

在文件头部添加本地环境配置（在 `import 'common.just'` 之后）：

```just
import 'common.just'

NameSuffix := if Arch == "aarch64" { "windows-arm64" } else { "windows64" }

# ===== 本地环境配置（根据你的实际安装路径修改）=====
LocalQtPath := "D:/ProgramData/Qt/6.7.3/msvc2019_64/bin"
LocalFFmpegDir := "D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite"
LocalVcpkgPath := "E:/code/research/3rdparty/vcpkg/installed/x64-windows-release"
LocalLLVMPath := "C:/Program Files/llvm/bin"
LocalQmake := LocalQtPath / "qmake.exe"
LocalOpencvLinkPaths := LocalVcpkgPath / "lib," + LocalVcpkgPath / "lib/manual-link/opencv4_thirdparty"
LocalOpencvIncludePaths := LocalVcpkgPath / "include/opencv4"
LocalExtendedPath := LocalQtPath + ";" + LocalFFmpegDir + "/bin" + ";" + LocalFFmpegDir + "/bin/x64" + ";" + LocalVcpkgPath + "/bin" + ";" + LocalLLVMPath + ";" + env_var('PATH')
# ================================================
```

**步骤 2：修改编译任务传递环境变量**

找到 `build-debug`、`build`、`run`、`debug`、`profile` 等任务，修改为显式传递环境变量：

```just
[no-cd]
build-debug *param:
    $Env:OPENCV_LINK_LIBS="{{OPENCV_LINK_LIBS}},opencv_ml4,opencv_videoio4" ; \
    $Env:FFMPEG_DIR="{{LocalFFmpegDir}}" ; \
    $Env:LIBCLANG_PATH="{{LocalLLVMPath}}" ; \
    $Env:QMAKE="{{LocalQmake}}" ; \
    $Env:OPENCV_LINK_PATHS="{{LocalOpencvLinkPaths}}" ; \
    $Env:OPENCV_INCLUDE_PATHS="{{LocalOpencvIncludePaths}}" ; \
    $Env:PATH="{{LocalExtendedPath}}" ; \
    cargo build -- {{param}}

[no-cd]
build *param:
    $Env:FFMPEG_DIR="{{LocalFFmpegDir}}" ; \
    $Env:LIBCLANG_PATH="{{LocalLLVMPath}}" ; \
    $Env:QMAKE="{{LocalQmake}}" ; \
    $Env:OPENCV_LINK_PATHS="{{LocalOpencvLinkPaths}}" ; \
    $Env:OPENCV_INCLUDE_PATHS="{{LocalOpencvIncludePaths}}" ; \
    $Env:PATH="{{LocalExtendedPath}}" ; \
    cargo build --release -- {{param}}

[no-cd]
run *param:
    $Env:FFMPEG_DIR="{{LocalFFmpegDir}}" ; \
    $Env:LIBCLANG_PATH="{{LocalLLVMPath}}" ; \
    $Env:QMAKE="{{LocalQmake}}" ; \
    $Env:OPENCV_LINK_PATHS="{{LocalOpencvLinkPaths}}" ; \
    $Env:OPENCV_INCLUDE_PATHS="{{LocalOpencvIncludePaths}}" ; \
    $Env:PATH="{{LocalExtendedPath}}" ; \
    cargo run --release -- {{param}}

[no-cd]
debug *param:
    $Env:OPENCV_LINK_LIBS="{{OPENCV_LINK_LIBS}},opencv_ml4,opencv_videoio4" ; \
    $Env:FFMPEG_DIR="{{LocalFFmpegDir}}" ; \
    $Env:LIBCLANG_PATH="{{LocalLLVMPath}}" ; \
    $Env:QMAKE="{{LocalQmake}}" ; \
    $Env:OPENCV_LINK_PATHS="{{LocalOpencvLinkPaths}}" ; \
    $Env:OPENCV_INCLUDE_PATHS="{{LocalOpencvIncludePaths}}" ; \
    $Env:PATH="{{LocalExtendedPath}}" ; \
    cargo run -- {{param}}

[no-cd]
profile *param:
    $Env:FFMPEG_DIR="{{LocalFFmpegDir}}" ; \
    $Env:LIBCLANG_PATH="{{LocalLLVMPath}}" ; \
    $Env:QMAKE="{{LocalQmake}}" ; \
    $Env:OPENCV_LINK_PATHS="{{LocalOpencvLinkPaths}}" ; \
    $Env:OPENCV_INCLUDE_PATHS="{{LocalOpencvIncludePaths}}" ; \
    $Env:PATH="{{LocalExtendedPath}}" ; \
    cargo run --profile profile -- {{param}}
```

**关键点说明**：
1. **使用普通变量而非 export**：由于 `QMAKE`、`FFMPEG_DIR` 等在 `common.just` 中已定义，不能在 `windows.just` 中重复 `export`，只能定义为普通变量（如 `LocalQmake`）
2. **显式传递环境变量**：在 PowerShell 命令中使用 `$Env:VAR="{{LocalVar}}"` 将 just 变量传递给 cargo 子进程
3. **完整 PATH**：`LocalExtendedPath` 包含了所有依赖的 bin 目录

**步骤 3：编译运行**

```powershell
# Debug 模式
just build-debug
just debug

# Release 模式
just build
just run

# Profile 模式
just profile
```

**首次编译或依赖变更时需清理缓存**：

```powershell
Remove-Item -Recurse -Force "target/release/build/ffmpeg-sys-next*" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "target/release/build/opencv-*" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "target/release/build/gyroflow-*" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "target/release/build/gyroflow_core-*" -ErrorAction SilentlyContinue
```

---

#### 方式三：手动设置环境变量后用 cargo 编译

如果你不想修改 `windows.just`，可以直接设置环境变量后用 cargo 编译：

```powershell
# 设置环境变量
$env:QMAKE = "D:/ProgramData/Qt/6.7.3/msvc2019_64/bin/qmake.exe"
$env:LIBCLANG_PATH = "C:/Program Files/llvm/bin"
$env:FFMPEG_DIR = "D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite"
$env:OPENCV_LINK_PATHS = "E:/code/research/3rdparty/vcpkg/installed/x64-windows-release/lib,E:/code/research/3rdparty/vcpkg/installed/x64-windows-release/lib/manual-link/opencv4_thirdparty"
$env:OPENCV_INCLUDE_PATHS = "E:/code/research/3rdparty/vcpkg/installed/x64-windows-release/include/opencv4"
$env:OPENCV_LINK_LIBS = "opencv_core4,opencv_calib3d4,opencv_features2d4,opencv_imgproc4,opencv_video4,opencv_flann4,opencv_imgcodecs4,opencv_objdetect4,opencv_stitching4"

# 扩展 PATH
$env:PATH = "D:/ProgramData/Qt/6.7.3/msvc2019_64/bin;" + `
            "D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite/bin;" + `
            "D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite/bin/x64;" + `
            "E:/code/research/3rdparty/vcpkg/installed/x64-windows-release/bin;" + `
            "C:/Program Files/llvm/bin;" + `
            $env:PATH

# 清理缓存（首次或依赖变更时）
Remove-Item -Recurse -Force "target/release/build/ffmpeg-sys-next*"
Remove-Item -Recurse -Force "target/release/build/opencv-*"

# 编译运行
cargo run --release
```

---

### Windows 常见问题

#### 1. 编译找不到 FFmpeg/OpenCV

**错误现象：**
```
LINK : fatal error LNK1181: 无法打开输入文件"avcodec.lib"
```

或

```
Failed to find installed OpenCV package using probes
```

**原因**：build.rs 缓存了旧的环境变量，指向了错误的依赖路径。

**解决方案**：
```powershell
# 清理相关构建缓存
Remove-Item -Recurse -Force "target/release/build/ffmpeg-sys-next*" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "target/release/build/opencv-*" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "target/release/build/gyroflow-*" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "target/release/build/gyroflow_core-*" -ErrorAction SilentlyContinue

# 重新编译
just run
```

---

#### 2. Qt qmake 找不到

**错误现象：**
```
qmake returned with error:
```

**原因**：`QMAKE` 环境变量指向了不存在的 `qmake.bat`，应使用 `qmake.exe`。

**解决方案**：
```powershell
# 错误
$env:QMAKE = ".../qmake.bat"

# 正确
$env:QMAKE = ".../qmake.exe"
```

---

#### 3. OpenCV 检测失败

**错误现象：**
```
Failed to find installed OpenCV package using probes: environment, pkg_config, cmake, vcpkg_cmake, vcpkg
```

或

```
Could not get OpenCV version from include_paths
```

**可能原因**：
1. OpenCV 安装在 `x64-windows-release`，但 just 的 `common.just` 使用 `x64-windows`
2. just 会覆盖 PowerShell 中设置的环境变量
3. 缺少 `OPENCV_LINK_LIBS` 环境变量

**解决方案**：

参考上文 "方式二：修改 windows.just 后使用 just 编译"，显式传递所有 OpenCV 环境变量。

---

#### 4. FFmpeg 链接错误

**错误现象：**
```
error: failed to run custom build command for `ffmpeg-sys-next v8.1.0`
```

**原因**：FFmpeg 版本不匹配或环境变量未正确设置。

**解决方案**：
1. 确认使用正确的 FFmpeg 版本：`ffmpeg-8.1-windows-desktop-vs2026-gpl-lite`
2. 设置 `FFMPEG_DIR` 环境变量
3. 确认 FFmpeg 包含 `.lib` 文件（shared 版本）
4. 确认 `FFMPEG_DIR` 目录下的lib目录存在pkgconfig目录且有`libavcodec.pc`、`libavutil.pc`等文件

---

#### 5. bindgen 报错（libclang 找不到）

**错误现象：**
```
error: failed to run custom build command for `opencv v0.98.2`
（或类似 bindgen 相关错误）
```

**解决方案**：
```powershell
$env:LIBCLANG_PATH = "C:/Program Files/llvm/bin"
```

---

#### 6. VS Code 调试配置

如果你使用 VS Code 进行调试，只需修改 `.vscode/launch.json` 中的 PATH 值指向你的本地依赖路径即可。

**配置步骤：**

1. **修改 `.vscode/launch.json`**

   找到 `windows` 下的 `environment.PATH`，修改为：

   ```json
   {
       "name": "PATH",
       "value": "${env:Path};D:/ProgramData/Qt/6.7.3/msvc2019_64/bin;D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite/bin;E:/code/research/3rdparty/vcpkg/installed/x64-windows-release/bin;C:/Program Files/llvm/bin"
   }
   ```

   > 注意：根据你的实际安装路径调整

2. **安装 C/C++ 扩展**

   在 VS Code 扩展商店搜索并安装 **Microsoft C/C++** 扩展，这是使用 `cppvsdbg` 调试器必需的。

3. **启动调试**

   - 按 `F5` 启动调试（会自动先执行 `Build Gyroflow (Debug)` 任务）
   - 或按 `Ctrl+Shift+D` 打开调试面板选择配置

**配置原理：**
- **构建时**：通过 `tasks.json` 调用 `just build-debug`，由 `windows.just` 提供正确的环境变量
- **运行时**：通过 `launch.json` 中的 `PATH` 环境变量让程序找到 Qt、FFmpeg 等 DLL

**`.vscode/settings.json` 的作用：**

这个文件配置了 Rust 分析器和编辑器的设置（**不影响调试功能**）：

```json
{
    // Rust 分析器启用 opencv 和 opencl 特性
    "rust-analyzer.cargo.features": ["opencv", "opencl"],
    // 使用 clippy 进行代码检查
    "rust-analyzer.check.command": "clippy",
    // 保存时自动格式化
    "editor.formatOnSave": true,
    // 显示行内提示（类型提示等）
    "editor.inlayHints.enabled": "on",
    // LLDB 调试器设置（如使用 CodeLLDB 扩展）
    "lldb.displayFormat": "auto",
    "lldb.showDisassembly": "never"
}
```

这些设置主要用于代码编辑时的智能提示、检查和格式化，调试功能主要由 `launch.json` 和 `tasks.json` 控制。

---

#### run-local.ps1 参考

以下是一个完整的 `run-local.ps1` 脚本模板，根据你的实际安装路径修改：

```powershell
# Gyroflow Windows 本地开发启动脚本
# 设置所有必要的环境变量并运行编译

# ========== 配置区域 - 根据你的实际安装路径修改 ==========

# Qt 安装路径
$QtVersion = "6.7.3"
$QtPath = "D:/ProgramData/Qt/$QtVersion/msvc2019_64"
$env:QMAKE = "$QtPath/bin/qmake.exe"

# LLVM/Clang (bindgen 必需)
$env:LIBCLANG_PATH = "C:/Program Files/llvm/bin"

# FFmpeg 路径
$env:FFMPEG_DIR = "D:/ProgramData/FFmpeg/ffmpeg-8.1-windows-desktop-vs2026-gpl-lite"

# OpenCV/vcpkg 路径
$env:VCPKG_ROOT = "E:/code/research/3rdparty/vcpkg"
$VcpkgTriplet = "x64-windows-release"
$env:OPENCV_LINK_PATHS = "$env:VCPKG_ROOT/installed/$VcpkgTriplet/lib,$env:VCPKG_ROOT/installed/$VcpkgTriplet/lib/manual-link/opencv4_thirdparty"
$env:OPENCV_INCLUDE_PATHS = "$env:VCPKG_ROOT/installed/$VcpkgTriplet/include/opencv4"
$env:OPENCV_LINK_LIBS = "opencv_core4,opencv_calib3d4,opencv_features2d4,opencv_imgproc4,opencv_video4,opencv_flann4,opencv_imgcodecs4,opencv_objdetect4,opencv_stitching4"

# ========== PATH 配置 ==========

$depPaths = @(
    "$QtPath/bin"
    "$env:FFMPEG_DIR/bin"
    "$env:FFMPEG_DIR/bin/x64"
    "$env:VCPKG_ROOT/installed/$VcpkgTriplet/bin"
    $env:LIBCLANG_PATH
)
$env:PATH = ($depPaths -join ";") + ";$env:PATH"

# ========== 打印环境变量（调试用）==========

Write-Host "=== Gyroflow 环境变量配置 ===" -ForegroundColor Cyan
Write-Host "QMAKE:             $env:QMAKE"
Write-Host "LIBCLANG_PATH:     $env:LIBCLANG_PATH"
Write-Host "FFMPEG_DIR:        $env:FFMPEG_DIR"
Write-Host "VCPKG_ROOT:        $env:VCPKG_ROOT"
Write-Host "OPENCV_LINK_PATHS: $env:OPENCV_LINK_PATHS"
Write-Host "OPENCV_INCLUDE_PATHS: $env:OPENCV_INCLUDE_PATHS"
Write-Host "==================================" -ForegroundColor Cyan
Write-Host ""

# ========== 清理缓存（可选）==========

# 首次编译或依赖变更时取消注释以下行
# Remove-Item -Recurse -Force "target/release/build/ffmpeg-sys-next*" -ErrorAction SilentlyContinue
# Remove-Item -Recurse -Force "target/release/build/opencv-*" -ErrorAction SilentlyContinue
# Remove-Item -Recurse -Force "target/release/build/qttypes-*" -ErrorAction SilentlyContinue

# ========== 编译运行 ==========

Write-Host "开始编译 Gyroflow..." -ForegroundColor Green
& cargo run --release
```

---

## Linux

*待补充*

---

## Android

### Android 环境准备

#### 1. 安装 Rust Android 目标

```powershell
# 添加 Android 目标
rustup target add aarch64-linux-android

# 安装 cargo-apk（可选，用于构建 APK）
cargo install --git https://github.com/rust-mobile/cargo-apk.git cargo-apk

# 安装 cargo-ndk（推荐，用于构建 Android 库，更稳定）
cargo install cargo-ndk
```

#### 2. 安装 Android SDK/NDK

**方式一：通过 Android Studio 安装（推荐）**

1. 下载安装 [Android Studio](https://developer.android.com/studio)
2. 打开 SDK Manager，安装以下组件：
   - Android SDK Platform 26 (或更高)
   - Android SDK Build-Tools 36.0.0
   - NDK (Side by side) 27.0 或更高版本

**方式二：手动下载**

```powershell
# 下载 Android SDK Command Line Tools
# 从 https://developer.android.com/studio#command-tools 下载

# 下载 NDK
# 从 https://developer.android.com/ndk/downloads 下载
```

目录结构示例：
```
D:/ProgramData/Android/
├── Sdk/
│   ├── build-tools/36.0.0/
│   ├── platforms/android-36/
│   └── platform-tools/
└── Ndk/
    └── android-ndk-r27d/
```

#### 3. 安装 JDK

Android 编译需要 JDK 17 或更高版本。

```powershell
# 下载 Oracle JDK 或 OpenJDK
# 例如：JDK 20
# https://www.oracle.com/java/technologies/downloads/
```

#### 4. 安装 Qt for Android

```powershell
pip install aqtinstall===3.2.0

# 安装 Qt 6.7.3 for Android
aqt install-qt windows android 6.7.3 android_arm64_v8a

# 同时安装 MinGW 版本（用于 qmlimportscanner 等工具）
aqt install-qt windows desktop 6.7.3 win64_mingw
```

#### 5. 下载 FFmpeg for Android

```powershell
cd D:/ProgramData/FFmpeg

# 下载 Android 版 FFmpeg
wget "https://master.dl.sourceforge.net/project/avbuild/android/ffmpeg-8.1-android-gpl-lite.tar.xz?viasf=1" -outfile "ffmpeg.tar.xz"

# 解压
7z x ffmpeg.tar.xz
7z x ffmpeg.tar
```

目录结构：
```
D:/ProgramData/FFmpeg/ffmpeg-8.1-android-gpl-lite/
├── lib/
│   ├── arm64-v8a/
│   ├── pkgconfig/
│   └── ...
└── include/
```

#### 6. 下载 OpenCV for Android

```powershell
cd E:/code/research/3rdparty/vcpkg/installed/android

# 下载 OpenCV Android SDK
wget "https://github.com/opencv/opencv/releases/download/4.13.0/opencv-4.13.0-android-sdk.zip" -outfile "opencv.zip"

7z x opencv.zip
```

目录结构：
```
E:/code/research/3rdparty/vcpkg/installed/android/OpenCV-android-sdk/
├── sdk/
│   ├── native/
│   │   ├── jni/include/
│   │   ├── staticlibs/arm64-v8a/
│   │   └── 3rdparty/libs/arm64-v8a/
```

#### 7. 安装 LLVM

与 Windows 编译相同，需要 LLVM 用于 bindgen。

```powershell
# 下载 LLVM
wget "https://github.com/llvm/llvm-project/releases/download/llvmorg-19.1.7/LLVM-19.1.7-win64.exe" -outfile "llvm.exe"

# 解压
7z x -y llvm.exe -ollvm
```

---

### Android 环境变量配置

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `ANDROID_NDK_HOME` | NDK 根目录 | `D:/ProgramData/Android/Ndk/android-ndk-r27d` |
| `ANDROID_HOME` | SDK 根目录 | `D:/ProgramData/Android/Sdk` |
| `JAVA_HOME` | JDK 根目录 | `D:/Programy/Java/jdk-20.0.2` |

---

### Android 编译配置

#### 修改 `_scripts/android.just`

由于项目默认使用 `ext/` 目录的相对路径，需要修改为本地实际安装路径。

> 详细内容参考下文 "附录：android.just 参考" 章节

---

### Android 编译运行

```powershell
# 使用 just 编译（推荐）
just android deploy

# 编译并安装到设备
just android deploy install
```

编译输出：
- APK: `_deployment/_binaries/Gyroflow.apk`
- AAB: `_deployment/_binaries/Gyroflow.aab`（如果设置 BUILD_APP_STORE_BUNDLE）

---

### Android 常见问题

#### 1. ffmpeg-sys-next 编译失败

**错误现象：**
```
error: failed to run custom build command for `ffmpeg-sys-next`
```

**原因**：
使用 `cargo apk` 构建时，FFmpeg 的跨编译配置可能出现问题。`cargo ndk` 提供更稳定的 Android 交叉编译支持。

**解决方案：**

使用 `cargo ndk` 替代 `cargo apk` 进行构建：
```powershell
# 正确 - 使用 cargo ndk
cargo ndk -t arm64-v8a build --profile $Env:BUILD_PROFILE
```

---

#### 2. libaaudio.so 链接失败

**错误现象：**
```
ld.lld: error: unable to find library -laaudio
```

**原因**：
AAudio 是 Android API level 26+ 引入的音频库，链接器找不到 NDK 中的系统库路径。

**解决方案：**

在 `android.just` 中添加 NDK 系统库路径（已包含在上述修改中）：

```powershell
# 将 NDK 的系统库路径添加到链接器搜索路径
$Env:CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER = "rust-lld"
$Env:CARGO_TARGET_AARCH64_LINUX_ANDROID_RUSTFLAGS = "-C link-arg=-L$NDK_REPLACED/toolchains/llvm/prebuilt/windows-x86_64/sysroot/usr/lib/aarch64-linux-android/{{LocalAndroidApiLevel}}"
```

> 注意：路径中的 `{{LocalAndroidApiLevel}}` 对应 API level 目录（如 26），确保与 `Cargo.toml` 中的 `min_sdk_version` 一致。

---

#### 3. mdk-sdk 下载失败

**错误现象：**
```
Unable to download or extract mdk-sdk. Please make sure you have 7z in PATH...
```

**原因**：
`qml-video-rs` 依赖需要下载 mdk-sdk，但 SourceForge 下载可能因网络问题失败或文件损坏。

**解决方案：**

**方法 1：设置代理**

```powershell
$env:HTTP_PROXY = "http://127.0.0.1:1080"
$env:HTTPS_PROXY = "http://127.0.0.1:1080"
```

或在 `.cargo/config.toml` 中配置：
```toml
[http]
proxy = "http://127.0.0.1:1080"

[https]
proxy = "http://127.0.0.1:1080"
```

**方法 2：手动下载**

1. 从 https://sourceforge.net/projects/mdk-sdk/ 下载 `mdk-sdk-android.7z`
2. 解压到构建目录：
   ```
   target/aarch64-linux-android/release/build/qml-video-rs-<hash>/out/mdk-sdk/
   ├── include/
   └── lib/arm64-v8a/libmdk.so
   ```
3. 删除损坏的下载文件（如果存在）

---

#### 4. APK 打包失败 - 找不到 lib 目录

**错误现象：**
```
Copy-Item : 找不到路径 "...\target\release\apk\lib"，因为该路径不存在。
```

**原因**：
脚本期望的是 `cargo apk` 的输出结构，但实际使用的是 `cargo ndk`，输出路径不同：
- `cargo apk` → `target/release/apk/lib/`
- `cargo ndk` → `target/aarch64-linux-android/release/`

**解决方案：**

修改 `android.just` 中的库复制路径（已包含在上述修改中）：

```powershell
# cargo ndk outputs to target/aarch64-linux-android/<profile>/
Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libgyroflow.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
```

---

#### 5. androiddeployqt 找不到 gradle 模板

**错误现象：**
```
Cannot find template directory .../ext/6.7.3/android_arm64_v8a/src/3rdparty/gradle
```

**原因**：
`android-deploy.json` 中的 Qt 路径配置为相对路径 `ext/`，但实际安装在 `D:/ProgramData/Qt/`。

**解决方案：**

修改 `android.just` 中的 androiddeploy 配置（已包含在上述修改中）：

```powershell
$qtAndroidPathUnix = "{{LocalQtAndroidPath}}".Replace('\', '/')
$qtMingwPathUnix = "{{LocalQtMingwPath}}".Replace('\', '/')

$androiddeploy = @"
{
    "qt": "$qtAndroidPathUnix",
    "qml-importscanner-binary": "$qtMingwPathUnix/bin/qmlimportscanner.exe",
    "rcc-binary": "$qtMingwPathUnix/bin/rcc.exe",
    ...
}
"@
```

---

#### 6. Qt 库文件找不到

**错误现象：**
构建过程中提示找不到 `libQt6Core_arm64-v8a.so` 等文件。

**原因**：
Qt Android 版本的库文件命名包含架构后缀，需要创建无后缀的副本用于链接。

**解决方案：**

在 `android.just` 中已包含自动复制脚本：

```powershell
Copy-Item -Path "$QT_LIBS\libQt6Core_arm64-v8a.so" -Destination "$QT_LIBS\libQt6Core.so" -ErrorAction SilentlyContinue
Copy-Item -Path "$QT_LIBS\libQt6Gui_arm64-v8a.so" -Destination "$QT_LIBS\libQt6Gui.so" -ErrorAction SilentlyContinue
# ... 其他 Qt 库
```

---

#### 7. 应用启动崩溃 - 缺少 FFmpeg/MDK 库

**错误现象：**
APK 安装后启动立即崩溃，logcat 显示以下错误：
```
java.lang.UnsatisfiedLinkError: dlopen failed: library "libavcodec.so" not found
```
或
```
java.lang.UnsatisfiedLinkError: dlopen failed: library "libmdk.so" not found
```

**原因**：
Android 构建时，`cargo ndk` 编译的 `libffmpeg.so` 和 `libmdk.so` 位于 `target/aarch64-linux-android/release/` 目录，但 `android.just` 脚本默认没有将这些库复制到 APK 打包目录，导致 APK 中缺少 FFmpeg 和 MDK 视频播放依赖库。

**诊断方法**：

如果应用启动后立即崩溃，可以通过以下步骤确认是否缺少库文件：

```bash
# 1. 获取完整崩溃日志
adb logcat -d > full_crash.log

# 2. 搜索崩溃关键信息
cat full_crash.log | grep -A 50 "FATAL EXCEPTION\|DEBUG\|signal\|backtrace"
```

**典型错误输出示例**：
```
E AndroidRuntime: FATAL EXCEPTION: qtMainLoopThread
E AndroidRuntime: Process: xyz.gyroflow, PID: 4453
E AndroidRuntime: java.lang.UnsatisfiedLinkError: dlopen failed: library "libavcodec.so" not found
E AndroidRuntime:        at java.lang.Runtime.load0(Runtime.java:938)
E AndroidRuntime:        at java.lang.System.load(System.java:1631)
E AndroidRuntime:        at org.qtproject.qt.android.QtLoader.loadLibraryHelper(QtLoader.java:494)
```

如果看到 `UnsatisfiedLinkError` 或 `dlopen failed` 错误，说明 APK 中缺少对应的 `.so` 库文件。

**解决方案**：

修改 `_scripts/android.just`，在库复制阶段添加 FFmpeg 和 MDK 库的复制：

```powershell
# 在第 198 行后添加以下内容：
Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libffmpeg.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libmdk.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force -ErrorAction SilentlyContinue
```

如果 FFmpeg 库分散在多个文件中（`libavcodec.so`, `libavformat.so` 等），需要一并复制：

```powershell
# Copy all FFmpeg libraries
$ffmpegLibs = @(
    "libavcodec.so",
    "libavformat.so",
    "libavutil.so",
    "libswscale.so",
    "libswresample.so"
)
foreach ($lib in $ffmpegLibs) {
    $libPath = "$FFMPEG_DIR\lib\$lib"
    if (Test-Path $libPath) {
        Copy-Item -Path $libPath -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
        Write-Host "Copied $lib"
    } else {
        Write-Warning "FFmpeg library not found: $libPath"
    }
}
```

**验证修复**：

1. 重新构建 APK
2. 解压 APK 检查库文件：
   ```bash
   unzip -l Gyroflow.apk | grep -E "(ffmpeg|mdk|av)"
   ```
3. 确保 `lib/arm64-v8a/` 目录下包含所有必要的 `.so` 文件

---

### Android 完整构建流程总结

```powershell
# 1. 安装 cargo-ndk
cargo install cargo-ndk

# 2. 修改 _scripts/android.just 中的本地路径配置

# 3. 执行构建
just android deploy

# 4. 输出位置
# APK: _deployment/_binaries/Gyroflow.apk
```

---

## 附录：android.just 参考

以下是修改后的完整 `_scripts/android.just` 文件内容，供参考：

```just
import 'common.just'

KeyStore         := env_var_or_default("KEY_STORE_PATH",  "")
KeyStoreAlias    := env_var_or_default("KEY_STORE_ALIAS", "")
KeyStorePassword := env_var_or_default("KEY_STORE_PASS",  "")

# ===== 本地环境配置（根据你的实际安装路径修改）=====
# Android SDK/NDK/JDK 路径
LocalAndroidNdk := "D:/ProgramData/Android/Ndk/android-ndk-r27d"
LocalAndroidSdk := "D:/ProgramData/Android/Sdk/"
LocalJavaHome   := "D:/Programy/Java/jdk-20.0.2"
LocalAndroidApiLevel := "26"

# Qt 路径（Android 版）
LocalQtVersion := "6.7.3"
LocalQtAndroidPath := "D:/ProgramData/Qt/" + LocalQtVersion + "/android_arm64_v8a"
LocalQtMingwPath   := "D:/ProgramData/Qt/" + LocalQtVersion + "/mingw_64"

# FFmpeg 路径（Android 版）
LocalFFmpegDir := "D:/ProgramData/FFmpeg/ffmpeg-8.1-android-gpl-lite"

# OpenCV 路径（Android 版）
LocalOpenCVPath := "E:/code/research/3rdparty/vcpkg/installed/android/OpenCV-android-sdk"

# LLVM 路径
LocalLLVMPath := "C:/Program Files/llvm/bin"

# 派生变量
LocalQtLibs := LocalQtAndroidPath + "/lib"
LocalOpencvLinkPaths := LocalOpenCVPath + "/sdk/native/staticlibs/arm64-v8a," + LocalOpenCVPath + "/sdk/native/3rdparty/libs/arm64-v8a"
LocalOpencvIncludePaths := LocalOpenCVPath + "/sdk/native/jni/include"
# ================================================

export ANDROID_NDK_HOME := env_var_or_default("ANDROID_NDK_HOME", LocalAndroidNdk)
export ANDROID_HOME := env_var_or_default("ANDROID_HOME", LocalAndroidSdk)
export JAVA_HOME        := env_var_or_default("JAVA_HOME",        LocalJavaHome)
export BUILD_PROFILE    := env_var_or_default("BUILD_PROFILE",    "release") # use "deploy" for final apk

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ Dependencies ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

[no-cd]
install-deps:
    #!powershell
    $ProgressPreference = 'SilentlyContinue'
    $ErrorActionPreference = 'Stop'
    mkdir "{{ExtDir}}" -ErrorAction SilentlyContinue
    cd {{ExtDir}}

    $Python = "python";
    if (Get-Command "python.exe" -ErrorAction SilentlyContinue) {
        $Python = "python";
    } elseif (Get-Command "python3.exe" -ErrorAction SilentlyContinue) {
        $Python = "python3";
    } else {
        if (-not (Test-Path -Path "{{ExtDir}}/WPy64-310100b3")) {
            echo "Downloading WinPython"
            wget https://github.com/winpython/winpython/releases/download/6.1.20230227/Winpython64-3.10.10.0dotb3.exe -outfile "winpython.exe"
            7z x -y winpython.exe
            del winpython.exe
        }
        $Python = resolve-path "{{ExtDir}}\WPy64-310100b3\python-3.10.10.amd64\python.exe"
    }

    cargo install --git https://github.com/rust-mobile/cargo-apk.git cargo-apk
    rustup target add aarch64-linux-android

    # ――――――――――――――――――――――――――――――――――――――――― Qt ――――――――――――――――――――――――――――――――――――――――――
    if (-not (Test-Path -Path "{{QtVersion}}/android_arm64_v8a")) {
        echo "Downloading Qt {{QtVersion}} for Android"
        & $Python -m pip install -U pip aqtinstall===3.2.0
        & $Python -m aqt install-qt windows desktop {{QtVersion}} win64_mingw
        & $Python -m aqt install-qt windows android {{QtVersion}} android_arm64_v8a
    }
    # ――――――――――――――――――――――――――――――――――――――――― Qt ――――――――――――――――――――――――――――――――――――――――――

    # ――――――――――――――――――――――――――――――――――――――― FFmpeg ――――――――――――――――――――――――――――――――――――――――
    $FFmpeg = "ffmpeg-8.1-android-gpl-lite"
    if (-not (Test-Path -Path "$FFmpeg")) {
        echo "Downloading $FFmpeg"
        wget "https://master.dl.sourceforge.net/project/avbuild/android/$FFmpeg.tar.xz?viasf=1" -outfile "ffmpeg.tar.xz"
        7z x -aoa ffmpeg.tar.xz
        del ffmpeg.tar.xz
        7z x -aoa ffmpeg.tar
        del ffmpeg.tar
    }
    # ――――――――――――――――――――――――――――――――――――――― FFmpeg ――――――――――――――――――――――――――――――――――――――――

    # ――――――――――――――――――――――――――――――――――――――― OpenCV ――――――――――――――――――――――――――――――――――――――――
    if (-not (Test-Path -Path "OpenCV-android-sdk/")) {
        echo "Downloading OpenCV for Android"
        wget "https://github.com/opencv/opencv/releases/download/4.11.0/opencv-4.11.0-android-sdk.zip" -outfile "opencv.zip"
        7z x -aoa opencv.zip
        del opencv.zip
    }
    # ――――――――――――――――――――――――――――――――――――――― OpenCV ――――――――――――――――――――――――――――――――――――――――

    # ――――――――――――――――――――――――――――――――――――――― LLVM ――――――――――――――――――――――――――――――――――――――――
    if (-not (Test-Path -Path "{{LLVMPath}}\libclang.dll")) {
        echo "Downloading LLVM"
        wget "https://github.com/llvm/llvm-project/releases/download/llvmorg-19.1.7/LLVM-19.1.7-win64.exe" -outfile "llvm-win64.exe"
        7z x -y llvm-win64.exe -ollvm
        del "llvm-win64.exe"
    }
    # ――――――――――――――――――――――――――――――――――――――― LLVM ――――――――――――――――――――――――――――――――――――――――

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ Development ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#[no-cd]
#run *param:
#    cargo run --release -- {{param}}
#
#[no-cd]
#debug *param:
#    cargo run -- {{param}}

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ Deployment ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

[no-cd]
deploy *param:
    #!powershell
    $ErrorActionPreference = 'Stop'

    $ProjectDir = (resolve-path "{{ProjectDir}}").ToString()
    cd $ProjectDir

    $Env:CARGO_TARGET_DIR = "$ProjectDir\target"

    # 使用本地路径变量（从 just 传递）
    $QT_LIBS = "{{LocalQtLibs}}"
    $QT_ANDROID_BIN = "{{LocalQtAndroidPath}}/bin"
    $QT_MINGW_BIN = "{{LocalQtMingwPath}}/bin"
    $FFMPEG_DIR = "{{LocalFFmpegDir}}"
    $OPENCV_PATH = "{{LocalOpenCVPath}}"

    $Env:Path = "$QT_ANDROID_BIN;$QT_MINGW_BIN;$Env:Path"
    $Env:LIBCLANG_PATH = "{{LocalLLVMPath}}"
    $Env:QMAKE = "$QT_ANDROID_BIN\qmake.bat"
    $Env:FFMPEG_DIR = $FFMPEG_DIR
    $Env:OPENCV_LINK_LIBS = "opencv_stitching,opencv_calib3d,opencv_features2d,opencv_imgproc,opencv_video,opencv_flann,opencv_core,tegra_hal,tbb,ittnotify,z,kleidicv,kleidicv_hal,kleidicv_thread"
    $Env:OPENCV_LINK_PATHS = "{{LocalOpencvLinkPaths}}"
    $Env:OPENCV_INCLUDE_PATHS = "{{LocalOpencvIncludePaths}}"

    $CLANG_LIB = $Env:LIBCLANG_PATH.replace('\', '/').replace('/bin', '/lib');
    $NDK_REPLACED = $Env:ANDROID_NDK_HOME.replace('\', '/');
    $SDK_REPLACED = $Env:ANDROID_HOME.replace('\', '/');
    $PROJECT_DIR_UNIX = $ProjectDir.replace('\', '/');
    $Env:BINDGEN_EXTRA_CLANG_ARGS = "-I$CLANG_LIB/clang/16.0.0/include --sysroot=$NDK_REPLACED/toolchains/llvm/prebuilt/windows-x86_64/sysroot"
    echo $Env:BINDGEN_EXTRA_CLANG_ARGS

    # 将 NDK 的系统库路径添加到链接器搜索路径
    $Env:CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER = "rust-lld"
    $Env:CARGO_TARGET_AARCH64_LINUX_ANDROID_RUSTFLAGS = "-C link-arg=-L$NDK_REPLACED/toolchains/llvm/prebuilt/windows-x86_64/sysroot/usr/lib/aarch64-linux-android/{{LocalAndroidApiLevel}}"

    # 打印构建详细日志（貌似没啥作用）
    #$Env:RUST_BACKTRACE = "full"
    #$Env:CARGO_PROFILE_RELEASE_BUILD_OVERRIDE_DEBUG = "true"

    if ("$Env:GITHUB_RUN_NUMBER") {
        $manifest = Join-Path $ProjectDir "_deployment\android\AndroidManifest.xml"
        $runNumber = $Env:GITHUB_RUN_NUMBER
        [System.IO.File]::WriteAllText($manifest, [regex]::Replace([System.IO.File]::ReadAllText($manifest), 'versionCode="\d+"', "versionCode=""$runNumber"""))
    }

    Copy-Item -Path "$QT_LIBS\libQt6Core_arm64-v8a.so"           -Destination "$QT_LIBS\libQt6Core.so"           -ErrorAction SilentlyContinue
    Copy-Item -Path "$QT_LIBS\libQt6Gui_arm64-v8a.so"            -Destination "$QT_LIBS\libQt6Gui.so"            -ErrorAction SilentlyContinue
    Copy-Item -Path "$QT_LIBS\libQt6Widgets_arm64-v8a.so"        -Destination "$QT_LIBS\libQt6Widgets.so"        -ErrorAction SilentlyContinue
    Copy-Item -Path "$QT_LIBS\libQt6Quick_arm64-v8a.so"          -Destination "$QT_LIBS\libQt6Quick.so"          -ErrorAction SilentlyContinue
    Copy-Item -Path "$QT_LIBS\libQt6Qml_arm64-v8a.so"            -Destination "$QT_LIBS\libQt6Qml.so"            -ErrorAction SilentlyContinue
    Copy-Item -Path "$QT_LIBS\libQt6QuickControls2_arm64-v8a.so" -Destination "$QT_LIBS\libQt6QuickControls2.so" -ErrorAction SilentlyContinue

    # Make sure qmlimportscanner finds QtQuick.Effects
    [System.IO.File]::WriteAllText("$ProjectDir\src\ui\components\Modal.qml", [System.IO.File]::ReadAllText("$ProjectDir\src\ui\components\Modal.qml").Replace("`n// import QtQuick.Effects", "`nimport QtQuick.Effects"))

    # Replace [[bin]] with [lib]
    [System.IO.File]::WriteAllText("$ProjectDir\Cargo.toml", [System.IO.File]::ReadAllText("$ProjectDir\Cargo.toml").Replace("[[bin]]", "[lib]`ncrate-type = [""cdylib""]"))

    # Replace gradle version
    [System.IO.File]::WriteAllText("$QT_LIBS\..\src\3rdparty\gradle\gradle\wrapper\gradle-wrapper.properties", [System.IO.File]::ReadAllText("$QT_LIBS\..\src\3rdparty\gradle\gradle\wrapper\gradle-wrapper.properties").Replace("gradle-8.3-bin", "gradle-8.6-bin"))
    [System.IO.File]::WriteAllText("$QT_LIBS\..\src\android\templates\build.gradle", [System.IO.File]::ReadAllText("$QT_LIBS\..\src\android\templates\build.gradle").Replace("7.4.1", "8.4.2"))

    cargo ndk -t arm64-v8a build --profile $Env:BUILD_PROFILE

    # Restore [[bin]]
    [System.IO.File]::WriteAllText("$ProjectDir\Cargo.toml", [System.IO.File]::ReadAllText("$ProjectDir\Cargo.toml").Replace("[lib]`ncrate-type = [""cdylib""]", "[[bin]]"))

    mkdir "$ProjectDir\target\android-build" -ErrorAction SilentlyContinue
    mkdir "$ProjectDir\target\android-build\libs" -ErrorAction SilentlyContinue
    mkdir "$ProjectDir\target\android-build\libs\arm64-v8a" -ErrorAction SilentlyContinue
    # cargo ndk outputs to target/aarch64-linux-android/<profile>/, copy the built library
    Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libgyroflow.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
    # Copy-Item -Path "$ProjectDir\_deployment\android\src" -Destination "$ProjectDir\target\android-build\" -Recurse -Force
    Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libffmpeg.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
    Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libmdk.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
    # Copy all FFmpeg libraries from FFmpeg directory
    $ffmpegLibs = @(
        "libavcodec.so",
        "libavformat.so",
        "libavutil.so",
        "libswscale.so",
        "libswresample.so"
    )
    foreach ($lib in $ffmpegLibs) {
        $libPath = "$FFMPEG_DIR\lib\$lib"
        if (Test-Path $libPath) {
            Copy-Item -Path $libPath -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
            Write-Host "Copied $lib"
        } else {
            Write-Warning "FFmpeg library not found: $libPath"
        }
    }
    # Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libqtav-mediacodec.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force
    Move-Item -Path "$ProjectDir\target\android-build\libs\arm64-v8a\libgyroflow.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\libgyroflow_arm64-v8a.so" -Force

    $qtlibs = @(
        "libQt6Core_arm64-v8a.so",
        "libQt6Gui_arm64-v8a.so",
        "libQt6LabsFolderListModel_arm64-v8a.so",
        "libQt6Network_arm64-v8a.so",
        "libQt6OpenGL_arm64-v8a.so",
        "libQt6Qml_arm64-v8a.so",
        "libQt6QmlLocalStorage_arm64-v8a.so",
        "libQt6QmlModels_arm64-v8a.so",
        "libQt6QmlWorkerScript_arm64-v8a.so",
        "libQt6QmlXmlListModel_arm64-v8a.so",
        "libQt6Quick_arm64-v8a.so",
        "libQt6QuickControls2_arm64-v8a.so",
        "libQt6QuickControls2Impl_arm64-v8a.so",
        "libQt6QuickControls2Material_arm64-v8a.so",
        "libQt6QuickControls2MaterialStyleImpl_arm64-v8a.so",
        "libQt6QuickControls2Basic_arm64-v8a.so",
        "libQt6QuickControls2BasicStyleImpl_arm64-v8a.so",
        "libQt6QuickDialogs2_arm64-v8a.so",
        "libQt6QuickDialogs2QuickImpl_arm64-v8a.so",
        "libQt6QuickDialogs2Utils_arm64-v8a.so",
        "libQt6QuickLayouts_arm64-v8a.so",
        "libQt6QuickParticles_arm64-v8a.so",
        "libQt6QuickShapes_arm64-v8a.so",
        "libQt6QuickTemplates2_arm64-v8a.so",
        "libQt6QuickEffects_arm64-v8a.so",
        "libQt6Sql_arm64-v8a.so",
        "libQt6Svg_arm64-v8a.so",
        "libQt6Widgets_arm64-v8a.so",
        "..\plugins\iconengines\libplugins_iconengines_qsvgicon_arm64-v8a.so",
        "..\plugins\imageformats\libplugins_imageformats_qsvg_arm64-v8a.so",
        "..\plugins\sqldrivers\libplugins_sqldrivers_qsqlite_arm64-v8a.so",
        "..\qml\Qt\labs\folderlistmodel\libqml_Qt_labs_folderlistmodel_qmlfolderlistmodelplugin_arm64-v8a.so",
        "..\qml\QtCore\libqml_QtCore_qtqmlcoreplugin_arm64-v8a.so",
        "..\qml\QtQml\Base\libqml_QtQml_Base_qmlplugin_arm64-v8a.so",
        "..\qml\QtQml\libqml_QtQml_qmlmetaplugin_arm64-v8a.so",
        "..\qml\QtQml\Models\libqml_QtQml_Models_modelsplugin_arm64-v8a.so",
        "..\qml\QtQml\WorkerScript\libqml_QtQml_WorkerScript_workerscriptplugin_arm64-v8a.so",
        "..\qml\QtQml\XmlListModel\libqml_QtQml_XmlListModel_qmlxmllistmodelplugin_arm64-v8a.so",
        "..\qml\QtQuick\Controls\Basic\impl\libqml_QtQuick_Controls_Basic_impl_qtquickcontrols2basicstyleimplplugin_arm64-v8a.so",
        "..\qml\QtQuick\Controls\Basic\libqml_QtQuick_Controls_Basic_qtquickcontrols2basicstyleplugin_arm64-v8a.so",
        "..\qml\QtQuick\Controls\impl\libqml_QtQuick_Controls_impl_qtquickcontrols2implplugin_arm64-v8a.so",
        "..\qml\QtQuick\Controls\libqml_QtQuick_Controls_qtquickcontrols2plugin_arm64-v8a.so",
        "..\qml\QtQuick\Controls\Material\impl\libqml_QtQuick_Controls_Material_impl_qtquickcontrols2materialstyleimplplugin_arm64-v8a.so",
        "..\qml\QtQuick\Controls\Material\libqml_QtQuick_Controls_Material_qtquickcontrols2materialstyleplugin_arm64-v8a.so",
        "..\qml\QtQuick\Dialogs\libqml_QtQuick_Dialogs_qtquickdialogsplugin_arm64-v8a.so",
        "..\qml\QtQuick\Dialogs\quickimpl\libqml_QtQuick_Dialogs_quickimpl_qtquickdialogs2quickimplplugin_arm64-v8a.so",
        "..\qml\QtQuick\Effects\libqml_QtQuick_Effects_effectsplugin_arm64-v8a.so",
        "..\qml\QtQuick\Layouts\libqml_QtQuick_Layouts_qquicklayoutsplugin_arm64-v8a.so",
        "..\qml\QtQuick\libqml_QtQuick_qtquick2plugin_arm64-v8a.so",
        "..\qml\QtQuick\LocalStorage\libqml_QtQuick_LocalStorage_qmllocalstorageplugin_arm64-v8a.so",
        "..\qml\QtQuick\NativeStyle\libqml_QtQuick_NativeStyle_qtquickcontrols2nativestyleplugin_arm64-v8a.so",
        "..\qml\QtQuick\Particles\libqml_QtQuick_Particles_particlesplugin_arm64-v8a.so",
        "..\qml\QtQuick\Shapes\libqml_QtQuick_Shapes_qmlshapesplugin_arm64-v8a.so",
        "..\qml\QtQuick\Templates\libqml_QtQuick_Templates_qtquicktemplates2plugin_arm64-v8a.so",
        "..\qml\QtQuick\tooling\libqml_QtQuick_tooling_quicktoolingplugin_arm64-v8a.so",
        "..\qml\QtQuick\Window\libqml_QtQuick_Window_quickwindowplugin_arm64-v8a.so"
    );
    foreach ($x in $qtlibs) {
        Copy-Item -Path "$QT_LIBS\$x" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\" -Force -ErrorAction SilentlyContinue
    }

    # Strip all libs
    . "$Env:ANDROID_NDK_HOME\toolchains\llvm\prebuilt\windows-x86_64\bin\llvm-strip.exe" "$ProjectDir\target\android-build\libs\arm64-v8a\*.so"

    # Copy unstripped libgyroflow.so (from cargo ndk output)
    Copy-Item -Path "$ProjectDir\target\aarch64-linux-android\$Env:BUILD_PROFILE\libgyroflow.so" -Destination "$ProjectDir\target\android-build\libs\arm64-v8a\libgyroflow_arm64-v8a.so" -Force

    $qtAndroidPathUnix = "{{LocalQtAndroidPath}}".Replace('\', '/')
    $qtMingwPathUnix = "{{LocalQtMingwPath}}".Replace('\', '/')

    $androiddeploy = @"
    {
        "description": "",
        "qt": "$qtAndroidPathUnix",
        "qtDataDirectory": ".",
        "qtLibExecsDirectory": "libexec",
        "qtLibsDirectory": "lib",
        "qtPluginsDirectory": "plugins",
        "qtQmlDirectory": "qml",
        "sdk": "$SDK_REPLACED",
        "sdkBuildToolsRevision": "36.0.0",
        "ndk": "$NDK_REPLACED",
        "toolchain-prefix": "llvm",
        "tool-prefix": "llvm",
        "ndk-host": "windows-x86_64",
        "architectures": {"arm64-v8a":"aarch64-linux-android"},
        "android-min-sdk-version": "26",
        "android-package-source-directory": "$PROJECT_DIR_UNIX/_deployment/android",
        "android-target-sdk-version": "36",
        "qml-importscanner-binary": "$qtMingwPathUnix/bin/qmlimportscanner.exe",
        "rcc-binary": "$qtMingwPathUnix/bin/rcc.exe",
        "qml-root-path": "$PROJECT_DIR_UNIX/src",
        "stdcpp-path": "$NDK_REPLACED/toolchains/llvm/prebuilt/windows-x86_64/sysroot/usr/lib",
        "qrcFiles": "",
        "application-binary": "gyroflow"
    }
    "@
    $androiddeploy | Out-File -encoding utf8 -FilePath "$ProjectDir\target\android-build\android-deploy.json"

    $job = Start-Job -Name "Cleanup" -ArgumentList "$ProjectDir\target\android-build\libs\arm64-v8a" -ScriptBlock {
        for ($num = 1; $num -le 800 ; $num++) {
            try {
                $c = [System.IO.File]::ReadAllText("${args}\..\..\res\values\libs.xml") -replace "<item.+?(Fusion|Imagine|Universal).+?item>", ""
                [System.IO.File]::WriteAllText("${args}\..\..\res\values\libs.xml", $c)
                Remove-Item "${args}\*Universal*"
                Remove-Item "${args}\*Imagine*"
                Remove-Item "${args}\*Fusion*"
                Remove-Item "${args}\*Particles*"
                Remove-Item "${args}\*tooling*"
                Remove-Item "${args}\*qgif*"
                Remove-Item "${args}\*qico*"
            } catch {

            }
            Start-Sleep -Milliseconds 150;
        }
    }
    Get-job | Receive-Job

    $Sign = ""
    $ApkName = "apk\debug\android-build-debug.apk"
    if ("{{KeyStore}}" -and "{{KeyStoreAlias}}" -and "{{KeyStorePassword}}") {
        $Sign = "--release --sign {{KeyStore}} {{KeyStoreAlias}} --storepass {{KeyStorePassword}}"
        $ApkName = "apk\release\android-build-release-signed.apk"
    }

    $cmd = "androiddeployqt --input `"$ProjectDir\target\android-build\android-deploy.json`" --output `"$ProjectDir\target\android-build`" --deployment bundled $Sign --android-platform android-36 --jdk ${Env:JAVA_HOME} --gradle"

    iex "$cmd"
    if ("$Env:BUILD_APP_STORE_BUNDLE" -eq "true") {
        iex "$cmd --aab"
    }

    [System.IO.File]::WriteAllText("$ProjectDir\src\ui\components\Modal.qml", [System.IO.File]::ReadAllText("$ProjectDir\src\ui\components\Modal.qml").Replace("`nimport QtQuick.Effects", "`n// import QtQuick.Effects"))

    Stop-Job $job

    New-Item -Type dir "$ProjectDir\_deployment\_binaries\" -ErrorAction SilentlyContinue
    Copy-Item -Path "$ProjectDir\target\android-build\build\outputs\$ApkName" -Destination "$ProjectDir\_deployment\_binaries\Gyroflow.apk" -Force -ErrorAction SilentlyContinue
    Copy-Item -Path "$ProjectDir\target\android-build\build\outputs\bundle\release\android-build-release.aab" -Destination "$ProjectDir\_deployment\_binaries\Gyroflow.aab" -Force -ErrorAction SilentlyContinue

    if ("{{param}}" -eq "install") {
        adb install "$ProjectDir\_deployment\_binaries\Gyroflow.apk"
        adb logcat -c
        adb shell am start -n xyz.gyroflow/xyz.gyroflow.MainActivity
        adb logcat -s Gyroflow
    }
```

**主要修改点总结：**

1. **添加本地路径变量**（第 7-31 行）：定义所有依赖的本地安装路径
2. **添加 API Level 变量**（第 13 行）：`LocalAndroidApiLevel := "26"`
3. **将 `cargo apk` 改为 `cargo ndk`（更稳定）**
4. **修复库输出路径**：将 `cargo apk` 输出路径改为 `cargo ndk` 输出路径
5. **添加 NDK 系统库路径**：修复 libaaudio.so 链接问题
6. **修复 Qt 路径**：使用 `LocalQtAndroidPath` 和 `LocalQtMingwPath` 变量
7. **修复 app 启动崩溃**：添加FFmpeg、mdk库文件拷贝到打包目录

## 参考链接

- [Gyroflow 官方文档](https://docs.gyroflow.xyz/)
- [Gyroflow GitHub](https://github.com/gyroflow/gyroflow)
- [opencv-rust 文档](https://github.com/twistedfall/opencv-rust#getting-opencv)
- [ffmpeg-sys-next 文档](https://github.com/zmwangx/rust-ffmpeg-sys)
