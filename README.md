# gpui-x

[gpui](https://github.com/zed-industries/zed/tree/main/crates/gpui) 的独立发行版：自 [zed-industries/zed](https://github.com/zed-industries/zed) 抽离的 gpui crate 家族（含 macOS / Windows / Linux / wGPU / Web 平台后端），面向**不使用 zed 编辑器本体**的 gpui 应用。

- License: **Apache-2.0**（与上游一致，各 crate 目录内附带 LICENSE-APACHE）
- Upstream: zed-industries/zed，抽离基线 main @ `916fc2b`
- Zed 商标归 Zed Industries 所有；本仓库与其无隶属关系

## 与上游的差异（2026-09）

1. `Window::paint_path_scaled` / `paint_path_scaled_at`：预缩放 Path 的缓存直绘 API，`Path::origin` 支持零拷贝平移——几何只细分一次，每帧平移重画；
2. Metal：相邻 path batch 合并为单次中间渲染 pass（多 order 场景的全窗 MSAA 中继从 N 次降为 1 次）；path 顶点缓冲预分配；
3. Path 中间渲染 MSAA **8x**（设备支持时；否则回退 4x）；
4. DirectX 渲染器同步 origin 偏移（该文件 cfg 门控于 Windows，未经本机编译验证）。

## 消费方式

```toml
[patch.crates-io]
gpui = { git = "https://github.com/beyondlex/gpui-x", branch = "main" }
```

⚠️ 还需**显式依赖 `gpui_platform` 并开启 `font-kit`**（gpui 仅在其 dev-dependencies 中启用该 feature 链，缺失时 gpui_macos 退化为 NoopTextSystem——排版正常但整窗无文字）：

```toml
gpui_platform = { git = "https://github.com/beyondlex/gpui-x", branch = "main", features = ["font-kit"] }
```

## 已知限制

- `assets/fonts/` 仅含 gpui 单测引用的字体；本仓库内 examples 需要完整 zed assets，未随仓分发；
- 同步上游 = 从 zed 重新抽离对应目录 + 重放上述补丁（补丁很小，见 git 历史）。
