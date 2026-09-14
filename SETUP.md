# Pi 配置清单（给 AI 执行）

> 本文件的**执行者是一台机器上的 AI 助手**，不是人。人类请回 [README.md](README.md)。

---

## 交给执行者（AI）的元指令

在动手之前，**先把本文件从头到尾读完**，然后遵守以下五条：

1. **先计划后执行。** 通读本文件、`settings.template.json`、`APPEND_SYSTEM.md`、`package.json` 以及 `extensions/` 下 4 个 `.ts` 文件，然后输出一份改动计划：将创建/修改哪些文件、执行哪些命令、会覆盖用户哪些已有配置。**等用户确认后再动手。**
2. **逐步验证。** 每步末尾都有「验证」，不通过就**停下来报告**，不要跳到下一步。
3. **最小侵入 + 幂等。**
   - 改 `settings.json` 用**嵌套对象合并**，不许整体覆盖用户的其他键；
   - 与用户已有值冲突的键，逐条列出交用户决定，**不要静默覆盖**；
   - 不读、不写、不打印 `auth.json`（内含 API 凭据）。
4. **平台适配。** 文中 `~` 在 Windows 上指 `%USERPROFILE%`，路径分隔符按当前平台处理。命令若在 Windows 下不可用，优先用 Git Bash（Pi 在 Windows 默认走 `C:\Program Files\Git\bin\bash.exe`）。
5. **不越界。** 不安装本清单未列出的软件，不改系统 PATH，不动其他仓库（第 6 步列出的 pi-zh 除外），不执行任何数据库变更。

---

## 0. 环境自检

先确认前置条件全部满足：

| # | 检查项 | 命令 | 期望 |
|---|--------|------|------|
| 0.1 | Node.js | `node -v` | `v22` 或更高 |
| 0.2 | npm | `npm -v` | 能正常输出 |
| 0.3 | Git | `git --version` | 能正常输出 |
| 0.4 | Pi 是否已装 | `pi --version` | 有版本号；报「找不到命令」则进第 1 步 |
| 0.5 | Windows 专属 | 检查 `C:\Program Files\Git\bin\bash.exe` 是否存在 | 存在。不存在则让用户先装 Git for Windows（**硬性前置**，Pi 依赖它执行命令） |

**验证**：0.1–0.4 全部通过方可继续。0.5 仅 Windows 需要。

---

## 1. 安装 Pi

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

**验证**：

```bash
pi --version
```

输出失败且提示命令不存在 → Windows 上加 PATH 环境变量 `%APPDATA%\npm`，重开终端再试；不要自行修改系统级配置，让用户来做。

---

## 2. 安装扩展包

```bash
pi install git:github.com/jiangnanquan/pi-team-setup@main
```

该包**零依赖**，Pi 只做一次 git clone，不会触发 `npm install`。

**验证**：

```bash
pi list                     # 应出现 pi-team-setup
```

---

## 3. 安装插件（9 个，版本已锁定）

逐条执行，**不要省略版本号**（省略会被 `pi update --extensions` 升到新版，导致各人环境不一致）：

```bash
pi install npm:pi-powerline-footer@0.17.1
pi install npm:pi-subagents@0.67.0
pi install npm:pi-background-tasks@2.5.0
pi install npm:pi-cc-extensions@0.8.70
pi install npm:cc-safety-net@2.3.4
pi install npm:pi-deepseek-search@1.0.20
pi install npm:@bacnh85/pi-deepseek-tools@0.13.1
pi install npm:@juicesharp/rpiv-todo@2.9.0
pi install npm:@juicesharp/rpiv-ask-user-question@2.9.0
```

**验证**：

```bash
pi list
```

输出中应包含上面 9 个包加上第 2 步的 `pi-team-setup`，共 10 条。缺哪条就单独重跑那条命令，报告失败原文。

---

## 4. 合并标准配置

### 4.1 合并规则（务必按此执行）

