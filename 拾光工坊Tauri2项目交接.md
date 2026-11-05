# 拾光工坊 Tauri 2 项目交接备忘录

更新日期：2026-11-05（Asia/Hong_Kong）  
项目目录：E:\picturework\shiguang-studio  
GitHub：yaquyese/beiwanglu（本文件是 Tauri 2 项目的独立记录）

> 本项目是新的 Tauri 2 Windows 桌面应用，与旧的 Flutter 项目分开维护。旧 Flutter 项目记录仍在“拾光工坊项目记忆与会话交接.md”，不要覆盖或把两个项目的环境、代码状态混为一谈。

## 目标与约束

- 制作证件照图片处理工具：证件照规格裁切、自由取景/缩放、图片裁剪、本地 AI 抠图，并合成白色或蓝色背景、导出 JPG。
- 当前只做 Windows 桌面版；未来可能评估 Android，现阶段不配置 Android 工具链。
- 人像分割在本机离线运行，照片不上传云端。
- 用户电脑 C 盘空间有限、E 盘充足。所有可选开发软件、依赖缓存、模型、构建和临时文件优先放 E 盘；工程固定在 E:\picturework。只有不能改位置的系统组件放 C 盘，并记录原因。不要清理 E 盘中与项目无关的文件。

## 当前实现

项目使用 Tauri 2 + React + TypeScript + Vite。界面已实现图片导入/拖放、证件照规格选择、裁切框拖动与缩放、JPEG 质量调节、原背景/白底/蓝底切换和 JPG 保存。当前预设像素为一寸 295×413、两寸 413×579、方形 600×600；这些是应用的起步像素预设，并不代表所有正式机构标准。当前没有 DPI 元数据或自定义规格功能。

本地 AI 使用 Transformers.js、Xenova MODNet 量化 ONNX 模型和 onnxruntime-web WASM。模型在 public/models/Xenova/modnet，env.allowRemoteModels=false；运行时资源由 Vite 打包。模型许可与来源见项目 MODEL_NOTICE.md。Rust 端实现用户 Pictures/拾光工坊默认路径与保存命令；Tauri Dialog 插件用于选择保存位置。首页及控件为中文。

## 已验证与限制

- npm run build：通过（TypeScript 检查及 Vite 生产构建）。
- cargo check --manifest-path src-tauri/Cargo.toml：通过。
- npm run tauri -- build：Windows release 构建通过；简体中文 MSI 成功生成：
  E:\picturework\shiguang-studio\src-tauri\target\release\bundle\msi\拾光工坊_0.1.0_x64_zh-CN.msi（12,304,384 字节，约 11.73 MiB）。主程序：src-tauri\target\release\shiguang-studio.exe。MSI 尚未安装。
- 浏览器前端用合成头像验证导入、规格切换、MODNet 推理、蓝底合成和 JPG 导出；一寸输出读回 295×413，蓝底左上像素约 #4387CE，两寸界面输出为 413×579。此测试只验证处理链路，不代表真人发丝/肩部边缘质量验收。
- 启动 release exe 后，Windows 辅助功能树确认中文界面加载及默认输出目录 C:\Users\Administrator\Pictures\拾光工坊。
- 原生图片选择器/保存对话框尚未完成实际点测：电脑上的 Keyboard Mouse Locker 曾拦截自动化鼠标键盘输入。不要擅自关闭用户的锁定软件；用户明确解锁或退出后再测试。
- 构建会提示前端 chunk 大于 500 KB；ONNX WASM 约 26.9 MB，是离线 AI 带来的安装体积成本。MSI 还没有在目标机器上安装后验收。默认照片目录在 C 盘用户 Pictures；保存对话框可选择 E 盘。

## 开发环境、安装和缓存位置

