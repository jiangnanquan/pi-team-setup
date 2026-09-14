# 江南泉的 pi 配置

我自己的 [Pi](https://pi.dev) 配置打包：插件、状态栏扩展、界面设置、中文汉化。同事想装成一模一样的，照下面走一遍就行。

仓里只有安装步骤和配置，**没有任何业务数据、内网信息或 API key**。

## 怎么装（5 步）

### 1. 装 Node.js 22+ 和 Git for Windows

Pi 在 Windows 上默认用 Git Bash 跑命令，**Git for Windows 是硬性前置**。

```powershell
winget install OpenJS.NodeJS.LTS
winget install Git.Git
```

装完重开终端确认：

```bash
node -v      # 需要 ≥ v22
git --version
```

### 2. 装 Pi

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
pi --version
```

> `pi` 命令找不到：npm 全局目录没进 PATH，把 `%APPDATA%\npm` 加进去，重开终端。

### 3. 把这个仓拉到本地

```bash
git clone https://github.com/jiangnanquan/pi-team-setup ~/pi-team-setup
```

拉不动就重试一次，或者用 `gh repo clone jiangnanquan/pi-team-setup`。

### 4. 让 Pi 自己配置自己

```bash
cd ~/pi-team-setup
pi
```

然后对 Pi 说：

> 读取 `SETUP.md`，按它配置我的 Pi。**先输出完整改动计划，等我确认后再执行。**

`SETUP.md` 是写给 AI 看的清单：9 步，每步带验证命令，装不上会停下告诉你，不会硬闯。第 6 步会把界面汉化成简体中文（来自 [pi-zh](https://github.com/jiangnanquan/pi-zh)，命令名保持英文，可一键还原）。

### 5. 两件 AI 代替不了的事

- **登录模型**：在 Pi 里执行 `/login`，选 DeepSeek，粘贴**你自己的** API key。它只存在你本机 `~/.pi/agent/auth.json`，不会外传
- **终端字体**：状态栏有 `⚡`、`¥`、进度条这些字符，Windows Terminal 里要装个 Nerd Font 并设成字体，否则是豆腐块

## 装完长什么样

Pi 底部会多出这几段：

```
[====------] 2.6%   ← context-bar：上下文占用进度条
¥1664.54            ← ds-balance：DeepSeek 余额
CH 94.2% (累计 91%) ← cache-hit：prompt 缓存命中率
⚡ 42 tok/s         ← tps-status：实时生成速率
```

界面也是中文的：帮助、命令说明、设置菜单、插件简介与欢迎页都已汉化，命令名和参数仍是英文（来自 [pi-zh](https://github.com/jiangnanquan/pi-zh)）。想回官方英文：`cd ~/pi-zh && bash scripts/apply_patch.sh --restore`。

一起装的 9 个插件：`pi-powerline-footer`（状态栏底座）、`pi-subagents`（多 Agent 编排）、`pi-background-tasks`（后台任务）、`pi-cc-extensions`（Claude Code 风格 UI）、`cc-safety-net`（拦截破坏性命令）、`pi-deepseek-search`（联网搜索）、`@bacnh85/pi-deepseek-tools`（DeepSeek 工具调用修复）、`@juicesharp/rpiv-todo`、`@juicesharp/rpiv-ask-user-question`。

## 哪些配置跟着仓走，哪些得自己设

Pi 的「包」机制只能分发扩展和技能，**分发不了 `settings.json`** —— 所以有些东西得靠 AI 写进你自己的配置里。

| 内容 | 怎么来 |
|------|--------|
| 9 个插件 + 4 个扩展 | `pi install` 自动装，跟着仓走 |
| 状态栏布局、思考等级、中文偏好 | AI 按 `SETUP.md` 合并进 `settings.json` |
| 界面汉化（pi-zh） | AI 按 `SETUP.md` 第 6 步 clone + 打补丁，可 `--restore` 还原 |
| 主题、TUI 模式、默认模型、终端设置 | **我没放进去，你自己调**（覆盖了反而难受） |
| API key | 自己 `/login`，永不入仓 |

## 我改了配置之后，你们怎么更新

```bash
pi update --extensions   # 拉最新代码
pi                       # 重启生效
```

如果插件版本号变了（`SETUP.md` 第 3 步），要重跑对应的 `pi install npm:xxx@新版本`。

**升级 Pi 本体后汉化会掉** —— `npm install -g` 装上的新版会把汉化过的文件覆盖回英文，重跑一次就好：

```bash
cd ~/pi-zh && bash scripts/apply_patch.sh && bash scripts/install_plugin_i18n.sh && bash scripts/apply_plugin_ui.sh
```

第一条若报「不在已知适配清单」，说明汉化还没跟上这个 Pi 版本，等 pi-zh 更新后再跑（那之前界面是干净的官方英文）。

## 别往这个仓提交这些

公开仓，以下是红线：

| 别提交 | 原因 |
|--------|------|
| `~/.pi/agent/auth.json` | 里面有 API key |
| `~/.pi/agent/models.json` | 自定义 provider 时**可能内联 apiKey** |
| `~/.pi/agent/sessions/` | 会话记录，含你的对话和业务上下文 |
| `trust.json`、`npm/`、`git/`、`tasks/` | 本机状态，没意义 |
| 内网 IP、系统名、表名、业务口径 | 跟 Pi 配置无关 |

想分享自己的配置？只贴 `settings.json` 里界面相关的段落，贴之前肉眼过一遍有没有 key、个人路径、内网信息。

## 常见问题

**`pi` 命令找不到** —— npm 全局目录没进 PATH。Windows 加 `%APPDATA%\npm`；macOS/Linux 看 `npm config get prefix` 对应的 bin 目录。

**界面还是英文** —— 汉化没装，或被 Pi 升级覆盖了。到 `~/pi-zh` 重跑：CLI 汉化 `bash scripts/apply_patch.sh`；插件简介 `bash scripts/install_plugin_i18n.sh`；欢迎页等文案 `bash scripts/apply_plugin_ui.sh`。报「不在已知适配清单」说明汉化还没适配你当前的 Pi 版本。

**状态栏右边几段不显示** —— 依次查：`pi list` 里有没有 `pi-powerline-footer`；`settings.json` 里 `powerline.customItems` 齐不齐；终端是否支持真彩色（Windows Terminal 可以）。

**装插件报网络错误** —— 重跑失败的那条就行，`pi install` 可以重复执行。国内网络可以把 npm 源换成镜像。

**想让 Pi 也读我在别的工具里的 skills** —— 在 `~/.pi/agent/settings.json` 里加：

```json
{ "skills": ["~/.claude/skills"] }
```