1. 读取 `~/.pi/agent/settings.json`；文件不存在则视为 `{}`（先不要创建）。
2. 把 `settings.template.json` 的每个**叶子键**与用户现有值对比：
   - 用户没有该键 → 加入
   - 用户有且值相同 → 跳过
   - 用户有且值不同 → 记入**冲突清单**，暂不改动
3. `powerline` 是深层嵌套对象，**逐叶子键合并**。
4. `powerline.disabledSegments` 与 `powerline.customItems` 是**数组，整体替换而不是合并** —— 若用户已有自定义项，这两项必进冲突清单交用户决定。
5. 写入前把最终 diff 展示给用户；写入前先备份为 `settings.json.bak-<时间戳>`。

### 4.2 模板内容说明

| 键 | 用途 |
|----|------|
| `modelThinkingLevels` | 统一思考等级（`deepseek-flash` → `max`） |
| `markdown.mermaid` | Mermaid 图流式渲染 |
| `enableSkillCommands` | 启用 `/skill:xxx` 命令 |
| `showHardwareCursor` | 中文输入法光标可见（**中文环境必需**） |
| `collapseChangelog` / `quietStartup` | 精简启动输出 |
| `powerline.*` | 状态栏布局与 4 个扩展段位的注册（**不合并此项，扩展段位不会显示**） |

**模板中刻意不包含**：`theme`、`tuiMode`、`defaultProvider`、`defaultModel`、`hideThinkingBlock`、`shellPath`、`npmCommand`、`externalEditor`、`terminal.*` —— 这些属于个人审美或个人终端能力，覆盖会破坏使用者体验。

**验证**：合并后重新读取 `settings.json`，确认能解析为合法 JSON，且用户在冲突清单中已逐条拍板。

---

## 5. 写入通用提示追加

把本仓库的 `APPEND_SYSTEM.md` 复制到 `~/.pi/agent/APPEND_SYSTEM.md`。

- 若目标文件不存在 → 直接复制。
- 若已存在 → 展示双方差异，让用户决定是覆盖、追加还是跳过。

> 注意：该文件对**所有项目**生效，会改变 Pi 的中文输出与 Markdown 层级偏好。

**验证**：文件存在且能被读取；内容与本仓库副本一致（或按用户决定处理）。

---

## 6. 安装中文界面汉化（pi-zh）

