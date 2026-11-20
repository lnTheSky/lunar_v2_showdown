<p align="center">
  <a href="https://t.me/lunaraim2">
    <img src="assets/brand/lunar-aim-banner.png" alt="LunarAim 2.2 — 全新界面" width="760">
  </a>
</p>

<p align="center">
  <a href="https://t.me/in_the_skyyy"><img src="https://img.shields.io/badge/%40in__the__skyyy-2AABEE?style=for-the-badge&logo=telegram&logoColor=white" alt="联系 @in_the_skyyy"></a>
  <a href="https://t.me/lunaraim"><img src="https://img.shields.io/badge/%40lunaraim-2548BB?style=for-the-badge&logo=telegram&logoColor=white" alt="LunarAim 频道 @lunaraim"></a>
  <a href="https://t.me/lunaraim2"><img src="https://img.shields.io/badge/%40lunaraim2-17C3E1?style=for-the-badge&logo=telegram&logoColor=white" alt="LunarAim 频道 @lunaraim2"></a>
</p>

<p align="center"><a href="README.md">English</a> · <a href="README.ru-RU.md">Русский</a> · <a href="README.zh-CN.md">简体中文</a></p>

<h3 align="center">构建技术</h3>

<p align="center">
  <a href="https://developer.microsoft.com/en-us/microsoft-edge/webview2/"><img src="https://img.shields.io/badge/WebView2-107C10?style=for-the-badge&logo=microsoftedge&logoColor=white" alt="Microsoft WebView2"></a>
  <a href="https://learn.microsoft.com/en-us/windows/win32/"><img src="https://img.shields.io/badge/Win32%20API-0078D4?style=for-the-badge&logo=windows&logoColor=white" alt="Win32 API"></a>
  <a href="https://cmake.org/"><img src="https://img.shields.io/badge/CMake-064F8C?style=for-the-badge&logo=cmake&logoColor=white" alt="CMake"></a>
  <a href="https://visualstudio.microsoft.com/vs/features/cplusplus/"><img src="https://img.shields.io/badge/C%2B%2B%20%2F%20MSVC-5C2D91?style=for-the-badge&logo=visualstudio&logoColor=white" alt="C++ 和 MSVC"></a>
  <a href="https://developer.nvidia.com/cuda-toolkit"><img src="https://img.shields.io/badge/CUDA-76B900?style=for-the-badge&logo=nvidia&logoColor=white" alt="NVIDIA CUDA"></a>
  <a href="https://developer.nvidia.com/tensorrt"><img src="https://img.shields.io/badge/TensorRT-76B900?style=for-the-badge&logo=nvidia&logoColor=white" alt="NVIDIA TensorRT"></a>
  <a href="https://pytorch.org/"><img src="https://img.shields.io/badge/PyTorch-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white" alt="PyTorch"></a>
  <a href="https://github.com/ultralytics/ultralytics"><img src="https://img.shields.io/badge/Ultralytics%20YOLO-111F68?style=for-the-badge&logoColor=white" alt="Ultralytics YOLO"></a>
</p>

<details>
<summary>目录</summary>
<br>

