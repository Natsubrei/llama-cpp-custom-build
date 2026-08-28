# llama.cpp Windows CUDA All Quants Build

使用 GitHub Actions 自动构建适用于 Windows 的 [`llama.cpp`](https://github.com/ggml-org/llama.cpp) `llama-server`。

本仓库**不包含 llama.cpp 源代码**。工作流运行时会自动从上游 `ggml-org/llama.cpp` 拉取最新源码，并使用 CUDA 13.3 构建启用了 CUDA Flash Attention 与 All Quants 支持的 Windows 版本。

## 构建配置

当前工作流使用：

* Windows Server 2022
* Visual Studio 2022
* CUDA 13.3
* CMake
* Ninja
* x64
* Blackwell CUDA 架构
* `llama-server`
* BoringSSL

主要 CMake 选项：

```text
GGML_CUDA=ON
GGML_CUDA_FA=ON
GGML_CUDA_FA_ALL_QUANTS=ON
GGML_NATIVE=OFF
CMAKE_CUDA_ARCHITECTURES=120a-real
LLAMA_BUILD_BORINGSSL=ON
LLAMA_BUILD_SERVER=ON
LLAMA_BUILD_TESTS=OFF
LLAMA_BUILD_EXAMPLES=OFF
```

其中：

* `GGML_CUDA=ON`：启用 CUDA 后端。
* `GGML_CUDA_FA=ON`：启用 CUDA Flash Attention。
* `GGML_CUDA_FA_ALL_QUANTS=ON`：为支持的量化类型启用 Flash Attention CUDA 实现。
* `GGML_NATIVE=OFF`：避免针对 GitHub Actions 构建机 CPU 生成本机专用指令。
* `CMAKE_CUDA_ARCHITECTURES=120a-real`：针对对应的 Blackwell CUDA 架构生成实际 GPU 代码。
* `LLAMA_BUILD_BORINGSSL=ON`：构建 llama.cpp 的 BoringSSL 支持。
* `LLAMA_BUILD_TESTS=OFF`：不构建测试程序。
* `LLAMA_BUILD_EXAMPLES=OFF`：不构建其他示例程序。

## 使用方法

进入仓库的：

**Actions → Build llama.cpp CUDA All Quants → Run workflow**

手动启动构建即可。

工作流只配置了：

```yaml
on:
  workflow_dispatch:
```

因此不会因为 push 或 pull request 自动运行。

## 构建流程

GitHub Actions 会依次执行：

```text
拉取 ggml-org/llama.cpp
        ↓
安装 CUDA 13.3
        ↓
安装 Ninja
        ↓
配置 CMake
        ↓
构建 llama-server
        ↓
复制 CUDA Runtime DLL
        ↓
验证 CUDA 构建配置
        ↓
上传 Artifact
```

构建完成后，可以在对应的 GitHub Actions Workflow Run 页面下载：

```text
llama-cpp-win-cuda13.3-blackwell-all-quants
```

## Artifact 内容

Artifact 来自：

```text
build/bin/Release/*
```

其中包括构建产生的 `llama-server.exe`、llama.cpp 运行时 DLL，以及所需的 CUDA Runtime DLL。

工作流会额外搜索并复制：

```text
cudart64_*.dll
cublas64_*.dll
cublasLt64_*.dll
```

因此下载 Artifact 后通常无需单独从 CUDA Toolkit 目录复制这些运行库。

## 构建验证

上传 Artifact 之前，工作流会检查至少存在：

```text
llama-server.exe
ggml-cuda.dll
```

同时验证 `CMakeCache.txt` 中：

```text
GGML_CUDA=ON
GGML_CUDA_FA=ON
GGML_CUDA_FA_ALL_QUANTS=ON
```

只有这些检查通过后才会继续上传构建结果。

## 上游版本

本仓库没有固定 llama.cpp commit。

每次运行 Workflow 时都会执行：

```yaml
uses: actions/checkout@v6
with:
  repository: ggml-org/llama.cpp
  fetch-depth: 1
```

因此构建的是**工作流运行时 llama.cpp 上游默认分支的最新版本**。

这意味着不同时间运行得到的二进制文件可能来自不同的 llama.cpp commit，也可能因为上游构建系统发生变化而出现构建失败。

如果需要完全可复现的构建，可以在 Workflow 中为 `actions/checkout` 添加固定的 `ref`，例如指定 commit SHA。

## 适用范围

这个 Workflow 主要用于生成：

```text
Windows x64
+
CUDA 13.3
+
Blackwell
+
llama-server
+
Flash Attention
+
All Quants
```

版本的 llama.cpp。

它不是通用的 llama.cpp Windows 构建矩阵，也不会生成 CPU-only、Vulkan、HIP 或其他 CUDA 架构版本。

## License

本仓库仅包含 GitHub Actions 构建配置。

`llama.cpp` 本身由 `ggml-org/llama.cpp` 项目维护，其源代码和生成二进制文件的许可条款请参阅上游项目：

https://github.com/ggml-org/llama.cpp