把 [pi-zh](https://github.com/jiangnanquan/pi-zh) 拉到本机，执行三条互相独立的汉化线。

**本步是增强项，允许跳过。** 前置不满足或版本不匹配时，跳过并报告，**不要 `--force` / `--allow-missing` 硬闯**，不影响第 7、8 步。

**前置检查**：`python3 --version` 能正常输出（A 线、C 线的引擎依赖）。缺失则跳过本步，并告知用户「汉化需要 Python 3（Windows 上只有 `python` 而没有 `python3` 时同样先跳过），其余配置不受影响」。

```bash
# 6.1 克隆汉化仓
git clone https://github.com/jiangnanquan/pi-zh ~/pi-zh
cd ~/pi-zh

# 6.2 A 线：CLI / TUI 汉化（命令名保持英文；写盘前自动留存 .zh-backup 干净基底）
bash scripts/apply_patch.sh

# 6.3 B 线：第三方插件简介汉化（运行时覆盖，不改任何插件源码）
bash scripts/install_plugin_i18n.sh

# 6.4 C 线：插件渲染文案汉化（欢迎页等）
bash scripts/apply_plugin_ui.sh
```

三条线各自独立，均可单独还原：

```bash
bash scripts/apply_patch.sh --restore           # A 线：还原官方英文
bash scripts/install_plugin_i18n.sh --uninstall # B 线：移除软链，插件简介恢复英文
bash scripts/apply_plugin_ui.sh --restore       # C 线：还原插件英文
```

**验证**：

```bash
pi --help
```

说明文字应为简体中文，而 `--help`、`--version` 等参数与所有命令名保持英文。

**失败处理**（按此降级，不要自作主张）：

| 现象 | 处置 |
|------|------|
| A 线报「当前版本 vX.Y.Z 不在已知适配清单」 | Pi 版本超前于汉化适配，属预期情形。**跳过 A 线**，继续 B / C 线，最终报告写明「CLI 汉化待上游适配（本机 vX.Y.Z）」 |
| A 线报 `python3` 缺失 | 跳过整步（见前置检查） |
| B 线报「链接已存在且不是本项目软链」 | 用户 `~/.pi/agent` 下已有同名文件或软链。**停下报告**，由用户决定是否加 `--force`，不要代拍板 |
| C 线严格模式报「未命中 / 拒绝写盘」或找不到插件包 | 字典适配版本与本机插件版本不一致，或第 3 步未装齐。**跳过 C 线**并报告 |

**范围边界**：只执行上面 4 条命令。不修改 `~/pi-zh` 里的 `i18n/*.json` 与 `scripts/*`；发现翻译问题，让用户到上游仓库反馈。

---

## 7. 必须由用户亲自完成的（AI 不能代劳）

停下并明确告知用户以下两件事：

1. **登录模型**：在 Pi 里执行 `/login` → 选 DeepSeek → 粘贴**自己的** API key。凭据落在本机 `~/.pi/agent/auth.json`，不要帮用户填报、不要读取该文件、不要试图用他人 key 代替。
2. **终端字体**：状态栏含 `⚡`/`¥`/进度条等字符，Windows Terminal 需装 Nerd Font 并设置为终端字体，否则显示豆腐块。

**可选**：若要关闭 DeepSeek 的会话亲和头，创建 `~/.pi/agent/models.json`：

```json
{
  "providers": {
    "deepseek": {
      "compat": {
        "sendSessionAffinityHeaders": false
      }
    }
  }
}
```

此文件**只能放在全局位置**，Pi 不支持项目级 `models.json`。用户若已有该文件，合并 `providers.deepseek.compat` 一层即可。

---

## 8. 最终验收

逐项执行，全部通过才算完成：

| # | 验收项 | 命令 / 观察 | 期望 |
|---|--------|------------|------|
| 8.1 | Pi 可用 | `pi --version` | 有版本号 |
| 8.2 | 包全部就位 | `pi list` | 10 条（1 个 git 包 + 9 个 npm 包） |
| 8.3 | 扩展已加载 | `pi config` 或启动 Pi 查看扩展列表 | 4 个扩展 `context-bar` / `cache-hit` / `ds-balance` / `tps-status` 均为启用状态 |
| 8.4 | 状态栏段位 | 启动 `pi`，观察底部 | 出现进度条 `[====------] x%`、`¥余额`、`CH x%`、`⚡x tok/s` |
| 8.5 | 中文偏好生效 | 在 Pi 里提问 | 输出为简体中文 |
| 8.6 | 界面汉化 | `pi --help` | 说明文字为简体中文、命令名与参数仍为英文。第 6 步按其降级规则跳过的，此项记为「未生效（待上游适配）」写入报告，不算安装失败 |

**8.4 失败时的排查顺序**：
1. `pi-powerline-footer` 是否装上（第 3 步第一条）；
2. `settings.json` 里 `powerline.customItems` 是否完整（第 4 步）；
3. 终端是否为 Windows Terminal / 支持真彩色的终端。

---

## 附：本清单不做的事（如用户提出，请先确认再执行）

- 不改数据库连接、不跑任何 SQL
- 不安装/卸载本清单以外的 Pi 插件
- 不复制 `~/.pi/agent/npm/node_modules`（含平台原生二进制，跨机器复制必然损坏，让 Pi 自己装）
- 不动 `sessions/`、`trust.json`、`auth.json`
- 不执行 pi-zh 的懒人包安装（`bash scripts/install_bundle.sh`）：它是维护者本机环境的另一种分发渠道，与本清单第 2、3 步的扩展与插件高度重叠，重复执行会互相覆盖 `powerline` 配置
- 不改 pi-zh 的翻译字典与脚本（`i18n/*.json`、`scripts/*`）；翻译或补丁问题让用户到上游仓库反馈
