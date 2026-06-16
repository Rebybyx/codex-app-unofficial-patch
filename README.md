# codex-app-unofficial Linux patchset

这里保存了我本机用于修复 `codex-app-unofficial` Linux 桌面版的一组 AUR patch。

当前 patchset 基于 AUR `codex-app-unofficial 26.609.30741_launcher.27` 调整并验证。它不是上游官方补丁，也不保证未来版本直接可用；每次 AUR 上游更新后，都应重新从干净工作树验证 patch 是否还能无冲突应用。

## Patch 列表

必须按顺序应用：

1. `codex-app-unofficial-settings-overlap.patch`
2. `codex-app-unofficial-avatar-overlay-linux.patch`

### settings-overlap

文件：`codex-app-unofficial-settings-overlap.patch`

用途：

- 修复 Linux 下 Codex Desktop 主窗口、renderer surface、设置页出现透明、透叠、重影的问题。
- 给 AUR `PKGBUILD` 增加 `patch-settings-overlap.mjs`，并在 `prepare()` 里 patch `resources/app.asar`。

环境敏感度：

- 相对不挑运行环境。
- 它主要修改 Electron 主窗口背景策略和前端 CSS token/root background，属于应用自身资源修复。
- 仍可能随 Codex Desktop 上游 bundle 名、minified 变量名、CSS 锚点变化而失效，所以它挑“Codex/AUR 版本”，但通常不强依赖具体桌面环境。

当前 hash：

```text
codex-app-unofficial-settings-overlap.patch
c4196038a4d2904dc2abb55d6282af8ee8a218e1fafa0a070fa17268cc7ffa2f

patch-settings-overlap.mjs
d4ae8564206d0e8ea3aba5973fcd103e4d2c30edb8acda5fc07166101541fae8
```

### avatar-overlay-linux

文件：`codex-app-unofficial-avatar-overlay-linux.patch`

用途：

- 修复 Linux 下头像悬浮层的透明视觉、点击、选择、拖拽、窗口形状同步相关问题。
- 依赖 settings patch 先写入共享 root background anchor，因此必须第二个应用。

环境敏感度：

- 比 settings patch 更挑运行环境。
- 它涉及 Electron 透明窗口、`setIgnoreMouseEvents`、Linux `BrowserWindow.setShape()`、窗口管理器和合成器行为。
- 这些行为在 X11/Wayland、不同 WM/DE、不同合成器、不同 Electron/Chromium 版本上可能不同。
- 当前仅确认在下面的“已验证环境”中可用；Wayland、GNOME、KDE、无合成器环境、外部合成器环境都需要重新验证。

当前 hash：

```text
codex-app-unofficial-avatar-overlay-linux.patch
6168ad78296df4e64337dd145240aa0825706e1dbaecd3ce5b932bb6c31efc45

patch-avatar-overlay-linux.mjs
a4f3f46cde7fa9038e6fe7cf7af75d5841147a641302df2106cf6d1dfb78f314
```

## 已验证环境

本机确认可用环境：

```text
发行版        Arch Linux
内核          linux 7.0.12.arch1-1 / 7.0.12-arch1-1
会话类型      X11
桌面环境      XFCE
显示          DISPLAY=:0.0
Wayland       未使用，WAYLAND_DISPLAY 为空
Xorg          xorg-server 21.1.23-1
XFCE session  xfce4-session 4.20.4-1
XFWM          xfwm4 4.20.0-2
XFWM compositor /general/use_compositing = true
GTK3          gtk3 1:3.24.52-1
libX11        libx11 1.8.13-1
libXext       libxext 1.3.7-1
已安装包      codex-app-unofficial 26.609.30741_launcher.27-2
```

未验证：

- Wayland 会话。
- GNOME Shell / Mutter。
- KDE Plasma / KWin。
- i3、bspwm、Hyprland 等其它窗口管理器。
- 禁用合成器的 X11 环境。
- 使用 picom 等外部合成器的 X11 环境。

## 使用方法

准备内层 AUR 仓库：

```bash
git clone https://aur.archlinux.org/codex-app-unofficial.git
cd codex-app-unofficial
```

应用两份 patch：

```bash
patch -p1 < ../codex-app-unofficial-settings-overlap.patch
patch -p1 < ../codex-app-unofficial-avatar-overlay-linux.patch
```

构建安装：

```bash
makepkg -Cfsi
```

如果只是验证构建，不安装：

```bash
makepkg -Cf
```

## 预期构建日志

`makepkg` 应该能看到：

```text
==> Starting prepare()...
Patched .../.vite/build/main-DS-bu3Xr.js
Patched .../webview/assets/app-D6IMMkHW.css
Patched .../webview/assets/settings-page-VwJS5VYK.js
Patched .../.vite/build/main-DS-bu3Xr.js
Avatar pointer policy old=0/new=1
Transparent visuals old=0/new=1
Avatar tray visibility store old=0/new=1
Avatar Linux window shape old=0/new=1
Patched .../webview/assets/avatar-overlay-native-page-ZObW9StG.js
Patched .../webview/assets/codex-avatar-C9vBe-eu.js
Avatar dynamic layout and priority anchors verified
```

文件名中的 hash 可能随上游版本变化；如果脚本报找不到锚点，需要重新适配 patch。

## 常见问题

### patch 出现 `.rej`

不要继续 `makepkg`。

这通常表示当前 AUR `PKGBUILD` / `.SRCINFO` 已经和 patch 生成时不同。需要重新基于当前 AUR 版本生成 patch。

### patch 命令成功但没有生效

检查 `makepkg` 日志里是否真的进入 `prepare()` 并执行两个 `.mjs`。

如果只看到 `patching file patch-xxx.mjs`，但 `PKGBUILD/.SRCINFO` 的 hunk 失败，那么脚本虽然落到了目录里，却没有被 AUR 构建流程调用，最终安装包不会包含修复。

### avatar patch 在其它桌面环境表现不同

这是可能的。

头像悬浮层修复依赖 Linux 透明窗口、输入穿透、窗口 shape、合成器和窗口管理器行为。若在 Wayland、GNOME、KDE 或其它 WM 下失效，优先记录：

```bash
uname -r
echo "$XDG_SESSION_TYPE"
echo "$XDG_CURRENT_DESKTOP"
echo "$DESKTOP_SESSION"
pacman -Q xorg-server xfce4-session xfwm4 gtk3 libx11 libxext 2>/dev/null
```

并检查 `patch-avatar-overlay-linux.mjs` 中的 minified 锚点是否仍能在当前 `app.asar` 中唯一命中。

## 维护提醒

- 这组 patch 是手工维护的 AUR patchset，不是长期自动兼容层。
- 每次上游 AUR 更新后，先从干净工作树验证：

```bash
git reset --hard
git clean -fd
patch -p1 < ../codex-app-unofficial-settings-overlap.patch
patch -p1 < ../codex-app-unofficial-avatar-overlay-linux.patch
makepkg -Cf
```

- 如果修改了 `.mjs` 脚本，必须同步更新 `PKGBUILD` 和 `.SRCINFO` 中对应 SHA256。
- `settings -> avatar` 顺序不能交换。
