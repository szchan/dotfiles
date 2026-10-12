# Dotfiles 仓库维护指引

本仓库使用 chezmoi 管理个人配置，以 Git 保存历史，默认位置为 `~/dotfiles`。目标是支持多台机器；目前已验证的基线为一台 Omarchy/Linux 机器。维护时先核对实际状态，不把已有文档中的日期快照当成永久事实。

## 文档入口

- [README.md（English）](README.md)：英文概览、仓库结构与常用操作。
- [README_CN.md（中文）](README_CN.md)：与英文 README 对应的中文概览。
- [LICENSE](LICENSE)：原创内容的 MIT 许可证；第三方文件继续保留各自的许可与版权声明。
- [docs/SETUP.md](docs/SETUP.md)：初始化步骤、配置映射、字体依赖、日常维护、恢复流程、提交与验证记录及后续事项。
- [docs/SSH.md](docs/SSH.md)：第二批 SSH 与 1Password 模板、权限、恢复依赖、授权复用及验证记录。
- [docs/INPUT-METHOD.md](docs/INPUT-METHOD.md)：第二批 Fcitx5 / Rime 的五个配置文件、恢复依赖、导入范围及验证记录。
- [docs/STARSHIP.md](docs/STARSHIP.md)：提示符普通配置、Bash 接入、恢复依赖与验证记录。
- [docs/BTOP.md](docs/BTOP.md)：btop 普通配置、Omarchy 主题链接模板、恢复依赖与验证记录。
- [docs/OMARCHY-SHELL.md](docs/OMARCHY-SHELL.md)：界面普通配置、八个第三方插件依赖、恢复次序与验证记录。
- [docs/DEFAULT-APPS.md](docs/DEFAULT-APPS.md)：默认终端、浏览器与协议关联、Clash 入口、邮件排除规则及验证记录。

开始维护前阅读本文和与任务相关的 SETUP 段落。新增配置、调整维护流程或完成待办后，更新相关文档；应用配置的详细说明优先放在 `docs/`。

## 工作范围与用户协作

- 这是独立的 dotfiles 仓库。系统安装、故障排查和机器维护记录由另一个 `System-Maintain` 仓库保存；不要把两个仓库混为一体。
- 用户正在通过亲自执行步骤学习 chezmoi。教学任务应给出可执行步骤，由用户操作；用户说“做完了，看看”时，以只读核查为主。
- 用户明确委托写文档或执行维护时，完成已授权工作。是否应用配置、安装软件、提交或推送，应依据当前任务的授权范围，不因本文件默认扩大权限。
- 修改前检查 Git 工作区与 chezmoi 差异，保留用户尚未提交或尚未同步的变化。完成后说明改动、验证结果和仍需用户操作的事项。

## 仓库布局与源文件规则

- [.chezmoiroot](.chezmoiroot) 的内容是 `home`，只有 `home/` 是 dotfiles 部署源根目录。
- 根目录的 README、本文和 `docs/` 保留在仓库中，不部署到用户家目录。
- [home/.chezmoi.toml.tmpl](home/.chezmoi.toml.tmpl) 生成本机 `~/.config/chezmoi/chezmoi.toml`。本机运行配置不要作为普通 dotfile 导入。
- 保留 chezmoi 的属性命名：`dot_`、`private_`、`executable_`、`symlink_` 和 `.tmpl` 等，不把它们当作多余前后缀删除。
- 普通源文件和应用实际文件是两份文件。编辑源配置使用 `chezmoi edit` 或明确的源路径，部署通过 `chezmoi apply`。
- 导入时选择明确的文件或已检查的配置目录。不要整体导入 `~/.config`、认证目录、缓存、会话、日志、历史数据库、安装目录或旧备份；不要未经评估使用 `--exact`。
- 相同环境是否存在某工具，适合由 shell 的可读性或可执行性检查处理；每台机器的偏好适合使用模板数据。共享配置避免写死用户名和工具安装版本路径。

## 第一批配置的维护约定

