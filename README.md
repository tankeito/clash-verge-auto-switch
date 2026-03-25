# Clash Verge Auto Switch

> Automatically detect live Clash selector groups and switch to the fastest working node on macOS.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/Platform-macOS-lightgrey.svg)](https://apple.com/macos)
[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://python.org)
[![Clash](https://img.shields.io/badge/Clash-Verge%2F Mihomo-orange.svg)](https://github.com/clash-verge-rev/clash-verge-rev)

---

## 📖 Overview

**Clash Verge Auto Switch** is a macOS-native automation tool that speed-tests your Clash Verge Rev or Mihomo proxy groups and automatically switches selector groups to the lowest-latency healthy node.

Unlike passive health-check tools, this skill actively selects the fastest working proxy from each target group, ensuring optimal network performance without manual intervention.

### ✨ Key Features

- 🎯 **Auto-Detection**: Discovers currently active selector groups from live Clash controller
- ⚡ **Speed Test**: Measures latency for all candidate proxies using controller delay API
- 🔄 **Auto-Switch**: Switches selector groups to fastest healthy node automatically
- 📅 **Scheduled Execution**: Install macOS launchd job for periodic auto-switching
- 🔧 **Flexible Configuration**: Support explicit groups, group scopes, and custom controller URLs
- 🛡️ **Safe Operation**: Dry-run mode, controller reachability check, and detailed logging

### 🎯 Use Cases

- Automatically switch to fastest node when current proxy becomes slow
- Periodic optimization of multiple Clash selector groups (Proxy, ChatGPT, etc.)
- Diagnose controller connectivity issues
- Set up hands-free proxy management on macOS

---

## 🚀 Quick Start

### 1. Installation

```bash
# Clone or copy the skill to your Codex skills directory
mkdir -p ~/.codex/skills
cp -r clash-verge-auto-switch ~/.codex/skills/
```

### 2. Basic Usage

Run the main script with default settings:

```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py
```

By default, it will:
- Auto-discover current active selector chain from live Clash controller
- Test all candidate proxies in discovered groups
- Switch each group to the fastest healthy node

### 3. Advanced Options

**Inspect discovered groups**:
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py --list-groups
```

**Target explicit groups**:
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py \
  --group 'Proxy' \
  --group 'ChatGPT'
```

**Dry-run (measure only, no switching)**:
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py --dry-run
```

**Launch Clash Verge if controller is offline**:
```bash
/usr/bin/python3 ~/.codex/skills/clash-verge-auto-switch/scripts/switch_fastest.py --launch-if-needed
```

---

## 📋 Command Reference

### Main Script: `switch_fastest.py`

#### Basic Options

| Option | Description | Default |
|--------|-------------|---------|
| `--group <name>` | Selector group to optimize (repeatable) | Auto-discover |
| `--group-scope <scope>` | How to auto-discover groups: `current`, `top-level`, `all` | `current` |
| `--controller-url <url>` | HTTP controller base URL (e.g., `http://127.0.0.1:9097`) | Auto-discover |
| `--unix-socket <path>` | Unix socket path for controller | Auto-discover |
| `--secret <token>` | Controller secret token | From config |
| `--delay-url <url>` | URL used for delay testing | `http://1.1.1.1` |
| `--timeout-ms <ms>` | Delay test timeout in milliseconds | `5000` |
| `--dry-run` | Measure and report only; do not switch groups | `false` |
| `--list-groups` | Print discovered target groups and exit | `false` |
| `--expand-select-groups` | Recursively expand nested selector groups | `false` |
| `--launch-if-needed` | Try to open Clash Verge before giving up | `false` |
| `--launch-wait <sec>` | Seconds to wait after launching Clash Verge | `8` |

#### Group Scope Modes

| Mode | Description |
|------|-------------|
| `current` | Follow currently selected chain, optimize actively selected groups |
| `top-level` | Optimize selector groups not nested under another selector |
| `all` | Optimize every selector group except `GLOBAL` |

### Installation Script: `install_launch_agent.sh`

Install a macOS launchd job for periodic auto-switching:

```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 30 \
  --group-scope current
```

#### Options

| Option | Description | Required |
|--------|-------------|----------|
| `--interval-minutes <N>` | Execution interval in minutes | Yes |
| `--group <name>` | Passed to switch_fastest.py (repeatable) | No |
| `--group-scope <scope>` | Passed to switch_fastest.py | No |
| Other flags | Any valid switch_fastest.py arguments | No |

#### Output

```
Installed com.codex.clash-verge-auto-switch
Interval: 30 minute(s)
Plist: /Users/username/Library/LaunchAgents/com.codex.clash-verge-auto-switch.plist
Stdout: /Users/username/Library/Logs/clash-verge-auto-switch.log
Stderr: /Users/username/Library/Logs/clash-verge-auto-switch.err.log
```

### Uninstallation Script: `uninstall_launch_agent.sh`

Remove the scheduled job:

```bash
~/.codex/skills/clash-verge-auto-switch/scripts/uninstall_launch_agent.sh
```

---

## 🔧 Configuration

### Controller Discovery

The script discovers the Clash controller in the following order:

1. **Explicit arguments**: `--unix-socket` or `--controller-url`
2. **Environment variables**:
   - `CLASH_API_UNIX_SOCKET`
   - `CLASH_API_URL`
   - `CLASH_API_SECRET`
3. **Local Clash config** (YAML parsing):
   - `external-controller-unix`
   - `external-controller`
   - `secret`

**Note**: The script treats `set-your-secret` as an unset placeholder and avoids sending it.

### Configuration File Paths

The script checks these common Clash config locations:

```bash
~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/config.yaml
~/.config/clash/config.yaml
```

---

## 📊 Workflow

### 1. Controller Discovery

```
User Arguments → Environment Variables → Config Files → Controller Found
```

If `--launch-if-needed` is set and controller is unreachable:
- Attempt to launch Clash Verge Rev application
- Wait specified seconds (default: 8s)
- Retry controller discovery

### 2. Group Discovery

When `--group` is not provided, the script discovers target groups based on `--group-scope`:

```
/proxies API Response → Parse Group Tree → Filter by Scope → Target Groups
```

**Discovery Rules**:

| Scope | Behavior |
|-------|----------|
| `current` | Follow selected chain from GLOBAL, optimize active selectors |
| `top-level` | Exclude nested selectors, optimize root-level groups |
| `all` | Include all selector groups except GLOBAL |

### 3. Proxy Testing & Switching

For each target group:

1. **Expand candidates**: Resolve `url-test`, `fallback`, `load-balance` groups into leaf proxies
2. **Measure delay**: Test each candidate with controller delay API
3. **Select winner**: Choose lowest-latency healthy node
4. **Switch group**: PUT request to `/proxies/{group}` with winner name
5. **Report result**: Log switching action or dry-run result

---

## 📅 Scheduling

### Why launchd Instead of Codex Automations?

Codex recurring schedules only support **hourly intervals**. For finer control (e.g., every 15 or 30 minutes), use the bundled `launchd` installer.

### Installation Examples

**Every 30 minutes, optimize current groups**:
```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 30 \
  --group-scope current
```

**Every 15 minutes, optimize explicit groups**:
```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 15 \
  --group 'Proxy' \
  --group 'ChatGPT'
```

**Every hour with dry-run (testing)**:
```bash
~/.codex/skills/clash-verge-auto-switch/scripts/install_launch_agent.sh \
  --interval-minutes 60 \
  --dry-run
```

### Log Files

After installation, check logs for execution results:

```bash
# View stdout log
tail -f ~/Library/Logs/clash-verge-auto-switch.log

# View error log
tail -f ~/Library/Logs/clash-verge-auto-switch.err.log
```

---

## 🛡️ Safety & Best Practices

### Dry-Run Mode

Before enabling automatic switching, test with `--dry-run`:

```bash
# Test what groups would be discovered
python3 switch_fastest.py --list-groups

# Test delay measurement without switching
python3 switch_fastest.py --dry-run

# Test with explicit groups
python3 switch_fastest.py --group 'Proxy' --dry-run
```

### Controller Reachability

If the controller is unreachable:

1. Ensure Clash Verge Rev is running
2. Check external-controller settings in Clash config
3. Verify secret token matches
4. Use `--launch-if-needed` for auto-launch

### Group Selection Guidelines

| Scenario | Recommended Scope |
|----------|------------------|
| Daily use, optimize active groups | `current` (default) |
| Optimize all root-level groups | `top-level` |
| Full optimization, all groups | `all` |
| Specific groups only | `--group 'Proxy' --group 'ChatGPT'` |

---

## 📁 Project Structure

```
clash-verge-auto-switch/
├── SKILL.md                          # Skill definition and metadata
├── README.md                         # English documentation
├── README_ZH.md                      # Chinese documentation
├── agents/
│   └── openai.yaml                  # OpenAI agent interface config
├── references/
│   └── runtime-notes.md             # Controller discovery and group detection rules
├── scripts/
│   ├── switch_fastest.py            # Main speed test and switch script
│   ├── install_launch_agent.sh      # macOS launchd installer
│   └── uninstall_launch_agent.sh    # launchd job remover
└── LICENSE
```

### File Descriptions

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill definition, triggers, and workflow documentation |
| `switch_fastest.py` | Core Python script for delay testing and group switching |
| `install_launch_agent.sh` | Bash script to install periodic launchd job |
| `uninstall_launch_agent.sh` | Bash script to remove scheduled job |
| `runtime-notes.md` | Technical notes on controller discovery and group rules |
| `openai.yaml` | Agent interface configuration for OpenAI Codex |

---

## 🔍 Troubleshooting

### Controller Unreachable

**Symptoms**: Script exits with "Could not reach the Mihomo controller"

**Solutions**:
1. Open Clash Verge Rev application
2. Check external-controller in Clash config:
   ```yaml
   external-controller: 127.0.0.1:9097
   secret: your-secret-token
   ```
3. Use explicit controller URL:
   ```bash
   python3 switch_fastest.py --controller-url http://127.0.0.1:9097 --secret your-secret
   ```
4. Enable auto-launch:
   ```bash
   python3 switch_fastest.py --launch-if-needed
   ```

### No Groups Found

**Symptoms**: Script exits with "No selector groups found to optimize"

**Solutions**:
1. List discovered groups:
   ```bash
   python3 switch_fastest.py --list-groups
   ```
2. Try broader scope:
   ```bash
   python3 switch_fastest.py --group-scope all
   ```
3. Specify explicit groups:
   ```bash
   python3 switch_fastest.py --group 'Proxy' --group 'ChatGPT'
   ```

### Proxies Not Switching

**Symptoms**: Script reports "no healthy proxies responded"

**Solutions**:
1. Increase timeout:
   ```bash
   python3 switch_fastest.py --timeout-ms 10000
   ```
2. Use different delay URL:
   ```bash
   python3 switch_fastest.py --delay-url http://www.google.com
   ```
3. Check network connectivity and proxy status

### launchd Job Not Running

**Symptoms**: Scheduled execution not happening

**Solutions**:
1. Check job status:
   ```bash
   launchctl list | grep clash-verge
   ```
2. View logs:
   ```bash
   tail -f ~/Library/Logs/clash-verge-auto-switch.log
   ```
3. Reinstall job:
   ```bash
   # Uninstall first
   ./uninstall_launch_agent.sh
   # Then reinstall
   ./install_launch_agent.sh --interval-minutes 30
   ```

---

## 📄 License

MIT License. See [LICENSE](LICENSE) for details.

---

## 🤝 Contributing

- **GitHub Issues**: https://github.com/tankeito/clash-verge-auto-switch/issues
- **Repository**: https://github.com/tankeito/clash-verge-auto-switch
- **Email**: tqd354@gmail.com

---

**Clash Verge Auto Switch** | Automated Proxy Optimization for macOS

**Developed by tankeito** | 2026
