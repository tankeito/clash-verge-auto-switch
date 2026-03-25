**中文** | [English](README.md)

---

# Clash Verge Auto Switch

> 在 macOS 上自动检测 Clash 选择器组并切换到最快的可用节点。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/Platform-macOS-lightgrey.svg)](https://apple.com/macos)
[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://python.org)
[![Clash](https://img.shields.io/badge/Clash-Verge%2F Mihomo-orange.svg)](https://github.com/clash-verge-rev/clash-verge-rev)

---

## 📖 项目简介

**Clash Verge Auto Switch** 是一个 macOS 原生自动化工具，可对 Clash Verge Rev 或 Mihomo 代理组进行速度测试，并自动将选择器组切换到延迟最低的健康节点。

与被动的健康检查工具不同，此技能主动从每个目标组中选择最快的工作代理，确保最佳网络性能，无需手动干预。

### ✨ 核心特性

- 🎯 **自动检测**：从实时 Clash 控制器发现当前活跃的选择器组
- ⚡ **速度测试**：使用控制器延迟 API 测量所有候选代理的延迟
- 🔄 **自动切换**：自动将选择器组切换到最快的健康节点
- 📅 **定时执行**：安装 macOS launchd 任务进行定期自动切换
- 🔧 **灵活配置**：支持显式组、组范围模式和自定义控制器 URL
- 🛡️ **安全操作**：干跑模式、控制器可达性检查、详细日志记录

### 🎯 使用场景

- 当前代理变慢时自动切换到最快节点
- 定期优化多个 Clash 选择器组（Proxy、ChatGPT 等）
- 诊断控制器连接问题
- 在 macOS 上设置免手动代理管理

---

## 🚀 快速开始

### 1. 安装

```bash
# 克隆或复制技能到 Codex skills 目录
mkdir -p ~/.codex/skills
cp -r clash-verge-auto-switch ~/.codex/skills/
```

### 2. 基本用法

使用默认设置运行主脚本：

```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py
```

默认情况下，它将：
- 从实时 Clash 控制器自动发现当前活跃的选择器链
- 测试发现组中的所有候选代理
- 将每个组切换到最快的健康节点

### 3. 高级选项

**检查发现的组**：
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py --list-groups
```

**指定显式组**：
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py \
  --group 'Proxy' \
  --group 'ChatGPT'
```

**干跑（仅测量，不切换）**：
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py --dry-run
```

**如果控制器离线则启动 Clash Verge**：
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py --launch-if-needed
```

---

## 📋 命令参考

### 主脚本：`switch_fastest.py`

#### 基本选项

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `--group <name>` | 要优化的选择器组（可重复） | 自动发现 |
| `--group-scope <scope>` | 如何自动发现组：`current`、`top-level`、`all` | `current` |
| `--controller-url <url>` | HTTP 控制器基础 URL（如 `http://127.0.0.1:9097`） | 自动发现 |
| `--unix-socket <path>` | 控制器 Unix socket 路径 | 自动发现 |
| `--secret <token>` | 控制器密钥令牌 | 从配置读取 |
| `--delay-url <url>` | 用于延迟测试的 URL | `http://1.1.1.1` |
| `--timeout-ms <ms>` | 延迟测试超时（毫秒） | `5000` |
| `--dry-run` | 仅测量和报告，不切换组 | `false` |
| `--list-groups` | 打印发现的目标组并退出 | `false` |
| `--expand-select-groups` | 递归展开嵌套选择器组 | `false` |
| `--launch-if-needed` | 放弃前尝试打开 Clash Verge | `false` |
| `--launch-wait <sec>` | 启动 Clash Verge 后等待的秒数 | `8` |

#### 组范围模式

| 模式 | 说明 |
|------|------|
| `current` | 跟随当前选择的链，优化活跃选择器 |
| `top-level` | 优化不嵌套在其他选择器下的组 |
| `all` | 优化除 `GLOBAL` 外的所有选择器组 |

### 安装脚本：`install_launch_agent.sh`

安装 macOS launchd 任务进行定期自动切换：

```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 30 \
  --group-scope current
```

#### 选项

| 选项 | 说明 | 必需 |
|------|------|------|
| `--interval-minutes <N>` | 执行间隔（分钟） | 是 |
| `--group <name>` | 传递给 switch_fastest.py（可重复） | 否 |
| `--group-scope <scope>` | 传递给 switch_fastest.py | 否 |
| 其他标志 | 任何有效的 switch_fastest.py 参数 | 否 |

#### 输出

```
Installed com.codex.clash-verge-auto-switch
Interval: 30 minute(s)
Plist: /Users/username/Library/LaunchAgents/com.codex.clash-verge-auto-switch.plist
Stdout: /Users/username/Library/Logs/clash-verge-auto-switch.log
Stderr: /Users/username/Library/Logs/clash-verge-auto-switch.err.log
```

### 卸载脚本：`uninstall_launch_agent.sh`

移除定时任务：

```bash
~/.codex/skills/clash-verge-auto-switch/scripts/uninstall_launch_agent.sh
```

---

## 🔧 配置

### 控制器发现

脚本按以下顺序发现 Clash 控制器：

1. **显式参数**：`--unix-socket` 或 `--controller-url`
2. **环境变量**：
   - `CLASH_API_UNIX_SOCKET`
   - `CLASH_API_URL`
   - `CLASH_API_SECRET`
3. **本地 Clash 配置**（YAML 解析）：
   - `external-controller-unix`
   - `external-controller`
   - `secret`

**注意**：脚本将 `set-your-secret` 视为未设置占位符，避免发送它。

### 配置文件路径

脚本检查这些常见的 Clash 配置位置：

```bash
~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/config.yaml
~/.config/clash/config.yaml
```

---

## 📊 工作流程

### 1. 控制器发现

```
用户参数 → 环境变量 → 配置文件 → 找到控制器
```

如果设置了 `--launch-if-needed` 且控制器不可达：
- 尝试启动 Clash Verge Rev 应用程序
- 等待指定秒数（默认：8 秒）
- 重试控制器发现

### 2. 组发现

当未提供 `--group` 时，脚本根据 `--group-scope` 发现目标组：

```
/proxies API 响应 → 解析组树 → 按范围过滤 → 目标组
```

**发现规则**：

| 范围 | 行为 |
|------|------|
| `current` | 从 GLOBAL 跟随选择链，优化活跃选择器 |
| `top-level` | 排除嵌套选择器，优化根级别组 |
| `all` | 包含除 GLOBAL 外的所有选择器组 |

### 3. 代理测试与切换

对每个目标组：

1. **展开候选**：将 `url-test`、`fallback`、`load-balance` 组解析为叶代理
2. **测量延迟**：使用控制器延迟 API 测试每个候选
3. **选择获胜者**：选择最低延迟的健康节点
4. **切换组**：向 `/proxies/{group}` 发送 PUT 请求，附带获胜者名称
5. **报告结果**：记录切换动作或干跑结果

---

## 📅 定时调度

### 为什么使用 launchd 而不是 Codex 自动化？

Codex 重复调度仅支持**小时间隔**。为了更精细的控制（如每 15 或 30 分钟），使用捆绑的 `launchd` 安装程序。

### 安装示例

**每 30 分钟，优化当前组**：
```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 30 \
  --group-scope current
```

**每 15 分钟，优化显式组**：
```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 15 \
  --group 'Proxy' \
  --group 'ChatGPT'
```

**每小时干跑（测试）**：
```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 60 \
  --dry-run
```

### 日志文件

安装后，检查日志以查看执行结果：

```bash
# 查看标准输出日志
tail -f ~/Library/Logs/clash-verge-auto-switch.log

# 查看错误日志
tail -f ~/Library/Logs/clash-verge-auto-switch.err.log
```

---

## 🛡️ 安全与最佳实践

### 干跑模式

在启用自动切换之前，使用 `--dry-run` 测试：

```bash
# 测试会发现哪些组
python3 switch_fastest.py --list-groups

# 测试延迟测量但不切换
python3 switch_fastest.py --dry-run

# 测试显式组
python3 switch_fastest.py --group 'Proxy' --dry-run
```

### 控制器可达性

如果控制器不可达：

1. 打开 Clash Verge Rev 应用程序
2. 检查 Clash 配置中的 external-controller：
   ```yaml
   external-controller: 127.0.0.1:9097
   secret: your-secret-token
   ```
3. 使用显式控制器 URL：
   ```bash
   python3 switch_fastest.py --controller-url http://127.0.0.1:9097 --secret your-secret
   ```
4. 启用自动启动：
   ```bash
   python3 switch_fastest.py --launch-if-needed
   ```

### 组选择指南

| 场景 | 推荐范围 |
|------|---------|
| 日常使用，优化活跃组 | `current`（默认） |
| 优化所有根级别组 | `top-level` |
| 完全优化，所有组 | `all` |
| 仅特定组 | `--group 'Proxy' --group 'ChatGPT'` |

---

## 📁 项目结构

```
clash-verge-auto-switch/
├── SKILL.md                          # 技能定义和元数据
├── README.md                         # 英文文档
├── README_ZH.md                      # 中文文档
├── agents/
│   └── openai.yaml                  # OpenAI 代理接口配置
├── references/
│   └── runtime-notes.md             # 控制器发现和组检测规则
├── scripts/
│   ├── switch_fastest.py            # 主速度测试和切换脚本
│   ├── install_launch_agent.sh      # macOS launchd 安装程序
│   └── uninstall_launch_agent.sh    # launchd 任务移除程序
└── LICENSE
```

### 文件说明

| 文件 | 用途 |
|------|------|
| `SKILL.md` | 技能定义、触发器和工作流文档 |
| `switch_fastest.py` | 用于延迟测试和组切换的核心 Python 脚本 |
| `install_launch_agent.sh` | 安装定期 launchd 任务的 Bash 脚本 |
| `uninstall_launch_agent.sh` | 移除定时任务的 Bash 脚本 |
| `runtime-notes.md` | 关于控制器发现和组规则的技术说明 |
| `openai.yaml` | OpenAI Codex 的代理接口配置 |

---

## 🔍 故障排查

### 控制器不可达

**症状**：脚本退出，显示 "Could not reach the Mihomo controller"

**解决方案**：
1. 打开 Clash Verge Rev 应用程序
2. 检查 Clash 配置中的 external-controller：
   ```yaml
   external-controller: 127.0.0.1:9097
   secret: your-secret-token
   ```
3. 使用显式控制器 URL：
   ```bash
   python3 switch_fastest.py --controller-url http://127.0.0.1:9097 --secret your-secret
   ```
4. 启用自动启动：
   ```bash
   python3 switch_fastest.py --launch-if-needed
   ```

### 未找到组

**症状**：脚本退出，显示 "No selector groups found to optimize"

**解决方案**：
1. 列出发现的组：
   ```bash
   python3 switch_fastest.py --list-groups
   ```
2. 尝试更广泛的范围：
   ```bash
   python3 switch_fastest.py --group-scope all
   ```
3. 指定显式组：
   ```bash
   python3 switch_fastest.py --group 'Proxy' --group 'ChatGPT'
   ```

### 代理不切换

**症状**：脚本报告 "no healthy proxies responded"

**解决方案**：
1. 增加超时：
   ```bash
   python3 switch_fastest.py --timeout-ms 10000
   ```
2. 使用不同的延迟 URL：
   ```bash
   python3 switch_fastest.py --delay-url http://www.google.com
   ```
3. 检查网络连接和代理状态

### launchd 任务未运行

**症状**：定时执行未发生

**解决方案**：
1. 检查任务状态：
   ```bash
   launchctl list | grep clash-verge
   ```
2. 查看日志：
   ```bash
   tail -f ~/Library/Logs/clash-verge-auto-switch.log
   ```
3. 重新安装任务：
   ```bash
   # 先卸载
   ./uninstall_launch_agent.sh
   # 然后重新安装
   ./install_launch_agent.sh --interval-minutes 30
   ```

---

## 📄 License

MIT License. 详见 [LICENSE](LICENSE)。

---

## 🤝 贡献

- **GitHub Issues**: https://github.com/tankeito/clash-verge-auto-switch/issues
- **仓库**: https://github.com/tankeito/clash-verge-auto-switch
- **邮箱**: tqd354@gmail.com

---

**Clash Verge Auto Switch** | macOS 自动化代理优化

**Developed by tankeito** | 2026
