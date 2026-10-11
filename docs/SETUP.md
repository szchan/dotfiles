# Dotfiles 初始化与第一批配置接入

核查日期：2026-10-11（Asia/Singapore）。

本仓库使用 chezmoi 管理多台机器的个人配置，并使用 Git 保存配置历史。仓库放在 `~/dotfiles/`，Markdown 文档放在 `docs/`。本机的安装、故障排查和修复过程仍由独立的 `System-Maintain` 仓库记录。

## 当前完成情况

- 本机 chezmoi 为 `v2.72.1`，命令入口为 `/usr/bin/chezmoi`。
- 已在 `/home/szchan/dotfiles` 建立独立 Git 仓库。
- `.chezmoiroot` 指定 `home/` 为配置源文件的根目录。
- 初始化模板已生成本机 `~/.config/chezmoi/chezmoi.toml`。
- 已纳入 Ghostty 和 herdr 的配置文件，未纳入日志、会话和旧备份。
- 核查时源文件与实际配置内容完全相同，`chezmoi diff` 无输出。
- 核查时尚无 Git 提交；首次提交及下文的编辑练习留待用户执行。

目前只保存了 1Password 的连接模式设置，没有添加秘密取值模板，也没有验证 1Password CLI 授权。第一批的 Git、Bash、mise、Neovim 和字体配置尚未纳入。

## 目录和文件的作用

```text
~/dotfiles/
├── .git/
├── .chezmoiroot
├── docs/
│   └── SETUP.md
└── home/
    ├── .chezmoi.toml.tmpl
    └── dot_config/
        ├── ghostty/
        │   └── config
        └── herdr/
            └── config.toml
```

| 位置 | 作用 |
| --- | --- |
| `~/dotfiles/` | Git 仓库，保存配置源文件和文档 |
| `~/dotfiles/home/` | chezmoi 读取的配置源文件根目录 |
| `~/.config/ghostty/config` 等 | 应用实际读取的配置文件 |
| `~/.config/chezmoi/chezmoi.toml` | 本机 chezmoi 的运行配置，由初始化模板生成 |

`.chezmoiroot` 内容只有一行：

```text
home
```

因此，根目录的 `docs/` 和以后增加的 `README.md`、`AGENTS.md` 不属于待部署的配置。chezmoi 的初始化模板、忽略规则等源根目录特殊文件应放在 `home/` 中。

`dot_config` 映射到实际位置的 `.config`。chezmoi 还可能使用 `private_`、`executable_` 等文件名前缀保存属性，应保留工具生成的命名。

## 本次初始化步骤

先确认安装：

```bash
chezmoi --version
```

如果在 Omarchy 上尚未安装，可使用：

```bash
omarchy pkg add chezmoi
```

创建仓库布局：

```bash
mkdir -p ~/dotfiles/home ~/dotfiles/docs
cd ~/dotfiles
nvim .chezmoiroot
```

将上文的 `home` 写入 `.chezmoiroot`。然后创建 `home/.chezmoi.toml.tmpl`，内容如下：

```toml
sourceDir = "{{ .chezmoi.homeDir }}/dotfiles"

[onepassword]
mode = "account"
prompt = false
```

`{{ .chezmoi.homeDir }}` 在初始化时替换为当前机器的家目录，避免将本机用户名写死在共享模板中。`mode = "account"` 使用个人账号模式；`prompt = false` 关闭 chezmoi 自己的自动登录处理，1Password CLI 的桌面应用授权由 `op` 管理。

初始化本地仓库并生成运行配置：

```bash
chezmoi init --source "$HOME/dotfiles"
cat ~/.config/chezmoi/chezmoi.toml
```

本机生成的 `sourceDir` 为 `/home/szchan/dotfiles`。此命令没有使用 `--apply`，不会执行配置部署。需要再次从模板生成运行配置时，可运行 `chezmoi init`；它会重新生成配置文件，手工维护的设置应同步写入模板。

纳入两个现有配置文件：

