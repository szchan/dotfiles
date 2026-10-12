# Starship 提示符配置

核查日期：2026-10-12（Asia/Singapore）。当前验证范围为本机 Omarchy/Linux。概览见 [SETUP.md](SETUP.md)。

## 管理的文件与偏好

| 源文件 | 实际文件 | 类型与权限 |
| --- | --- | --- |
| [starship.toml](../home/dot_config/starship.toml) | `~/.config/starship.toml` | 普通 TOML 文件，`644` |

当前配置没有机器选项，未使用 `.tmpl`。主要保存：

- 提示符包含目录、Git 分支、Git 状态和末尾字符。
- `add_newline = true`，`command_timeout = 200` 毫秒。
- 目录截断长度为 `2`，截断符号为 `…/`；仓库根目录使用 cyan 配色。
- 成功提示符为 `❯`，失败为 `✗`，均为 bold cyan。
- Git 分支为 italic cyan，Git 状态为 cyan，并保留已有的领先、落后、冲突、未跟踪和修改符号。
- `stashed`、`staged`、`renamed`、`deleted` 的状态符号设为空字符串。

这里保存显示规则，Git 状态的实际内容由 Starship 在运行时读取。[Starship 配置说明](https://starship.rs/config/)

## 恢复依赖与 Shell 接入

本机已安装 `starship 1.26.0-1`；版本为本次核查快照。配置中的部分 Git 图标需要字体支持，本机已有第一批记录的 JetBrainsMono Nerd Font。

当前 Bash 的初始化链为：

```text
~/.bashrc
  → /usr/share/omarchy/default/bash/rc
    → /usr/share/omarchy/default/bash/init
      → starship init bash
```

已只读核对：Omarchy 在交互 Shell、`TERM` 不为 `dumb` 且能找到 `starship` 时执行初始化。现有 `.bashrc` 已加载 Omarchy rc，不需要重复添加初始化。

在非 Omarchy 机器上，只恢复 TOML 不会自动启用 Starship。需要安装 Starship，并在对应 Shell 的启动配置中接入；Bash 使用 `eval "$(starship init bash)"`。应先检查现有初始化，按机器情况决定是否添加，避免重复执行。其他 Shell 需要使用自身的接入方式。

`/usr/share/omarchy/` 是系统包管理的只读依赖，不复制或修改其中的初始化脚本。

## 导入与日常维护

本次由用户执行：

```bash
chezmoi add ~/.config/starship.toml
chezmoi diff ~/.config/starship.toml
git -C ~/dotfiles status --short
```

只是保存现有配置，diff 为空时无需 `apply`。

后续修改源配置：

```bash
EDITOR=nvim chezmoi edit ~/.config/starship.toml
chezmoi diff ~/.config/starship.toml
chezmoi apply ~/.config/starship.toml
```

实际修改后在交互 Shell 中检查提示符；如果直接修改了实际 TOML，可以针对该普通文件重新 `chezmoi add`。缓存和日志不属于本项配置。

## 验证记录

只读检查确认：

- 本轮只新增 `home/dot_config/starship.toml`，为普通文件。
- 源文件与实际文件逐字节一致，双方权限为 `644`。
- Python `tomllib` 解析成功，目标 `chezmoi diff` 无输出。
- 已核对本机 Omarchy Bash 初始化链及 Starship 软件包版本。

本轮没有修改显示行为、应用配置或启动新的交互 Shell；未进行新机器的恢复测试。本次配置及文档待用户提交、推送。
