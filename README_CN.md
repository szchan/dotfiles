[English](README.md) | 简体中文

# 个人 Dotfiles

使用 [chezmoi](https://www.chezmoi.io/) 管理个人配置，用 Git 保存版本历史。仓库放在 `~/dotfiles`，与本机的系统维护工作目录分开。

Agent 维护指引见 [AGENTS.md](AGENTS.md)。

当前基线已在一台 Omarchy/Linux 机器上验证。仓库计划支持多台机器，其他平台和不同机器的配置适配仍在进行中。

## 已管理的配置

| 组件 | 纳入的内容 |
| --- | --- |
| Ghostty | 字体、窗口偏好、快捷键、Omarchy 主题引用和 Linux 崩溃规避设置 |
| herdr | 少量 UI 偏好，未设置的选项沿用官方默认值 |
| Git | 身份、别名，以及通过 `PATH` 查找 `gh` 的 GitHub/Gist 凭据助手 |
| Bash | `.bashrc`、`.bash_profile`、`.profile`、可选依赖加载和每台机器的代理开关 |
| mise | 工具版本声明和升级偏好 |
| Neovim | Lua 配置、插件锁文件和 Omarchy 主题符号链接 |
| Fontconfig | 等宽字体偏好与简体中文字体回退规则 |

应用缓存、会话、下载的工具和字体文件不属于这份配置基线。字体目录中的旧备份文件也没有导入。

## 仓库结构

```text
~/dotfiles/
├── README.md                  # 英文概览
├── README_CN.md               # 中文概览
├── AGENTS.md                  # 给维护本仓库的 Agent 的指引
├── LICENSE                    # 原创内容的 MIT 许可证
├── .chezmoiroot               # 指定 home/ 为配置源根目录
├── docs/
│   └── SETUP.md               # 初始化与日常维护的详细记录（中文）
└── home/
    ├── .chezmoi.toml.tmpl     # 生成本机 chezmoi 配置
    ├── dot_bashrc.tmpl
    ├── dot_bash_profile
    ├── dot_profile
    └── dot_config/            # 应用配置
```

只有 `home/` 是部署源根目录。README 和 `docs/` 保留在仓库内。

chezmoi 用文件名编码目标属性：`dot_` 映射为开头的点，`.tmpl` 表示模板，`symlink_` 声明符号链接。普通配置的源文件与部署后的实际文件是两份文件。

## 初始化已有的仓库副本

在兼容的机器上安装 Git 和 chezmoi 后，如果 `~/dotfiles` 尚不存在，可以先克隆仓库：

```bash
git clone https://github.com/szchan/dotfiles.git "$HOME/dotfiles"
```

仓库已经放在 `~/dotfiles` 时：

```bash
chezmoi init --source "$HOME/dotfiles"
chezmoi diff
```

初始化会生成 `~/.config/chezmoi/chezmoi.toml`，并询问这台机器是否使用 `127.0.0.1:7897` 的本地代理。默认关闭；之后重新初始化时会复用已保存的选择。

检查准备写入的变化后再应用：

```bash
chezmoi apply
chezmoi diff
```

应用及其依赖需要另外安装。在用户主目录运行 `mise install` 安装声明的工具；在 Neovim 中执行 `:Lazy restore`，可恢复 `lazy-lock.json` 中记录的插件版本。所需字体族及本机对应的 Arch 包名见详细配置记录。

没有 Omarchy 的机器应先适配主题，再应用完整基线。Neovim 主题链接依赖 Omarchy 当前主题目录；Ghostty 含有 `async-backend = io_uring`。Git 身份目前是共享配置；mise 中多数工具使用 `latest`，Node 固定为 `26.8.2`。

## 日常操作

编辑源配置、查看差异，然后应用：

```bash
EDITOR=nvim chezmoi edit ~/.config/ghostty/config
chezmoi diff ~/.config/ghostty/config
chezmoi apply ~/.config/ghostty/config
```

查看并提交源文件的变化：

```bash
git -C ~/dotfiles add home/
git -C ~/dotfiles diff --cached
git -C ~/dotfiles commit -m "Describe the configuration change"
```

如果应用直接修改了普通配置，可以用 `chezmoi add` 重新导入该文件，例如 mise 工具清单或 Neovim 插件锁文件。对于 `.bashrc`，应编辑源模板并保留条件逻辑。

需要重新选择代理开关时：

```bash
chezmoi init --source "$HOME/dotfiles" --prompt
chezmoi diff ~/.bashrc
chezmoi apply ~/.bashrc
```

关闭开关会移除生成文件中的代理导出语句，但不会清理已运行 shell 的环境变量。清理命令见详细配置记录。

## 详细文档与后续事项

[初始化与日常维护记录](docs/SETUP.md) 包含初始化步骤、配置映射、字体依赖、提交历史、验证结果和恢复流程。

截至 2026-10-11，第一批配置已完成，并已备份到公开的 GitHub 仓库 [szchan/dotfiles](https://github.com/szchan/dotfiles)。SSH 配置、1Password 秘密模板及第二台机器的部署验证仍待完成。初始化模板已声明 1Password `account` 模式及 `prompt = false`，当前 dotfiles 尚未从 1Password 读取秘密；GitHub 认证由 `gh` 处理。

## 许可证

本仓库的原创配置改动与文档采用 [MIT 许可证](LICENSE)，版权所有者为 Shaoze Chen，年份为 2026。

仓库中的第三方文件继续保留原有许可证。Neovim starter 自带的 [Apache-2.0 许可证](home/dot_config/nvim/LICENSE)已保留。
