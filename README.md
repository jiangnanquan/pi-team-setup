# pi-team-setup

团队 [Pi](https://pi.dev) 环境引导包 —— 让每个人的 Pi 功能保持一致。

**只含 Pi 的安装步骤与标准配置，不含任何业务数据、内网信息或凭据。**

---

## 为什么有这个仓

各自配 Pi 的结果是：插件版本不一、状态栏五花八门、行为差异说不清。这个仓把「团队标准」固化成一份可执行的东西：

- **插件版本锁死** —— 9 个插件全部指定精确版本，不会有人升到新版后行为不一致
- **统一的状态栏** —— 4 个扩展补上 Pi 原生的信息盲区（上下文占用、缓存命中率、账户余额、生成速率）
- **一份配置，所有人相同** —— 思考等级、界面行为、中文交流偏好

新机器或新同事：照下面走一遍，几分钟得到和团队完全一致的 Pi。

## 组员怎么用（5 步）

### 1. 装 Node.js 22+ 与 Git for Windows

Pi 在 Windows 上默认用 Git Bash 执行命令，**Git for Windows 是硬性前置**。

```powershell
winget install OpenJS.NodeJS.LTS
winget install Git.Git
```

装完重开终端确认：

```bash
node -v      # 需 ≥ v22
git --version
```

### 2. 装 Pi

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
pi --version
```

> `pi` 命令找不到：npm 全局目录没进 PATH，把 `%APPDATA%\npm` 加进去后重开终端。

### 3. 把本仓拉到本地

```bash
git clone https://github.com/jiangnanquan/pi-team-setup ~/pi-team-setup
```

### 4. 让 Pi 自己完成配置

```bash
cd ~/pi-team-setup
pi
```

然后对 Pi 说：

> 读取 `SETUP.md`，按它配置我的 Pi。**先输出完整改动计划，等我确认后再执行。**

`SETUP.md` 是写给 AI 执行的清单：分 8 步，每步带验证命令，跑不通会停下报告而不是硬闯。

### 5. 两件 AI 代替不了的事（手工）

- **登录模型**：在 Pi 里执行 `/login`，选 DeepSeek，粘贴**你自己的** API key。凭据只落在本机 `~/.pi/agent/auth.json`，不会外传
- **终端字体**：状态栏含 `⚡`/`¥`/进度条等字符，Windows Terminal 需装一个 Nerd Font 并在设置里指定字体，否则显示豆腐块

## 配置分两层，各自归属不同

这是容易混淆的地方：**有些配置跟着仓库走，有些必须各人自己设**。

| 类型 | 具体内容 | 归属 | 原因 |
|------|---------|------|------|
| **插件与扩展** | 9 个 npm 插件 + 4 个状态栏扩展 | 跟随本仓，`pi install` 自动装 | 版本必须一致，否则行为不一致 |
| **团队配置** | 状态栏布局、思考等级、中文偏好 | 跟随本仓，由 AI 合并进 `settings.json` | Pi 的「包」机制**分发不了** `settings.json`，只能由 AI 或人写入 |
| **个人偏好** | 主题、TUI 模式、默认模型、终端设置 | **各人自己设，本仓刻意不碰** | 覆盖它们会破坏你的使用习惯 |
| **凭据** | API key | **各人自己 `/login`** | 永不入仓、永不代填 |

> Pi 没有「个人本地覆盖层」这回事，所以本仓只写团队必需项 —— 这是刻意的克制。

## 这个包里有什么

| 文件 | 作用 |
|------|------|
| `SETUP.md` | 给 AI 执行的配置清单（含验收命令），**核心文件** |
| `settings.template.json` | 待合并进 `~/.pi/agent/settings.json` 的团队标准配置 |
| `APPEND_SYSTEM.md` | 团队通用提示追加（中文交流偏好），写入 `~/.pi/agent/` |
| `package.json` | Pi 包清单（**零依赖**，clone 即用，不触发 `npm install`） |
| `extensions/` | 4 个状态栏扩展：`context-bar` / `cache-hit` / `ds-balance` / `tps-status` |

## 状态栏效果

配置完成后，Pi 底部会出现：

```
[====------] 2.6%   ← context-bar：上下文占用进度条
¥1664.54            ← ds-balance：DeepSeek 余额
CH 94.2% (累计 91%) ← cache-hit：prompt 缓存命中率
⚡ 42 tok/s         ← tps-status：实时生成速率
```

装齐的插件：`pi-powerline-footer`（状态栏底座）、`pi-subagents`（多 Agent 编排）、`pi-background-tasks`（后台任务）、`pi-cc-extensions`（Claude Code 风格 UI）、`cc-safety-net`（拦截破坏性命令）、`pi-deepseek-search`（联网搜索）、`@bacnh85/pi-deepseek-tools`（DeepSeek 工具调用修复）、`@juicesharp/rpiv-todo`、`@juicesharp/rpiv-ask-user-question`。

## 更新

维护者改了扩展或插件版本后，其他人执行：

```bash
pi update --extensions   # 拉取本仓最新代码
pi                       # 重启生效
```

若插件版本号变了（`SETUP.md` 第 3 步），需重跑对应的 `pi install npm:xxx@新版本` 命令。

## 红线：往这个仓库提交任何东西之前

本仓是**公开仓**，以下内容永不提交：

| 禁止提交 | 原因 |
|---------|------|
| `~/.pi/agent/auth.json` | 内含 API key |
| `~/.pi/agent/models.json` | 自定义 provider 时**可能内联 apiKey** |
| `~/.pi/agent/sessions/` | 会话记录，含你的对话与业务上下文 |
| `~/.pi/agent/` 下的 `trust.json`、`npm/`、`git/`、`tasks/` | 本机状态，无分享价值 |
| 内网 IP、内部系统名、表名、业务口径 | 与 Pi 配置无关 |

想分享自己的配置？只贴 `settings.json` 里与界面/状态栏相关的段落，贴之前肉眼过一遍有没有 key、个人路径、内网信息。

## 常见问题

**`pi` 命令找不到** —— npm 全局目录没进 PATH。Windows 加 `%APPDATA%\npm`，macOS/Linux 检查 `npm config get prefix` 对应的 bin 目录。

**状态栏右侧几段不显示** —— 依次检查：`pi list` 里有没有 `pi-powerline-footer`；`settings.json` 里 `powerline.customItems` 是否完整；终端是否支持真彩色（Windows Terminal 可以）。

**装插件时报网络错误** —— 重跑失败的那一条即可，`pi install` 是幂等的。国内网络下 `npm` 可考虑配镜像源。

**想让 Pi 也读我在别的工具里的 skills** —— 在 `~/.pi/agent/settings.json` 里加：

```json
{ "skills": ["~/.claude/skills"] }
```

---

**安全说明**：本仓内容仅限 Pi 的安装与配置，扩展代码由团队自己维护，仓库只有团队成员有合并权限。要改配置请只改自己机器的 `~/.pi/agent/settings.json`，不要往公共仓提未经讨论的配置。
