# codex-app-unofficial Linux patchset

这里保存本机用于调整 AUR `codex-app-unofficial` Linux 桌面版的 patch。

当前有效 patchset 基于 AUR `codex-app-unofficial 26.611.62324_launcher.29`。这个版本的 Codex Desktop 已经不再需要旧的左侧菜单/设置页重影修复，因此当前只维护桌宠修复。

这不是上游官方补丁，也不是长期自动兼容层。每次 AUR 或 Codex Desktop 上游更新后，都必须从干净工作树重新验证 patch 是否能无冲突应用，并确认 `app.asar` 内的 minified 锚点仍然唯一命中。

## 当前 Patch

当前有效文件：

```text
codex-app-unofficial-avatar-overlay-linux-26.611.62324.patch
```

用途：

- 给 AUR `PKGBUILD` 增加 `patch-avatar-overlay-linux.mjs`，并在 `prepare()` 阶段 patch `resources/app.asar`。
- 修复 Linux 下桌宠头像悬浮层的黑底、透明窗口背景、点击/穿透、窗口 shape 和动画播放逻辑。
- 保留已定好的非 idle 状态持久动画：不再播放三轮后退回 idle。
- 不包含左侧菜单/设置页重影修复。

当前哈希：

```text
codex-app-unofficial-avatar-overlay-linux-26.611.62324.patch
b76baba897000ceb643288c41491ff5b5df6da67107c8e3fd78371967b528680

patch-avatar-overlay-linux.mjs
eac04e6664c09e6154a01858e5529f29616133957280b8fe5fb9b165e197e9c8
```

## 使用方法

准备内层 AUR 仓库：

```bash
git clone https://aur.archlinux.org/codex-app-unofficial.git
cd codex-app-unofficial
```

应用当前 patch：

```bash
patch -p1 < ../codex-app-unofficial-avatar-overlay-linux-26.611.62324.patch
```

构建并安装：

```bash
makepkg -Cfsi
```

只验证构建、不安装：

```bash
makepkg -Cf
```

关键 bundle patch 后哈希：

```text
.vite/build/main-DLo8G5hp.js
9c85ae88845f6d1427a3a9f3169508dcb1474432b5ff946d9c98b317db886e37

webview/assets/avatar-overlay-native-page-Ct6lOFWD.js
821f80fe4bcab2a95a966183af7a7904d1938059b823897d3f28626206628d7b

webview/assets/avatar-overlay-native-frame-Bb-BcOSq.js
3e618bb4da8846e8d0ed1e7542f675d3edb9a1b501e309b9c6061a52d0cef4b0

webview/assets/codex-avatar-BYRhu0mv.js
ff0c2691e541387b95e6de3492947d55085f514e14858dfd43667fa4f2c488b7
```

本机运行时排查确认：

```text
桌面环境        XFCE / X11
X server        支持 Composite / SHAPE
桌宠窗口        Depth: 32
setShape        已生效
sprite alpha    TrueColorAlpha
```

## 常见问题

### patch 出现 `.rej`

不要继续 `makepkg`。

这通常表示当前 AUR `PKGBUILD` 或 `.SRCINFO` 已经和 patch 生成时不同，需要重新基于当前 AUR 版本生成 patch。

### patch 命令成功但安装后没有生效

检查 `makepkg` 日志里是否真的进入 `prepare()` 并执行 `patch-avatar-overlay-linux.mjs`。

如果只看到 `patching file patch-avatar-overlay-linux.mjs`，但 `PKGBUILD/.SRCINFO` 的 hunk 失败，那么脚本虽然落到了目录里，却没有被 AUR 构建流程调用，最终安装包不会包含修复。

## 维护提醒

每次上游 AUR 更新后，先从干净工作树验证：

```bash
git reset --hard
git clean -fd
patch -p1 < ../codex-app-unofficial-avatar-overlay-linux-26.611.62324.patch
makepkg -Cf
```

如果修改 `patch-avatar-overlay-linux.mjs`：

- 必须同步更新 `PKGBUILD` 和 `.SRCINFO` 中对应 SHA256。
- 必须重新生成外层 `.patch` 文件。
- 必须用干净临时副本验证 `patch -p1 --dry-run`。
- 必须从完整上游 tar 解包后实际运行脚本验证 `app.asar` 可被 patch。

如果 Codex Desktop 再次出现主窗口或设置页重影，应单独评估是否恢复或重写 `settings-overlap`。不要默认把它和桌宠 patch 绑回一起。
