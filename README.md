# gpui-x

[gpui](https://github.com/zed-industries/zed/tree/main/crates/gpui) 的独立发行版：自 [zed-industries/zed](https://github.com/zed-industries/zed) 抽离的 gpui crate 家族（含 macOS / Windows / Linux / wGPU / Web 平台后端），面向**不使用 zed 编辑器本体**的 gpui 应用。

- License: **Apache-2.0**（与上游一致，各 crate 目录内附带 LICENSE-APACHE）
- Upstream: zed-industries/zed，抽离基线 main @ `916fc2b8cb3a815cbef4a3b40e13081be72036b6`（2026-09-19）
- Zed 商标归 Zed Industries 所有；本仓库与其无隶属关系

## 与上游的差异

全部功能差异收在 [`patches/`](./patches) 下：一个主题一个补丁，按 `series` 文件顺序整组应用。序列做过完整重建验证——应用到原始抽离后，除发布打包层外与仓库当前内容逐字节一致，且每个前缀状态 `cargo check`（gpui-x / gpui-x-apple / gpui-x-wgpu / gpui-x-windows）通过：

| 补丁 | 内容 |
| --- | --- |
| `0001-gpui-paint-path-scaled-and-path-origin` | `Window::paint_path_scaled` / `paint_path_scaled_at`：预缩放 Path 的缓存直绘 API；`Path::origin` 零拷贝平移（几何只细分一次，每帧平移重画） |
| `0002-renderers-path-origin-sync-and-vertex-prealloc` | Metal / wgpu / DirectX 渲染端同步 origin 平移 + 顶点缓冲预分配 |
| `0003-metal-merge-adjacent-path-batches` | 相邻 path batch 合并为单次中间渲染 pass（多 order 场景的全窗 MSAA 中继从 N 次降为 1 次） |
| `0004-metal-path-msaa-8x` | path 中间渲染 MSAA 8x（设备不支持时回退 4x） |
| `0005-metal-path-pass-union-bounds-clip` | path 中间 pass 按 union bounds（本 pass 所有路径包围盒的并集）裁剪 |
| `0006-gpui-path-vertices-arc-sharing` | Path 顶点 `Arc<Vec<_>>` 共享（immediate mode 每帧的 path 克隆从整段 memcpy 变引用计数加一）+ `PathVertex` 去 `content_mask` 变 Copy |
| `0007-gpui-path-builder-stroke-rounded` | `PathBuilder::stroke_rounded`：圆帽圆角描边构造器 |
| `0008-gpui-cold-path-noop-shim` | `std::hint::cold_path` 换本地 no-op shim，摆脱对 nightly feature 的依赖 |
| `0009-macos-atomic-top-anchored-resize` | macOS 程序化 resize 保留左上角，并在同一 Core Animation transaction 内强制重绘，避免旧尺寸 drawable 被拉伸一帧 |

发布打包层（不属于功能补丁，发布时重做即可）：24 个发布成员改名 `gpui-x-*`、crate 间依赖切 registry、`crates/gpui_apple/shaders/scene.h` 快照 + `build.rs` 找不到兄弟 crate 源码时回退读取该快照。

## 消费方式

```toml
gpui = { version = "0.2", package = "gpui-x" }
# ⚠️ 文字渲染需要显式依赖 platform 并开启 font-kit：缺失时 gpui-x-macos
# 退化为 NoopTextSystem——排版正常但光栅全零、整窗无文字
gpui_platform = { version = "0.1", package = "gpui-x-platform", features = ["font-kit"] }
```

## 同步上游（重放补丁）

1. 从 zed 新基线抽离同名 crate 目录（保持 `crates/<name>` 布局；现有抽离范围：24 个 crate + `tooling/perf` + `assets/fonts` 单测字体子集）；
2. 按序重放功能补丁：`for p in $(cat patches/series); do git apply "patches/$p" || break; done`——新基线上下文漂移造成的失败，按该补丁的主题人工解决后继续；
3. `cargo check --workspace`，再上真机渲染验收（MSAA、中间 pass 合并只能肉眼确认）；
4. 发布前重做打包层：改名 `gpui-x-*`、依赖切 registry、更新 `scene.h` 快照并保留 build.rs 回退。

## 已知限制

- `assets/fonts/` 仅含 gpui 单测引用的字体；本仓库内 examples 需要完整 zed assets，未随仓分发；
