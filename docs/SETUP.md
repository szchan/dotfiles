# Dotfiles 初始化、配置与日常维护

首次核查：2026-10-11；最近维护：2026-10-12（Asia/Singapore）。本机 chezmoi：`v2.72.1`，入口 `/usr/bin/chezmoi`。

本仓库使用 chezmoi 管理个人配置，用 Git 保存历史，放在 `~/dotfiles/`。文档放在 `docs/`；本机安装、故障排查和修复记录继续保存在独立的 `System-Maintain` 仓库。

## 1. 当前完成情况

第一批七类配置已全部导入并提交：Ghostty、herdr、Git、Bash、mise、Neovim、字体配置。

第一批完成时的核查结果：

- 全局 `chezmoi diff` 无输出，已管理的配置与本机实际文件一致。
- 文档更新前 Git 工作区干净，配置基线最后一笔提交为 `785e068`。
- 本机代理选项为 `enableProxy = true`；临时以 `false` 渲染时不会生成代理导出语句。
- Bash 启动文件语法检查、mise TOML、Neovim JSON、Fontconfig XML 检查通过。
- Neovim 主题链接保留了原始目标路径，当前能正常解析。
- 字体目录的旧 `.bak` 文件未导入仓库。

远程 Git 备份已完成：公开仓库为 [szchan/dotfiles](https://github.com/szchan/dotfiles)，`origin` 为 `https://github.com/szchan/dotfiles.git`，本地 `main` 跟踪 `origin/main`。

第二批已完成本机 SSH 与 1Password 模板接入：源文件只保存引用，渲染结果与实际配置一致；用户确认两台服务器真实登录成功。配置与验证记录见 [SSH.md](SSH.md)，已提交并推送为 `89603c1`。

第二批输入法配置已导入：Fcitx5 的四个文件和 Rime 的 `default.custom.yaml`，源文件与本机逐字节一致，目标 diff 为空。说明见 [INPUT-METHOD.md](INPUT-METHOD.md)，已提交并推送为 `d78a30e`。

Starship 提示符配置已导入，TOML 解析通过，源文件与本机一致、目标 diff 为空。见 [STARSHIP.md](STARSHIP.md)；本次 Starship 配置和文档尚待提交、推送。

1Password 使用 `account` 模式和 `prompt = false`，已验证模板读取服务器信息与公钥；SSH 私钥仍由 agent 使用。其他机器的实际部署验证尚未完成。Git 的 GitHub 凭据仍由 `gh` 获取。

## 2. 仓库布局与文件映射

```text
~/dotfiles/
├── .chezmoiroot                 # 内容为 home
├── docs/
│   ├── SETUP.md                 # 本文，不部署到家目录
│   ├── SSH.md                   # 第二批 SSH 配置与验证记录
│   ├── INPUT-METHOD.md          # 第二批输入法配置与恢复依赖
│   └── STARSHIP.md              # 提示符配置与 Shell 接入说明
└── home/
    ├── .chezmoi.toml.tmpl        # 初始化本机 chezmoi 配置
    ├── dot_bashrc.tmpl
    ├── dot_bash_profile
    ├── dot_profile
    ├── dot_local/share/fcitx5/rime/private_default.custom.yaml
    ├── private_dot_ssh/
    │   ├── private_config.tmpl
    │   └── private_id_ed25519_1password.pub.tmpl
    └── dot_config/
        ├── starship.toml
        ├── fcitx5/              # 四个选定的配置文件
        ├── ghostty/config
        ├── herdr/config.toml
        ├── git/config
        ├── mise/config.toml
        ├── nvim/                # 19 个普通文件及 1 个符号链接声明
        └── fontconfig/
            ├── fonts.conf
            └── conf.d/70-source-han-sc.conf
```

`.chezmoiroot` 将 `home/` 指定为配置源根目录，因此仓库根目录的 `docs/` 不属于部署内容。初始化模板、以后增加的忽略规则等特殊文件应放在 `home/`。[chezmoi 源目录说明](https://www.chezmoi.io/user-guide/advanced/customize-your-source-directory/)

| 仓库内的源路径 | 应用实际读取的位置 |
| --- | --- |
| `home/dot_bashrc.tmpl` | `~/.bashrc`，先渲染模板 |
| `home/dot_bash_profile` | `~/.bash_profile` |
| `home/dot_profile` | `~/.profile` |
| `home/dot_config/ghostty/config` | `~/.config/ghostty/config` |
| `home/dot_config/herdr/config.toml` | `~/.config/herdr/config.toml` |
| `home/dot_config/git/config` | `~/.config/git/config` |
| `home/dot_config/mise/config.toml` | `~/.config/mise/config.toml` |
| `home/dot_config/nvim/` | `~/.config/nvim/` |
| `home/dot_config/fontconfig/` | `~/.config/fontconfig/` 中选定的两个文件 |
| `home/private_dot_ssh/private_config.tmpl` | `~/.ssh/config`，从 1Password 渲染 |
| `home/private_dot_ssh/private_id_ed25519_1password.pub.tmpl` | `~/.ssh/id_ed25519_1password.pub`，从 1Password 渲染 |
| `home/dot_config/fcitx5/` | `~/.config/fcitx5/` 中选定的四个文件 |
| `home/dot_local/share/fcitx5/rime/private_default.custom.yaml` | `~/.local/share/fcitx5/rime/default.custom.yaml` |
| `home/dot_config/starship.toml` | `~/.config/starship.toml` |

`dot_` 表示目标名称以点开头，`.tmpl` 表示内容需要渲染，`symlink_` 表示创建符号链接。`private_` 限制目标的组与其他用户权限，但不加密源文件或 Git 内容。应保留 chezmoi 的属性命名。

本机运行配置为 `~/.config/chezmoi/chezmoi.toml`，由初始化模板生成，不作为普通配置导入。本仓库采用 chezmoi 默认的文件模式：普通配置的源文件和实际文件是两份文件；Neovim 的主题链接是明确保留的符号链接。

## 3. 初始化与导入记录

以下记录用于理解初始化过程；当前机器已完成，不需要重复导入全部配置。

在 Omarchy 上安装 chezmoi（尚未安装时）：

```bash
omarchy pkg add chezmoi
chezmoi --version
```

创建仓库目录，并将 `home` 写入 `.chezmoiroot`：

```bash
mkdir -p ~/dotfiles/home ~/dotfiles/docs
nvim ~/dotfiles/.chezmoiroot
```

当前完整初始化模板 [home/.chezmoi.toml.tmpl](../home/.chezmoi.toml.tmpl) 为：

```toml
{{- $enableProxy := promptBoolOnce . "enableProxy" "Enable local proxy at 127.0.0.1:7897" false -}}
sourceDir = "{{ .chezmoi.homeDir }}/dotfiles"

[onepassword]
mode = "account"
prompt = false

[data]
enableProxy = {{ $enableProxy }}
```

`sourceDir` 使用当前机器家目录，避免写死用户名。`promptBoolOnce` 复用已有布尔值；没有记录时询问，默认关闭代理。本机选择启用代理，生成的 `[data]` 中为 `enableProxy = true`。[promptBoolOnce 说明](https://www.chezmoi.io/reference/templates/init-functions/promptBoolOnce/)

`onepassword.prompt = false` 关闭 chezmoi 自己的登录提示逻辑；采用桌面集成时授权由 `op` 与 1Password 应用处理。此设置本身不获取任何秘密。[1Password 集成说明](https://www.chezmoi.io/user-guide/password-managers/1password/)

初始化独立 Git 仓库并生成本机运行配置：

```bash
chezmoi init --source "$HOME/dotfiles"
```

未加 `--apply`，因此此步骤不部署 dotfiles。修改初始化模板后，需要重新运行此命令才能重新生成本机配置。[init 命令说明](https://www.chezmoi.io/reference/commands/init/)

各批次的导入命令如下（之后的源配置修改见下一节）：

```bash
chezmoi add ~/.config/ghostty/config ~/.config/herdr/config.toml
chezmoi add ~/.config/git/config
chezmoi add --template ~/.bashrc
chezmoi add ~/.bash_profile ~/.profile
chezmoi add ~/.config/mise/config.toml
chezmoi add ~/.config/nvim
chezmoi add ~/.config/fontconfig/fonts.conf ~/.config/fontconfig/conf.d/70-source-han-sc.conf
```

Neovim 导入时保留链接，没有使用 `--follow`；字体只选择正在使用的两个文件。Bash 的 `.bashrc` 导入后加入了条件模板，今后应通过 `chezmoi edit` 维护这些逻辑。[add 命令说明](https://www.chezmoi.io/reference/commands/add/)

## 4. 七类配置的内容与维护要点

### Ghostty

保存字体、窗口、光标、键盘映射、滚动速度与 shell integration 设置。新增 `# Managed with chezmoi` 注释完成了第一次“编辑源文件 → 查看差异 → 应用 → 提交”练习。

主题继续引用 `~/.local/state/omarchy/current/theme/ghostty.conf`；该路径前的 `?` 保留在配置中。仓库没有复制 Omarchy 的动态主题目录。

`async-backend = io_uring` 是本机已有的崩溃规避设置，相关排查记录在 `System-Maintain/ghostty-maintenance/docs/ghostty-herdr-crash.md`。它属于 Linux 本机基线；其他系统部署前应复核。

### herdr

只保存 `~/.config/herdr/config.toml`。未设置的选项继续采用 herdr 官方内置默认值，保留已完成 onboarding 的状态以及少量 UI 偏好：pane 边框显示 Agent 标签、点状状态指示和终端 toast。

没有纳入会话、日志、历史数据库或旧配置备份，也没有恢复 Omarchy 的旧快捷键覆盖。本机安装与配置依据见 `System-Maintain/herdr-maintenance/docs/LOCAL-SETUP.md`。

### Git

保存个人身份、别名、diff 等已有偏好。姓名和邮箱目前仍是普通配置，尚未按机器参数化。

将 GitHub、Gist 凭据助手的固定 `gh` 安装版本路径改为通过 `PATH` 查找：

```gitconfig
[credential "https://github.com"]
    helper =
    helper = !gh auth git-credential

[credential "https://gist.github.com"]
    helper =
    helper = !gh auth git-credential
```

空的 `helper =` 清除继承的助手列表；`!` 表示后面是 shell 命令。两段配置不保存 token，实际认证由 `gh` 管理。运行 Git 的环境（包括 GUI 客户端）需要能在 `PATH` 中找到 `gh`。[Git 凭据助手说明](https://git-scm.com/docs/gitcredentials)

### Bash 与每台机器的代理选项

保存 `.bashrc`、`.bash_profile` 和 `.profile`。其中只有 `.bashrc` 是模板，现有代理导出语句被包在：

```gotemplate
{{ if .enableProxy }}
# Proxy
# 此处为现有的代理 export 语句
{{ end }}
```

HTTP/HTTPS/FTP 使用 `http://127.0.0.1:7897`，ALL/SOCKS 使用 `socks5://127.0.0.1:7897`。大小写形式均保留。当前地址不含凭据；启用该选项的机器需要在该地址运行代理服务。渲染后的 `.bashrc` 没有 `{{ ... }}` 指令。

Omarchy 的环境 bootstrap 和交互 rc、ble.sh、`~/.local/bin/env`、`~/.cargo/env` 均有可读性检查。`~/.local/share/../bin/env` 改写为等价且更清楚的 `~/.local/bin/env`。这些由其他工具生成的 env 文件没有随本轮导入。

`.bash_profile` 保留加载 `.bashrc` 的语句；两个登录文件仍保留各自原有的 env 加载顺序。本轮没有重新设计启动流程。`.bashrc` 也保留了 Ghostty/ble.sh 兼容补丁、vim 别名及 Omarchy Time Machine 别名；后一个别名依赖相应插件。

`.profile` 中的可选文件加载使用：

```sh
if [ -r "$HOME/.local/bin/env" ]; then
    . "$HOME/.local/bin/env"
fi
```

`[ -r 路径 ]` 判断文件是否可读；`.` 在当前 shell 中加载文件。`[` 和 `]` 与内部参数之间需要空格。`[ ... ]` 与 `.` 是 POSIX 写法；`[[ ... ]]` 和 `source` 属于部分 shell 的扩展，可以在 Bash 专用文件中使用。本机 `sh` 指向 Bash，所以 `sh -n` 的成功并不能单独证明兼容其他 shell；本次另行检查了 `.profile` 本身的 POSIX 语法。[POSIX test 说明](https://pubs.opengroup.org/onlinepubs/9699919799.2016edition/utilities/test.html)

### mise

保存 [mise 工具清单](../home/dot_config/mise/config.toml)：bun、claude、codex、gh、`github:can1357/oh-my-pi`、go、node、pi、python，以及 `settings.upgrade.auto_prune = false`。

Node 固定为 `26.8.2`，其余当前均为 `latest`。这记录了版本选择策略，尚未固定所有工具的具体版本。chezmoi 恢复配置后，工具安装由 mise 完成；在家目录运行 `mise install`，避免意外使用维护项目自身的 mise 配置。工具安装目录、二进制和下载缓存没有纳入仓库。[mise install 说明](https://mise.jdx.dev/cli/install.html)

### Neovim

保存完整配置目录，包括初始化 Lua、个人选项、插件声明、远程剪贴板、透明背景、主题热加载设置、`lazyvim.json` 与 `lazy-lock.json`。

`lazy-lock.json` 保存插件修订版本，插件更新后应检查并提交它。其他机器可在 Neovim 中执行 `:Lazy restore` 恢复锁文件中的版本；插件安装与缓存由 lazy.nvim 重建。[lazy.nvim 锁文件说明](https://lazy.folke.io/usage/lockfile)

主题文件保留为符号链接，其源声明为：

```text
home/dot_config/nvim/lua/plugins/symlink_theme.lua
```

内容为相对目标路径：

```text
../../../../.local/state/omarchy/current/theme/neovim.lua
```

chezmoi 部署后生成 `~/.config/nvim/lua/plugins/theme.lua` 链接，继续跟随 Omarchy 当前主题。没有将链接解引用为一份固定主题文件，也没有复制主题状态目录。相对路径避免写死用户名，但目标仍依赖 Omarchy；没有 Omarchy 的机器需要先增加备用主题方案。

远程剪贴板配置包含 Linux `/proc` 祖先进程检查，以及 Wayland 和 OSC 52 的处理；跨系统部署还需实际验证。本轮检查了文件同步与 JSON 格式，没有声称已在其他机器测试整个 Neovim 环境。

### 字体配置

仅保存 `fonts.conf` 和 `conf.d/70-source-han-sc.conf`：

- 等宽字体优先使用 JetBrainsMono Nerd Font。
- 简体中文回退优先使用 Source Han SC，再使用 Noto CJK SC；规则包含日文、韩文、繁体中文语言排除条件，serif 与其他字体族分别处理。

`conf.d/70-source-han-cn.conf.bak.20261007-224304` 是旧备份，未导入。

配置规则不包含字体文件。本机通过 `fc-match` 和 `pacman -Qo` 核实的字体包如下；版本是本次核查快照，不是安装时必须固定的版本。

| 字体 | 本机提供该字体的包 | 核查版本 |
| --- | --- | --- |
| JetBrainsMono Nerd Font | `ttf-jetbrains-mono-nerd-basic` | `3.5.1-1` |
| Source Han Sans SC | `adobe-source-han-sans-otc-fonts` | `2.005-2` |
| Source Han Serif SC | `adobe-source-han-serif-otc-fonts` | `2.003-2` |
| Noto Sans/Serif CJK SC | `noto-fonts-cjk` | `20240730-1` |

其他系统可以使用提供同名字体族的对应软件包。恢复后可用 `fc-match 'JetBrainsMono Nerd Font'`、`fc-match 'Source Han Sans SC'` 等检查字体解析。

## 5. 日常维护流程

### 编辑由 chezmoi 管理的源配置

```bash
EDITOR=nvim chezmoi edit ~/.config/ghostty/config
chezmoi diff ~/.config/ghostty/config
chezmoi apply ~/.config/ghostty/config
chezmoi diff ~/.config/ghostty/config

git -C ~/dotfiles add home/
git -C ~/dotfiles diff --cached
git -C ~/dotfiles commit -m "Describe the configuration change"
```

`chezmoi edit` 接收实际文件路径，但编辑源文件；`diff` 显示准备应用的变化，`apply` 写入实际文件。最后一次 diff 无输出表示一致。[chezmoi 日常操作说明](https://www.chezmoi.io/user-guide/daily-operations/)

修改文档只需 Git 提交，不需要 `chezmoi apply`。提交本文更新时：

```bash
git -C ~/dotfiles add docs/SETUP.md
git -C ~/dotfiles diff --cached
git -C ~/dotfiles commit -m "Document first-batch dotfiles setup and maintenance"
```

### 软件直接修改了实际配置

普通源文件可以重新导入，例如 `mise use -g` 修改了工具清单，或 lazy.nvim 更新了插件锁文件后：

```bash
chezmoi add ~/.config/mise/config.toml
# 或针对本次实际发生的插件更新：
chezmoi add ~/.config/nvim/lazy-lock.json
git -C ~/dotfiles diff
```

然后只暂存并提交需要保存的变化。`.bashrc` 是模板：直接改了实际文件时，应将需要的变化编辑回模板并保留条件逻辑，不要强制用渲染后的文件覆盖模板。

### 更改本机代理选择

重新询问初始化选项，选择 `yes` 或 `no`：

```bash
chezmoi init --source "$HOME/dotfiles" --prompt
chezmoi execute-template '{{ .enableProxy }}'
chezmoi diff ~/.bashrc
chezmoi apply ~/.bashrc
```

这改变本机运行配置中的选择，不需要把本机生成的 `chezmoi.toml` 提交到共享仓库。初始化模板仍保留默认关闭代理。

关闭选项只会移除之后生成的 `export` 语句，不会清理已运行 shell 中的环境变量。需要立即清理时，在对应 shell 执行：

```bash
unset http_proxy HTTP_PROXY https_proxy HTTPS_PROXY all_proxy ALL_PROXY ftp_proxy FTP_PROXY socks_proxy SOCKS_PROXY
```

### 查看整体状态

```bash
chezmoi managed
chezmoi diff
git -C ~/dotfiles status --short
git -C ~/dotfiles log --oneline
```

`chezmoi diff` 看部署差异；`git status` 看源仓库尚未提交的变化，二者不是同一件事。当前已配置 GitHub `origin`；使用 `git -C ~/dotfiles push` 将新提交同步到远程，未提交的工作区变化不会被推送。

## 6. 新机器恢复流程与当前适用范围

当前验证范围是这台 Omarchy/Linux 机器。Bash 已增加可选组件检查和代理开关，但仓库还没有完成所有配置的跨平台适配；恢复配置也不等于安装整个桌面系统。

在相同环境恢复前，准备 Git、chezmoi、相关应用、mise 和字体。Git 的凭据助手依赖已安装且可用的 `gh`；Neovim 主题依赖 Omarchy 主题目录。完整恢复现在还需要 1Password 桌面应用与 `op` CLI，启用 CLI 集成和 SSH agent，并能访问引用的条目。SSH 模板使用本机 Linux socket 路径；其他系统需要先适配。详见 [SSH：恢复依赖](SSH.md#恢复依赖)。输入法还需要 Fcitx5、Rime、雾凇拼音方案，以及对应桌面的启动和输入模块集成；详见 [输入法：恢复依赖](INPUT-METHOD.md#恢复依赖)。Starship 还需要程序安装与交互 Shell 初始化；本机由 Omarchy 的 Bash 初始化脚本负责。详见 [Starship：恢复依赖与 Shell 接入](STARSHIP.md#恢复依赖与-shell-接入)。

远程仓库已建立；新机器的 `~/dotfiles` 尚不存在时，可以按以下流程克隆并初始化：

```bash
git clone https://github.com/szchan/dotfiles.git "$HOME/dotfiles"
chezmoi init --source "$HOME/dotfiles"
chezmoi diff
```

初始化时根据新机器是否运行对应代理选择开关。检查差异，确认不会覆盖需要保留的本机配置后再部署：

```bash
chezmoi apply
chezmoi diff
```

随后在家目录运行 `mise install`，打开 Neovim 完成插件安装，必要时执行 `:Lazy restore`；检查字体解析和应用启动。应用配置前后仍需关注 Git 身份、Ghostty 的 Linux 设置和 Omarchy 主题依赖。当前没有自动安装或恢复这些依赖的 chezmoi 脚本。

对于没有 Omarchy 的机器，应先补上 Neovim 备用主题等条件配置，再采用上述全量应用流程。其他机器的实际恢复测试尚待记录。

## 7. 提交与验证记录

以下均为 2026-10-11 第一批配置的已完成提交：

| 提交 | 内容 |
| --- | --- |
| `e998864` | 初始化布局、文档及 Ghostty/herdr 基线 |
| `0b88997` | Ghostty 注释编辑与应用练习 |
| `ecdfc78` | Git 配置及通过 PATH 查找 gh 的凭据助手 |
| `712f064` | Bash 模板与可选本地代理 |
| `d2c50b6` | 登录启动文件与可选依赖加载检查 |
| `cde0f52` | mise 工具配置 |
| `6cea8bd` | Neovim 配置、插件锁文件与主题链接 |
| `785e068` | 字体偏好与中文回退规则 |

本次验证为配置一致性、语法和文件结构检查；未执行新的安装、登录、插件升级或其他机器部署。可重复的语法检查示例：

```bash
chezmoi cat ~/.bashrc | bash -n
chezmoi cat ~/.bash_profile | bash -n
chezmoi cat ~/.profile | sh -n
```

代理关闭分支也已用临时数据渲染验证，没有修改本机选项：

```bash
chezmoi --override-data '{"enableProxy":false}' cat ~/.bashrc | bash -n
```

此外，以 Python 的 TOML、JSON、XML 解析器检查对应配置格式，并比较渲染内容和实际文件字节；这些检查不等同于全部应用的功能测试。

### GitHub 公开备份与许可证（2026-10-11）

- 添加根目录 MIT [LICENSE](../LICENSE)，版权所有者为 Shaoze Chen，年份为 2026；中英文 README 均链接到许可证。
- 原创配置改动和文档使用 MIT；第三方文件保留各自许可证，包括 Neovim starter 的 [Apache-2.0 许可证](../home/dot_config/nvim/LICENSE)。
- 许可证提交为 `919c423`，已随完整本地提交历史首次推送到公开仓库 [szchan/dotfiles](https://github.com/szchan/dotfiles)。
- 已核对 GitHub 可见性为 `public`、许可证识别为 `MIT`，首次推送后的远程 `main` 与本地提交一致；已设置 upstream。
- 发布前检查了 Git 历史中的 41 个文件对象，常见 token、私钥、含凭据 URL 和字面秘密赋值模式未发现匹配。这是模式检查结果，不代表能识别所有类型的秘密。

后续日常推送：

```bash
git -C ~/dotfiles push
```

### 第二批 SSH 与输入法（2026-10-11）

SSH 模板及相关文档已在 `89603c1` 中提交，核对时 GitHub `main` 与本地提交一致。两台服务器的真实登录由用户验证。输入法五个文件的导入范围、内容一致性及权限已核对；本轮没有修改行为、重启输入法或执行重新部署。分别见 [SSH 验证记录](SSH.md#验证记录) 和 [输入法验证记录](INPUT-METHOD.md#验证记录)。

### 输入法备份与 Starship 导入（2026-10-12）

输入法配置及文档已在 `d78a30e` 中提交；核对时 GitHub `main` 与本地一致，工作区干净。随后用户导入 Starship 普通配置，已确认内容一致、权限为 `644`、TOML 解析成功且目标 diff 为空。没有改动提示符偏好、重新应用配置或启动新的交互 Shell。见 [Starship 验证记录](STARSHIP.md#验证记录)。

## 8. 后续事项

1. 保持远程备份更新：配置修改提交后，推送到已建立的 GitHub 仓库。
2. 提交并推送已导入的 Starship 配置及本次文档；个人学习词库以后单独制定私密备份方案。
3. 按其他应用需要继续加入 1Password 引用模板，不把秘密值提交到源仓库。
4. 为非 Omarchy 或其他系统补齐主题、工具选择和个人身份的条件配置。
5. 在第二台机器实际恢复并记录结果，逐步接入 Hyprland 等桌面配置。

本文保存初始化与配置维护流程；第二批的详细记录见 [SSH.md](SSH.md)、[INPUT-METHOD.md](INPUT-METHOD.md) 和 [STARSHIP.md](STARSHIP.md)。后续改动应同步更新相关段落和核查日期。