- [本仓库展示的内容](#本仓库展示的内容)
- [架构](#架构)
  - [Runtime 边界](#runtime-边界)
  - [新增内容](#新增内容)
- [界面画廊](#界面画廊)
  - [Runtime 仪表盘](#runtime-仪表盘)
  - [检测器配置](#检测器配置)
  - [Runtime 工具](#runtime-工具)
- [检测实验](#检测实验)
- [公开与私有边界](#公开与私有边界)
- [项目资料](#项目资料)

</details>

> **公开技术概览**：介绍 Lunar V2 的 runtime 边界、前端集成、视觉语言以及计算机视觉实验。生产应用及其实现仍为私有内容。

## 本仓库展示的内容

LunarAim 2.2 是一个采用原生 Windows 技术构建的应用，围绕一个小型、可观测的 runtime 核心组织。界面通过结构化 bridge 作为 backend 的客户端运行，平台相关的输入和 GPU 功能则被隔离在明确的边界之后。

- **CoreRuntime** 负责启动、运行、停止、配置、状态快照和 telemetry。
- **Frontend Bridge** 为嵌入式 UI 提供结构化 native API。
- **可组合的契约** 使 aim、detection、trigger 和 mouse output 可以独立替换。
- **隔离的平台模块** 将 Win32 Raw Input 与 CUDA / TensorRT runtime 分开。
- **操作工具** 展示 runtime 状态、性能、配置、filters、binds 和日志。

## 架构

```mermaid
flowchart TB
    frontend["Embedded frontend"]
    bridge["FrontendHost / FrontendBridge"]
    dispatcher["Native API dispatcher"]
    runtime["CoreRuntime<br/>starting → running → stopping<br/>error / stopped"]
    aim["AimController"]
    trigger["Trigger"]
    detector["Detector"]
    capture["Capture"]
    mouse["Mouse"]
    detectorContract["IDetector contract"]
    colorbot["Colorbot<br/>(OpenCV)"]
    cuda["CUDA / TensorRT module<br/>loaded separately"]
    rawInput["Win32 Raw Input<br/>separate thread"]
    triggerContract["ITrigger"]
    triggerV1["TriggerV1"]
    triggerV2["TriggerV2"]
    mouseContract["IMouse"]
    winapi["WinAPI"]
    lunarBox["LunarBox V1 / V2"]
    vdriver["Virtual Driver<br/>(VDriver)"]
    desktopDuplicator["DesktopDuplicator<br/>(DirectX)"]

    frontend --> bridge --> dispatcher --> runtime
    runtime --> aim
    runtime --> trigger
    runtime --> detector
    runtime --> capture
    runtime --> mouse
    detector --> detectorContract
    detectorContract --> colorbot
    detectorContract --> cuda
    trigger --> triggerContract
    triggerContract --> triggerV1
    triggerContract --> triggerV2
    mouse --> mouseContract
    mouseContract --> winapi
    mouseContract --> lunarBox
    mouseContract --> vdriver
    capture --> rawInput
    capture --> desktopDuplicator

    classDef core fill:#2548BB,color:#ffffff,stroke:#17C3E1,stroke-width:2px
    classDef boundary fill:#0f2f35,color:#ffffff,stroke:#17C3E1
    classDef module fill:#17243d,color:#ffffff,stroke:#2548BB
    class runtime core
    class bridge,dispatcher,detectorContract,triggerContract,mouseContract boundary
    class colorbot,cuda,rawInput,triggerV1,triggerV2,winapi,lunarBox,vdriver,desktopDuplicator module
```

### Runtime 边界

Runtime 被拆分为明确的边界，使每个部分都可以演进，而不会把实现细节泄漏到上层。

- **`CoreRuntime`。** 负责生命周期、worker thread、配置路径、状态快照和性能 telemetry。
- **`FrontendBridge`。** 通过 dispatcher 和 handler 将 UI 请求转换为稳定的 native API。
- **`IDetector`。** 将 aim 逻辑与检测器实现解耦，包括 Color 和 CUDA 路径。
- **`ITrigger`。** 将 runtime orchestration 与触发机制解耦。
- **`IMouse`。** 将应用逻辑与鼠标命令的发送方式解耦。
- **Raw Input module。** 通过独立线程和自身的状态边界收集 Win32 输入。
- **CUDA runtime module。** 将 CUDA / TensorRT 依赖放在单独加载的 GPU 模块中，不污染主 executable。

### 新增内容

LunarAim 2.2 将主要的 runtime 关注点整合为一条可观测的流程。嵌入式 frontend 通过结构化 bridge 通信，而不是直接访问内部实现；`CoreRuntime` 为启动、运行、停止和错误处理提供统一的生命周期。

Runtime 契约使系统更容易扩展：aim control、detection、triggering 和 mouse output 都可以独立替换。平台相关功能也被隔离：Win32 Raw Input 拥有独立的状态边界，CUDA / TensorRT 作为单独的 GPU 模块加载，而不是成为主 executable 的硬依赖。

配置、runtime 快照、性能 telemetry 和日志都是 operator experience 的一等组成部分。UI 专注于展示系统当前状态，具体实现则隐藏在清晰的接口之后。

## 界面画廊

UI 被设计为 runtime 的操作面板，并通过独立页面展示状态、检测、输入行为和诊断信息。

### Runtime 仪表盘

仪表盘直接展示 runtime 状态、当前配置、frame timing、FPS 和 telemetry。

<p align="center">
  <img src="assets/screenshots/dashboard-running-dark.png" alt="LunarAim 深色仪表盘" width="49%">
  <img src="assets/screenshots/dashboard-running-light.png" alt="LunarAim 浅色仪表盘" width="49%">
</p>

### 检测器配置

<p align="center">
  <img src="assets/screenshots/colorbot-dark.png" alt="Colorbot 配置 — 深色主题" width="49%">
  <img src="assets/screenshots/colorbot-light.png" alt="Colorbot 配置 — 浅色主题" width="49%">
</p>

### Runtime 工具

<p align="center">
  <img src="assets/screenshots/binds-light.png" alt="Binds 配置 — 浅色主题" width="49%">
  <img src="assets/screenshots/logs-dark.png" alt="核心日志 — 深色主题" width="49%">
</p>

<details>
<summary>更多界面截图</summary>
<br>

<p align="center">
  <img src="assets/screenshots/colorbot-dark-detail.png" alt="Colorbot 配置详情" width="49%">
  <img src="assets/screenshots/filters-dark.png" alt="Filters 编辑器" width="49%">
  <img src="assets/screenshots/dashboard-stopped-dark.png" alt="已停止 runtime 的仪表盘" width="49%">
  <img src="assets/screenshots/runtime-terminal.png" alt="Runtime 终端输出" width="49%">
</p>
</details>

## 检测实验

仓库包含一份可公开分享的 CS2 检测器实验 snapshot：

- 标注了 127 个游戏 demo；
- 模型训练了 150 个 epoch；
- 保存了训练曲线、标注统计和空间分布；
- `yolo11n_cs2.pt` checkpoint 作为实验产物提供。

<p align="center">
  <img src="experiments/yolo-cs2/results.png" alt="YOLO 训练和验证曲线" width="80%">
</p>

<p align="center">
  <img src="experiments/yolo-cs2/labels.jpg" alt="数据集标签分布" width="42%">
  <img src="experiments/yolo-cs2/labels_correlogram.jpg" alt="数据集标签相关图" width="42%">
</p>

## 公开与私有边界

这是一个 **showcase 仓库**，而不是 Lunar V2 源代码发行版。公开内容包括架构资料、设计文档、界面截图和选定的 ML 产物。私有 runtime 代码、生产集成、凭据和 proprietary infrastructure 均被有意排除。

<details>
<summary>仓库结构</summary>
<br>

```text
.
├── assets/
│   ├── brand/          # logo 和演示封面
│   └── screenshots/    # 命名后的 UI 截图
├── docs/               # 演示文稿和公开项目说明
├── experiments/
│   └── yolo-cs2/       # 检测器指标、图表和 checkpoint
└── README.md
```

</details>

## 项目资料

- [架构演示文稿](docs/Lunar.pdf)
- [项目说明](docs/project-description.md)
- [界面截图](assets/screenshots/)
- [CS2 检测实验](experiments/yolo-cs2/)

<br>

<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/LunAimW.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/LunAimB.svg">
    <img src="assets/brand/LunAimB.svg" alt="LunarAim logo" width="96">
  </picture>
  <h2>LunarAim 2.2</h2>
  <sub>原生 runtime 架构 · 结构化 UI bridge · GPU 检测</sub>
</div>
