# openai-codex-desktop Linux 桌宠补丁

这个仓库保存用于 AUR [`openai-codex-desktop`](https://aur.archlinux.org/packages/openai-codex-desktop) 的 Linux 桌宠兼容补丁。

补丁面向 Arch Linux、X11 和 XFCE 环境，修复 Codex Desktop 桌宠窗口无法响应鼠标、透明区域命中异常，以及缩放柄和会话气泡控件被窗口 shape 裁剪的问题。同时保留非 idle 动画持续循环逻辑。

这不是 OpenAI 或 AUR 上游官方补丁，也不是跨版本兼容层。Codex Desktop 或 AUR 包更新后，必须重新验证 minified bundle 锚点和运行时行为。

## 兼容版本

当前补丁只保证适用于以下原版 AUR 基线：

```text
AUR 仓库    https://aur.archlinux.org/openai-codex-desktop.git
AUR 提交    d0feb50bf1cbc7ee68ab5eeca2da1f60374f0036
原始版本    26.707.31428-1
补丁后版本  26.707.31428-4
Electron    42
```

已验证环境：

```text
发行版      Arch Linux
桌面环境    XFCE
显示协议    X11
X Server    支持 Composite / SHAPE
```

Wayland、其他桌面环境以及未来 AUR 版本尚未验证。

## 补丁文件

当前发布文件：

```text
openai-codex-desktop-avatar-overlay-linux-26.707.31428.patch
```

SHA-256：

```text
ccc8412118da3449df64b2fe2a55928a1511807dbb6f69ec31547acc54df777e
```

应用后新增的构建脚本：

```text
patch-linux-avatar-overlay.mjs
```

SHA-256：

```text
2d37573062c38ae3443d38a5608cf87fa725cc43219b8f23680323d079cd76c8
```

## 修复内容

- Linux 下保持桌宠窗口可响应鼠标点击和拖动，不再依赖 X11 无效的鼠标事件转发行为。
- 根据精灵图透明度生成桌宠窗口 shape，减少透明矩形区域拦截桌面点击。
- 将 `48x48` 缩放悬停区和通知徽标加入窗口 shape，恢复缩放柄与会话气泡收起按钮。
- 桌宠状态、控件增删或位置变化时重新计算并同步 shape。
- 保留旧补丁中的非 idle 动画持续循环逻辑，不再播放三轮后回到 idle 动画。
- 使用精确且唯一的 minified bundle 锚点；上游结构变化时构建会直接失败，避免静默生成未修复的软件包。

AUR 包原有的 Linux 启动、背景和原生模块构建逻辑保持不变。本补丁不会额外替换这些已有功能。

## 使用方法

克隆原版 AUR 仓库：

```bash
git clone https://aur.archlinux.org/openai-codex-desktop.git
cd openai-codex-desktop
```

先检查补丁能否干净应用：

```bash
git apply --check ../openai-codex-desktop-avatar-overlay-linux-26.707.31428.patch
```

应用补丁：

```bash
git apply ../openai-codex-desktop-avatar-overlay-linux-26.707.31428.patch
```

构建并安装：

```bash
makepkg -Cfsi
```

安装前应彻底退出正在运行的 Codex。Electron 不会热加载新安装的 `app.asar`。

只构建、不安装：

```bash
makepkg -Cfs
```

## 已完成验证

- 从 AUR 远端重新克隆干净仓库后，`git apply --check` 和 `git apply --index` 均通过。
- 补丁应用后的 `PKGBUILD`、`.SRCINFO` 和 `patch-linux-avatar-overlay.mjs` 与实测构建版本逐字一致。
- `makepkg --verifysource`、完整 `makepkg` 构建和 `.SRCINFO` 再生成检查通过。
- 最终包内确认包含鼠标交互、动态 shape、缩放悬停区、通知徽标和动画循环修改。
- XFCE/X11 运行时已确认桌宠能够点击、拖动和缩放，会话气泡控件不再被裁剪。
- 连续 8 秒画面采样确认 idle 动画持续循环。

本地验证构建包信息：

```text
openai-codex-desktop-26.707.31428-4-x86_64.pkg.tar.zst
SHA-256: 803d524c64947669906eeb6ba74576b438c20b0794b38deb92d97420e68f9f24
```

构建包未上传到本仓库；上述哈希仅用于复核同版本本地构建结果。

## 常见问题

### `git apply --check` 失败

不要强制应用，也不要继续构建。先确认 AUR 仓库处于干净状态，并核对其提交和 `pkgver` 是否与兼容版本一致。

如果 AUR 已更新，需要基于新版本重新定位 bundle、调整精确锚点并完整回归测试。

### 安装后仍然运行旧版本

检查已安装包版本：

```bash
pacman -Q openai-codex-desktop
```

目标版本应为：

```text
openai-codex-desktop 26.707.31428-4
```

确认已完全退出旧进程，再重新启动 Codex。

### 动画像是停止了

上游将 idle 动画帧时长统一放大了 6 倍，一个完整循环约为 6.6 秒，因此静止时间较明显。补丁保持这一上游速度，只让非 idle 状态持续循环。

### 构建时提示锚点数量不正确

补丁脚本要求每个原始锚点唯一命中。数量为 0 或大于 1 表示上游 bundle 已变化，当前补丁不再兼容；不要通过放宽检查绕过失败。

## 更新维护

每次 AUR 或 Codex Desktop 更新后，应在新的临时克隆中执行：

```bash
git clone https://aur.archlinux.org/openai-codex-desktop.git openai-codex-desktop-check
cd openai-codex-desktop-check
git apply --check ../openai-codex-desktop-avatar-overlay-linux-26.707.31428.patch
```

若修改 `patch-linux-avatar-overlay.mjs`：

- 同步更新 `PKGBUILD` 和 `.SRCINFO` 中的 SHA-256。
- 重新生成外层 `.patch` 文件。
- 在干净 AUR 克隆中验证完整 patch 与 staged diff 一致。
- 从上游应用归档重新解包并运行补丁脚本。
- 完整构建安装包，并在 X11 中复测点击、拖动、缩放、气泡和动画。

## 许可与声明

补丁脚本使用 `0BSD` SPDX 标识。Codex Desktop、Electron 及 AUR 包中的其他文件遵循各自上游许可。
