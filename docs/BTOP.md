# btop 配置与主题链接模板

核查日期：2026-10-12（Asia/Singapore）。当前验证范围为本机 Omarchy/Linux。概览见 [SETUP.md](SETUP.md)。

## 管理的内容

| 源文件 | 实际文件 | 类型 |
| --- | --- | --- |
| [btop.conf](../home/dot_config/btop/btop.conf) | `~/.config/btop/btop.conf` | 普通配置文件，`644` |
| [symlink_current.theme.tmpl](../home/dot_config/btop/themes/symlink_current.theme.tmpl) | `~/.config/btop/themes/current.theme` | 符号链接模板 |

配置保存面板布局、显示选项、Vim 按键和刷新偏好。当前主要设置包括：

- `color_theme = "current"`，跟随当前 Omarchy 主题。
- `vim_keys = true`，`shown_boxes = "cpu mem net proc"`，`update_ms = 2000`。
- 进程排序为 `cpu lazy`，图形符号使用 `braille`。
- 温度传感器和电池自动选择，网络接口为空；本轮没有指定固定硬件名称。
- `save_config_on_exit = true`，应用退出时可能保存配置。

没有导入运行日志或复制 Omarchy 的主题内容。[btop 官方项目](https://github.com/aristocratos/btop)

## 符号链接模板

现有主题链接使用绝对家目录路径。通过 `--template-symlinks` 导入后，源文件内容为：

```gotemplate
{{ .chezmoi.homeDir }}/.local/state/omarchy/current/theme/btop.theme
```

chezmoi 在部署时渲染目标路径，再创建符号链接。它保存的是链接声明，不是主题文件的副本；换一个用户名时会使用该用户的家目录。[chezmoi add 说明](https://www.chezmoi.io/reference/commands/add/#--template-symlinks)

不要使用 `--follow`，否则会将链接指向的主题导入为固定文件。

## 导入与状态查看

本次由用户执行：

```bash
chezmoi add ~/.config/btop/btop.conf
chezmoi add --template-symlinks ~/.config/btop/themes/current.theme
chezmoi diff ~/.config/btop/btop.conf ~/.config/btop/themes/current.theme
```

目标 diff 无输出表示普通文件内容和渲染后的链接目标都一致，本轮无需 `apply`。

`git status --short` 默认可能把未跟踪目录合并显示为 `?? home/dot_config/btop/`。这是显示方式，不代表导入范围不同。查看每个未跟踪文件时使用：

```bash
git -C ~/dotfiles status --short --untracked-files=all
```

## 日常维护与恢复依赖

源配置可以通过以下流程修改：

```bash
EDITOR=nvim chezmoi edit ~/.config/btop/btop.conf
chezmoi diff ~/.config/btop/btop.conf
chezmoi apply ~/.config/btop/btop.conf
```

在 btop 内调整显示偏好后，正常退出应用，再核对实际文件变化；需要保存时重新 `chezmoi add ~/.config/btop/btop.conf`。主题链接模板继续保留其家目录变量。

本机安装的软件包为 `btop 1.4.7-1`；版本是核查快照。新机器需要另外安装 btop，并提供 Omarchy 当前主题路径。没有 Omarchy 的机器需要先适配主题选择和链接管理，不能直接认为模板化家目录已经解决所有平台差异。

## 验证记录

只读检查确认：

- 本轮只新增普通配置和符号链接模板，共两个源文件。
- `btop.conf` 与实际文件逐字节一致，实际权限为 `644`。
- 链接仍是符号链接，目标主题文件存在。
- 源模板渲染出的目标与当前链接目标一致，两个目标的 `chezmoi diff` 无输出。

本轮没有修改显示偏好、运行 btop、切换主题或应用配置；其他机器恢复测试尚未进行。本次新增配置和文档待用户提交、推送。
