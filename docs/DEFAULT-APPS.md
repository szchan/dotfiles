# 默认终端与应用关联

核查日期：2026-10-12（Asia/Singapore）。当前验证范围为本机 Omarchy/Linux；概览见 [SETUP.md](SETUP.md)。

## 管理的三个文件

| 源文件 | 实际文件 | 类型与权限 |
| --- | --- | --- |
| [xdg-terminals.list](../home/dot_config/xdg-terminals.list) | `~/.config/xdg-terminals.list` | 普通文件，`644` |
| [mimeapps.list](../home/dot_config/mimeapps.list) | `~/.config/mimeapps.list` | 普通文件，`644` |
| [clash-verge-handler.desktop](../home/dot_local/share/applications/clash-verge-handler.desktop) | `~/.local/share/applications/clash-verge-handler.desktop` | 普通 desktop 入口，`644` |

这里只保存默认应用选择和协议入口，不包含浏览器资料、应用账号、邮件数据或 Clash 的订阅与认证配置。

## 默认终端

`xdg-terminals.list` 中的有效条目为：

```text
com.mitchellh.ghostty.desktop
```

`xdg-terminal-exec` 按偏好顺序选择存在且有效的终端入口。本机 Omarchy 的终端启动脚本使用该程序，因此此文件保存了 Ghostty 作为默认终端的选择。[项目配置说明](https://github.com/Vladimir-csp/xdg-terminal-exec#configuration)

## 浏览器与协议关联

`mimeapps.list` 当前保存的显式默认值：

| 类型或协议 | Desktop ID |
| --- | --- |
| `text/html`、`http`、`https`、`about`、`unknown` | `microsoft-edge.desktop` |
| `clash`、`clash-verge` | `clash-verge-handler.desktop` |
| `codex` | `chatgpt.desktop` |

本机原有的两个 Clash 协议曾指向不存在的 `clash-verge.desktop`。本轮改为实际有效的 `clash-verge-handler.desktop`，并备份该用户级入口。入口使用 `Exec="/usr/bin/clash-verge" %u`，`NoDisplay=true`，用于处理协议而不额外显示应用菜单项目。

用户明确选择移除过期 Omamail 关联，暂不指定邮件客户端：

- 从 `[Default Applications]` 删除 `x-scheme-handler/mailto=omamail.desktop`。
- 在 `[Removed Associations]` 保留以下规则，以排除该 desktop 入口宣告的邮件处理能力。

```ini
[Removed Associations]
x-scheme-handler/mailto=omamail.desktop;
```

Omamail 原 desktop 入口仍在本机，但其启动脚本已经不存在；本轮没有删除该入口或安装替代邮件程序。显式排除足以从邮件关联选择中去掉它。[添加与移除关联规则](https://specifications.freedesktop.org/mime-apps/latest/associations.html)

没有显式 mailto 默认值时，系统可以选择其他可用处理程序。本次查询返回 `HEY.desktop`，这是本机自动选择结果，未写入共享配置，也不作为必须安装的恢复依赖。[默认应用规则](https://specifications.freedesktop.org/mime-apps/latest/default.html)

## 恢复依赖

恢复时应提供或调整下列入口：

- Ghostty 的 `com.mitchellh.ghostty.desktop`。
- Microsoft Edge 的 `microsoft-edge.desktop`。
- Clash Verge 程序 `/usr/bin/clash-verge`；本仓库部署用户级协议入口。
- 处理 `codex` 协议的 `chatgpt.desktop`。

本机已确认这些依赖可用。基础工具核查快照为 `xdg-terminal-exec 0.14.3-1`、`xdg-utils 1.2.1-2`；本机 Clash Verge 包为 `clash-verge-rev-bin 2.5.8-1`。版本不是恢复时必须固定的要求。

应用程序需另外安装。其他机器若使用不同浏览器、终端或协议处理程序，应先调整共享选择，或在有实际需求时引入机器选项；当前文件没有机器模板。

## 操作与验证

本次用户先导入三个文件，再通过 `chezmoi edit ~/.config/mimeapps.list` 修改源关联并应用：

```bash
chezmoi add \
  ~/.config/xdg-terminals.list \
  ~/.config/mimeapps.list \
  ~/.local/share/applications/clash-verge-handler.desktop
EDITOR=nvim chezmoi edit ~/.config/mimeapps.list
chezmoi diff ~/.config/mimeapps.list
chezmoi apply ~/.config/mimeapps.list
```

后续通过 GUI 更改默认应用时，先核对实际文件及是否出现桌面专用的覆盖文件，再针对需要保存的普通文件重新导入。

只读查询示例：

```bash
xdg-mime query default x-scheme-handler/https
xdg-mime query default x-scheme-handler/clash
xdg-mime query default x-scheme-handler/clash-verge
xdg-mime query default x-scheme-handler/mailto
xdg-terminal-exec --print-id
desktop-file-validate ~/.local/share/applications/clash-verge-handler.desktop
```

`--print-id` 只输出选择的 terminal desktop ID，不启动终端。Agent 核查时将终端选择器的缓存目录临时设在 `/tmp`，没有写入本机原缓存。

## 验证记录

只读检查确认：

- 本轮新增范围恰好是上述三个普通文件，均与实际文件逐字节一致，权限为 `644`。
- 关联文件和 desktop 入口可解析；`desktop-file-validate` 检查通过。
- 三个目标的 `chezmoi diff` 无输出。
- `xdg-terminal-exec --print-id` 返回 `com.mitchellh.ghostty.desktop`。
- 两个 Clash 协议均返回 `clash-verge-handler.desktop`；HTTPS 返回 `microsoft-edge.desktop`，`codex` 返回 `chatgpt.desktop`。
- 源配置没有 mailto 显式默认值，Omamail 排除规则存在；实际邮件查询返回 `HEY.desktop`。

以上核查没有启动 GUI 应用、打开 URL、邮件或订阅链接。本轮没有执行新的应用安装或账号登录；新机器恢复测试尚未进行。本次新增配置及文档待用户提交、推送。