各项的完整配置与依赖说明见 [SETUP：七类配置的内容与维护要点](docs/SETUP.md#4-七类配置的内容与维护要点)。

| 配置 | 必须保留或复核的行为 |
| --- | --- |
| Ghostty | 保留 Omarchy 主题引用与已有快捷键；`io_uring` 是本机 Linux 崩溃规避设置，跨系统应用前复核 |
| herdr | 未设置选项沿用官方默认值，只保留少量 UI 偏好；不要恢复 Omarchy 旧快捷键覆盖 |
| Git | GitHub/Gist 的空 `helper =` 用于重置助手列表，后接 `!gh auth git-credential`；运行 Git 的环境需能找到 `gh` |
| Bash | `.bashrc` 是模板，保留代理条件、可选依赖检查与既有兼容补丁；`.profile` 保持 POSIX 语法 |
| mise | 配置保存工具声明，工具安装由 mise 完成；`latest` 不等于固定版本，安装与升级属于另外的操作 |
| Neovim | 保存配置和 `lazy-lock.json`，保留 Omarchy 主题符号链接；插件目录与缓存由插件管理器重建 |
| Fontconfig | 仅保存选定的规则文件，排除旧 `.bak`；字体文件及包安装另外处理 |

Neovim 的 [symlink_theme.lua](home/dot_config/nvim/lua/plugins/symlink_theme.lua) 是链接声明，指向 Omarchy 当前主题。不要使用 `chezmoi add --follow` 将其变成固定主题内容。没有 Omarchy 的机器需要先增加备用主题；当前仓库尚未完成跨平台验证。

修改 Omarchy 用户配置时遵循环境中适用的 Omarchy 指引；`/usr/share/omarchy/` 只读，修改用户配置或本仓库中的源配置。需要操作 herdr 本身时，遵循其官方 skill 和环境检查要求，不把仅查看本仓库配置变成会话操作。

## 输入法配置的维护约定

仅管理 [INPUT-METHOD.md](docs/INPUT-METHOD.md) 列出的四个 Fcitx5 文件及 Rime 的 `default.custom.yaml`。这些是普通文件，可在 GUI 更改后针对实际发生变化的文件重新 `chezmoi add`。保留 `private_` 属性和 Rime 的补丁结构。

不要整体导入 Rime 用户目录：学习词库、`sync/`、编译产物、安装标识和运行状态不属于公开配置基线。个人词库需要另外制定私密备份方案。雾凇拼音由系统包提供，不复制上游词典和编译文件到仓库；桌面启动和输入模块集成也尚未随本项纳入。导入现有配置且 diff 为空时，不为验证重启或重新部署输入法。

## Starship 维护约定

[home/dot_config/starship.toml](home/dot_config/starship.toml) 是普通 TOML 文件，当前不需要机器模板。维护方式见 [STARSHIP.md](docs/STARSHIP.md)。本机 Omarchy Bash 初始化脚本已经调用 `starship init bash`，不要重复添加初始化语句。非 Omarchy 的新机器需要按目标 Shell 另外接入；只恢复 TOML 不代表提示符已启用。

## btop 与当前显示器范围

btop 维护见 [BTOP.md](docs/BTOP.md)：`btop.conf` 是普通文件，主题链接是 `symlink_current.theme.tmpl`，使用 `.chezmoi.homeDir` 保持家目录可移植。保留链接，不使用 `--follow` 将主题复制为固定文件。btop 退出可能保存配置，偏好变更后应核对并针对普通配置重新导入。

用户于 2026-10-12 选择显示器配置暂时仅保留在本机，不纳入公开备份；尚未添加机器标识选项。后续不要因多机目标而自动导入 `monitors.lua`、`hyprmoncfg-monitors.lua` 或增加机器标识。实际需要时依据新的任务范围处理。

## Omarchy 界面配置

[home/dot_config/omarchy/private_shell.json](home/dot_config/omarchy/private_shell.json) 是普通 JSON 文件，保留 `private_` 属性。维护范围和插件依赖见 [OMARCHY-SHELL.md](docs/OMARCHY-SHELL.md)。界面工具改动实际文件后可针对该文件重新导入；不要因此整体导入插件目录、认证或运行状态。

当前配置停用了 `omarchy.menu` 和 `omarchy.lock`，引用八个第三方插件。在新机器应用之前按文档准备依赖并确认菜单和锁屏替代方案。配置备份不包含插件安装、账号认证或全部插件偏好。实际修改界面配置时先遵循 Omarchy skill；导入现有文件且 diff 为空时不为验证重启 shell。

## 默认应用维护约定

[DEFAULT-APPS.md](docs/DEFAULT-APPS.md) 列出的三个文件均为普通配置，可针对实际更改重新导入。Clash 两个协议指向 `clash-verge-handler.desktop`，不要恢复为本机不存在的 `clash-verge.desktop`。用户选择移除过期 Omamail 关联、暂不指定邮件客户端；保留 `Removed Associations` 中的排除规则，不因系统查询返回 `HEY.desktop` 就把它写为显式默认值。

恢复前核对应用是否安装及 desktop ID 是否有效；只读验证使用 `xdg-mime query`、`xdg-terminal-exec --print-id` 和 `desktop-file-validate`，不要为查看默认设置而打开浏览器、邮件或订阅链接。此项不包含应用账号、邮件数据或 Clash 订阅配置。

## 模板、本机选项与秘密

- `.bashrc` 从本机 `[data].enableProxy` 读取代理开关；初始化模板使用 `promptBoolOnce`，默认关闭，已有选择会复用。
- 修改初始化模板后需要重新 `chezmoi init --source "$HOME/dotfiles"` 才会生成新的本机运行配置。不要为重新生成配置顺手添加 `--apply`；先查看部署差异。
- 关闭代理选项不等于清理已运行进程的环境变量，具体处理见 [SETUP：更改本机代理选择](docs/SETUP.md#更改本机代理选择)。
- 模板被应用后成为普通文本。不要用实际 `.bashrc` 强制覆盖源模板，导致条件逻辑丢失。
- 1Password 使用 `account` 模式及 `prompt = false`；SSH 模板已通过 `onepasswordRead` 读取服务器信息与公钥。源文件保存引用，不提交实际服务器信息、密码、token、私钥或认证状态文件。
- SSH 维护见 [SSH.md](docs/SSH.md)。保留 `private_dot_ssh` 及文件的 `private_` 属性；`IdentityFile` 指向公钥，私钥由 1Password agent 使用。不要用 `chezmoi add ~/.ssh/config` 将渲染结果覆盖源模板，也不要整体导入 `~/.ssh` 或 1Password 自动生成的文件。当前 agent socket 路径按 Linux 配置，跨系统恢复需要先适配。
- 读取或验证秘密时避免把值输出到日志、文档或工具结果中。运行时如何向应用提供秘密按用户授权与应用需求决定，不把“不允许任何运行时明文”作为本仓库默认要求。

## 修改与验证流程

先检查实际状态：

```bash
git -C ~/dotfiles status --short
chezmoi diff
```

配置变更通常按以下方向处理：

```text
编辑源配置 → 查看目标文件的 diff → 按授权应用目标文件 → 再检查 diff → 查看 Git 差异 → 按授权提交
```

优先只应用本次涉及的目标，避免将其他尚未审阅的源变化一起部署。普通实际文件被应用修改后，可以针对该文件 `chezmoi add` 导回仓库，例如 mise 配置和 Neovim 锁文件；模板需保留源逻辑。具体命令见 [SETUP：日常维护流程](docs/SETUP.md#5-日常维护流程)。

验证应与变化相称：

- 文档：检查相对链接、Markdown 格式和命令示例；不需要应用 dotfiles 或启动应用。
- Bash 模板：先渲染，再检查 Bash 语法；条件改变时检查相关分支。
- `.profile`：检查 POSIX 写法。本机 `sh` 指向 Bash，`sh -n` 不能独立证明其他 shell 的兼容性。
- TOML、JSON、XML：检查格式；部署后比较源渲染结果与实际文件。
- 符号链接：核对链接目标与可解析性；格式或文件同步成功不等于跨机器功能验证。
- 涉及行为变化时再做相关应用验证；不要为只读检查或文档修改触发安装、升级、授权或会话操作。

常用仓库检查：

```bash
git -C ~/dotfiles diff --check
git -C ~/dotfiles diff
chezmoi diff
```

输出可能含个人信息或未来的秘密模板结果，应按任务需要控制检查范围；不要为查看 Git 凭据配置运行会输出 token 的命令。

## 文档与后续维护

- 两份 README 保持内容对应：英文版首行是 `English | [简体中文](README_CN.md)`，中文版首行是 `[English](README.md) | 简体中文`。当前语言为普通文字，只有另一种语言是链接。
- 更新配置范围、初始化方式或适用平台时，同时维护两份 README 和 SETUP 中相关内容。日期、验证状态和提交记录只写实际核实的结果。
- 远程备份、SSH、1Password 秘密模板及多机部署的进度见 [SETUP：后续事项](docs/SETUP.md#8-后续事项)。着手下一阶段前核对 Git remote、配置源和实际部署情况，再更新文档。
- 恢复其他机器前阅读 [SETUP：新机器恢复流程与当前适用范围](docs/SETUP.md#6-新机器恢复流程与当前适用范围)。配置仓库不包含全套系统或依赖安装，不应直接宣称恢复后与本机完全等价。