```bash
chezmoi add ~/.config/ghostty/config
chezmoi add ~/.config/herdr/config.toml
```

这里显式选择单个文件，避免将应用日志、会话、快照和旧备份一起纳入。

检查源文件路径和差异：

```bash
chezmoi source-path ~/.config/ghostty/config
chezmoi source-path ~/.config/herdr/config.toml
chezmoi managed
chezmoi diff
git -C ~/dotfiles status --short
```

刚导入时源文件与实际文件相同，`chezmoi diff` 无输出是正常现象。Git 的未跟踪文件提示意味着文件尚未提交，不代表 chezmoi 没有管理它们。

## 接下来：提交第一份基线

查看并提交源文件与本文档：

```bash
cd ~/dotfiles
git add .chezmoiroot home/ docs/
git diff --cached --stat
git diff --cached
git commit -m "Initialize chezmoi with Ghostty and herdr configs"
```

此处只是本地提交。远程仓库和首次推送将在后续配置；在完成多机适配前，当前 Ghostty 配置仍包含 Omarchy 主题引用和 Linux 专用设置。

## 接下来：用 Ghostty 练习编辑与应用

编辑源配置：

```bash
EDITOR=nvim chezmoi edit ~/.config/ghostty/config
```

参数虽然是应用实际文件的路径，`chezmoi edit` 编辑的是源配置。在文件顶部添加一行注释即可：

```text
# Managed with chezmoi.
```

保存退出后，查看准备写入实际文件的差异：

```bash
chezmoi diff ~/.config/ghostty/config
```

预期只有新增注释。确认后应用到该文件，再次检查：

```bash
chezmoi apply ~/.config/ghostty/config
chezmoi diff ~/.config/ghostty/config
```

第二次 diff 应无输出，表示源配置与实际文件一致。本练习仅增加注释，无须重启 Ghostty。

最后提交改动：

```bash
git -C ~/dotfiles add home/
git -C ~/dotfiles diff --cached
git -C ~/dotfiles commit -m "Mark Ghostty config as managed by chezmoi"
```

## 日常操作方向

| 命令 | 数据方向或作用 |
| --- | --- |
| `chezmoi add 路径` | 将实际文件收进源配置 |
| `chezmoi edit 路径` | 编辑源配置 |
| `chezmoi diff 路径` | 查看源配置应用后将产生的差异 |
| `chezmoi apply 路径` | 将源配置写回实际位置 |
| `git commit` | 在源仓库中保存一个版本 |
| `git push` | 将 Git 提交推送到远程仓库 |

默认使用普通文件，应用实际配置与仓库中的源文件是两份文件，不是 Stow 式软链接。常用流程为：

```text
chezmoi edit → chezmoi diff → chezmoi apply → git commit → git push
```

如果直接修改了应用实际配置，当前这类普通源文件可以再用 `chezmoi add 路径` 导回仓库。以后文件转为模板时，应保留模板逻辑，按需使用 `chezmoi merge` 合并，避免强制导入覆盖模板。

## 后续接入顺序

1. Git：调整写死的 mise `gh` 安装路径，再纳入个人设置。
2. Bash：区分通用设置、Omarchy 初始化和本机代理设置。
3. mise：保存工具声明，由 mise 恢复软件安装。
4. Neovim：保存配置与插件锁文件，插件安装目录由插件管理器重建。
5. 字体：保存字体替换规则，并记录所需字体包。

## 官方参考

- [初始化命令](https://www.chezmoi.io/reference/commands/init/)
- [自定义源文件根目录](https://www.chezmoi.io/user-guide/advanced/customize-your-source-directory/)
- [导入配置文件](https://www.chezmoi.io/reference/commands/add/)
- [编辑源配置](https://www.chezmoi.io/reference/commands/edit/)
- [日常操作](https://www.chezmoi.io/user-guide/daily-operations/)
- [1Password 集成](https://www.chezmoi.io/user-guide/password-managers/1password/)