| 组件 | 版本 | 路径/说明 |
|---|---|---|
| Node.js / npm | 26.13.0 / 11.20.0 | E:\DevTools\Node\node-v26.13.0-win-x64；npm 缓存 E:\DevTools\npm-cache |
| Rust / Cargo | 1.99.1，stable-x86_64-pc-windows-msvc | RUSTUP_HOME=E:\DevTools\Rustup；CARGO_HOME=E:\DevTools\Cargo；crate 与构建缓存位于 Cargo home 的 E 盘 |
| Git Portable | 2.56.0.windows.4 | E:\DevTools\Git，PATH 使用 E:\DevTools\Git\cmd |
| VS Code | 1.145.0 | E:\DevTools\VSCode；用户数据 E:\DevTools\VSCodeData；扩展 E:\DevTools\VSCodeExtensions；Rust Analyzer 0.3.3105 |
| VS Build Tools 2022 | 17.14.44；MSVC 14.44.35207 | E:\DevTools\VSBuildTools；共享组件 E:\DevTools\VSShared；安装缓存 E:\DevTools\VSCache |
| Windows SDK | 10.0.26100.0 | C:\Program Files (x86)\Windows Kits\10，约 1.67 GB。VS 安装子组件使用系统默认位置，未手动搬动 |
| VS Installer | 17.14 系列 | C:\Program Files (x86)\Microsoft Visual Studio\Installer，约 0.14 GB |
| WebView2 Runtime | 156.0.4314.58 | 系统预装，无需重复安装 |
| WiX | 3.14.1 | 项目本地缓存 E:\picturework\shiguang-studio\src-tauri\target\.tauri\WixTools314；配置 useLocalToolsDir=true。旧 C 盘缓存仍留在用户 AppData 下，未清理 |
| 项目依赖/模型/构建 | — | 工程 E:\picturework\shiguang-studio；node_modules、dist、src-tauri\target、模型都位于 E 盘 |
| 安装器与临时文件 | — | E:\DevTools\Installers、E:\DevTools\Temp；不要清空整个目录 |

用户级环境变量已设置：RUSTUP_HOME=E:\DevTools\Rustup、CARGO_HOME=E:\DevTools\Cargo、TEMP/TMP=E:\DevTools\Temp、NPM_CONFIG_CACHE=E:\DevTools\npm-cache。C:\Users\Administrator\.npmrc 只有一行 cache 路径配置；npm 包缓存本身在 E 盘。用户 PATH 包含 Node、Cargo\bin、Git\cmd、VSCode\bin 及 copilot-shim 的 E 盘目录。新开的程序/终端才会完整继承更新后的用户环境变量。

Cargo 网络配置 E:\DevTools\Cargo\config.toml 将 http.proxy 设为空；原有环境会误用失效的 127.0.0.1:443。Rust MSVC 构建前需初始化 VS 环境：在 cmd 中调用 E:\DevTools\VSBuildTools\VC\Auxiliary\Build\vcvars64.bat。若新终端找不到 Rust，先设 RUSTUP_HOME、CARGO_HOME，并将 E:\DevTools\Cargo\bin 加入 PATH；不要重复安装。

最近空间核对：C 盘约 4.4 GB 可用、E 盘约 159.4 GB 可用（构建后会变化）。Windows SDK 和 VS Installer 是主要不可避免的 C 盘开发组件。安装 MSI 前需检查并明确选择安装盘；当前没有把应用安装到任何位置。

## 后续接手

1. 在用户解锁键鼠或允许自动化操作后，安装/运行 MSI，实测原生导入对话框、规格裁剪、背景处理和 Rust 保存流程；若用户可提供真人样照，再人工验收发丝与肩部边缘。
2. 先询问正式证件照规格/DPI 的目标依据，再增加对应尺寸、DPI 或自定义规格；不要把现有预设宣称为官方标准。
3. 修改后至少运行 npm run build、cargo check；重打 MSI 前关闭运行中的 exe，并通过 vcvars64.bat 初始化 MSVC。
4. 大型 WASM chunk 是已知性能/包体积提示；有需要再评估拆分和优化，保持离线模型路径可用。
5. 以后每个关键阶段继续更新本文件并由用户决定是否推送；旧 Flutter 记录保持独立。

## 关键源码

- 前端：src/App.tsx、src/App.css
- Rust/Tauri：src-tauri/src/lib.rs、src-tauri/tauri.conf.json、src-tauri/capabilities/default.json
- 项目说明与模型许可：README.md、MODEL_NOTICE.md
- 本地详细环境清单：E:\picturework\项目交接备忘录.md
