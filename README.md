# 自留时间（PWA）

一个记录"一天中属于自己的时间"的轻量渐进式网页应用，数据全部存在手机/浏览器本地，不联网、不注册账号。

## 第一版功能（P0）

- **计时**：手动开始/结束一段"自己的时间"，结束时标注原因（自然结束 / 被打断 / 中途切走 / 手动收尾）
- **统计**：今日段数与累计时长、今日明细、近七天每日时长横条与合计
- **底线**：自定义自己的底线条款，可启用/停用、删除；预置三条示例
- **复盘**：按日期记录三行——做了什么 / 结果如何 / 下次改一句，历史复盘可回看

## 文件结构

```
self-time-pwa/
├── app.html               # 主页面（样式与脚本内联，原生 JS）
├── manifest.webmanifest   # PWA 清单：应用名、图标、独立窗口显示
├── sw.js                  # Service Worker：离线缓存
├── icons/                 # 应用图标 192 / 512 / maskable
└── README.md
```

## 本地调试

### 1. 起静态服务

PWA 的 Service Worker 不能通过 `file://` 打开，必须走 HTTP：

```powershell
python -m http.server 8088 --bind 0.0.0.0 --directory .
```

电脑浏览器访问 `http://127.0.0.1:8088/app.html`。

### 2. 手机访问（两种方式）

**方式 A：adb 反向转发（推荐，绕过校园网隔离和防火墙）**

手机用 USB 连上电脑并开启 USB 调试后：

```powershell
adb reverse tcp:8088 tcp:8088
```

手机浏览器打开 `http://127.0.0.1:8088/app.html`（这里的 127.0.0.1 指手机自身，流量经 USB 转发到电脑）。

**方式 B：局域网直连**

手机和电脑连同一 WiFi，电脑防火墙放行 8088 端口，手机访问 `http://<电脑局域网IP>:8088/app.html`。校园网若开启 AP 隔离，此方式可能不通。

### 3. 安装到主屏幕

Android 上用 Chrome / Edge 打开页面后：菜单（三个点）→ **添加到主屏幕** / **安装应用**。之后桌面会出现"自留时间"图标，点开全屏运行，可离线打开。

## 推送到 GitHub（远程仓库）

本地已初始化 git 仓库：分支 `main`，首次提交已完成，身份为 GitHub 账号 `SL1u0513`。

### 当前环境卡点（2026-11-18 实测）

- 内置 GitHub 连接器只有只读权限，不能建仓库或推送。
- 当前校园网下 `github.com` 主站 HTTPS 超时（被干扰），但 `api.github.com` 可达；`gh` 未登录、无缓存凭据、无 SSH 密钥。
- 因此远程推送需要先解决一条可写认证通道，二选一。

### 路径一：开代理后走浏览器授权（推荐）

1. 启动本机代理软件并打开系统代理（原残留配置为 `127.0.0.1:7890`），确认浏览器能打开 `https://github.com`。
2. 在项目目录执行：

```powershell
gh auth login --hostname github.com --git-protocol https --web
```

3. 复制终端显示的一次性码，浏览器打开 `https://github.com/login/device` 粘贴并授权。
4. 登录成功后建仓并推送：

```powershell
gh repo create self-time-pwa --public --source=. --remote=origin --push
```

### 路径二：用个人访问令牌（PAT）

1. 在能访问 GitHub 的环境下，头像 → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token，勾选 `repo` 权限，生成后复制。
2. 登录（令牌只走 `api.github.com`，当前网络可达）：

```powershell
$env:GH_TOKEN="ghp_你的令牌"
gh auth login --with-token
```

3. 建仓并推送：

```powershell
gh repo create self-time-pwa --public --source=. --remote=origin --push
```

> 令牌等同于密码，不要写进代码、提交或截图；用完可在 GitHub 后台撤销。

### GitHub Pages（已上线）

仓库：https://github.com/SL1u0513/self-time-pwa （公开，默认分支 `main`）

Pages 已从 `main` 分支根目录启用，长期访问链接：

```
https://sl1u0513.github.io/self-time-pwa/app.html
```

该链接是 HTTPS，手机浏览器打开后即可"添加到主屏幕"，不再依赖 USB 转发。注意：免费 Pages 要求仓库公开。

> 若以后需要手动调整：仓库 → **Settings** → **Pages** → Source 选 `Deploy from a branch` → 分支 `main`、目录 `/ (root)` → **Save**。

## 数据说明

- 记录存在浏览器 IndexedDB 中，卸载浏览器、清除站点数据会一并删除，重要复盘可手动抄录。
- 计时以起止时间戳计算，页面被系统冻结不影响时长结果；重新打开页面会自动恢复未结束的时段。
- 本版无法自动检测你在其他 App 中的行为，所有计时靠手动开始/结束；后台保活与应用识别属于后续原生打包阶段的能力。

## 后续规划（未在本版实现）

- P1：游戏 1 小时时间盒与预警、睡眠独立计时线
- P2：本地照片 + 一句话日常记录
- 阶段二：Capacitor 打包 APK，前台服务保活、前台应用识别、系统通知
