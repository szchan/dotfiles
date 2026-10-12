# Omarchy 界面配置

核查日期：2026-10-12（Asia/Singapore）。本项仅适用于 Omarchy shell；概览见 [SETUP.md](SETUP.md)。

## 纳入的文件与偏好

| 源文件 | 实际文件 | 类型与权限 |
| --- | --- | --- |
| [private_shell.json](../home/dot_config/omarchy/private_shell.json) | `~/.config/omarchy/shell.json` | 普通 JSON 文件，`600` |

`private_` 保留部署权限，不加密公开仓库中的内容。配置主要保存：

- 状态栏放在顶部，背景不透明，中心锚点为 `omarchy.clock`。
- 左侧包含 Keystroke 和工作区；中间包含指示器、时钟、键盘布局、天气、Todoist、插件管理和系统更新。
- 右侧包含托盘、AI Usage、中文输入法、通知中心、Time Machine、Agent、蓝牙、网络、音频、显示器和电源组件。
- 时钟格式 `dddd HH:mm`，替代格式 `d MMMM 'W'ww yyyy`，竖向格式 `HH\n—\nmm`。
- `idle.screensaver = 150`，`idle.lock = 300`，单位为秒。
- AI Usage 的 `colorCodeUsage`、`showAll` 开启，`lastSelectedEntryId = "openai"`。
- `omarchy.lock` 与 `omarchy.menu` 在 `disabledPlugins` 中；`lock-explorer` 作为启动插件配置，`boot = "theme"`。
- `cloneSourceRestores` 记录 `lock-explorer` 与 `evindor.keystroke`。

本项不包含插件程序、账号认证、插件各自的数据文件或全部桌面状态。

## 第三方插件依赖

下表是配置实际引用的八个第三方插件。本机 manifest 均存在；版本是核查快照，来源链接来自各目录的 Git `origin`。

| 插件 ID | 名称 | 本机版本 | Git 来源 |
| --- | --- | --- | --- |
| `akitaonrails.ai-usagebar` | AI Usage | `1.32.0` | [ai-usagebar](https://github.com/akitaonrails/ai-usagebar) |
| `community.cn-input` | Chinese Input | `1.0.0` | [omarchy-cn-input-plugin](https://github.com/gohlihan/omarchy-cn-input-plugin) |
| `evindor.keystroke` | Keystroke | `1.5.0` | [keystroke](https://github.com/evindor/keystroke) |
| `io.github.aryan-techie.todoist` | Todoist | `1.16.0` | [omarchy-todoist](https://github.com/aryan-techie/omarchy-todoist) |
| `io.github.sirjul1337.lock-explorer` | Lock Screen Explorer | `1.9.6` | [omarchy-lock-explorer](https://github.com/SirJul1337/omarchy-lock-explorer) |
| `jankeesvw.notification-center` | Notification Center | `1.1.0` | [omarchy-notification-center](https://github.com/jankeesvw/omarchy-notification-center) |
| `jankeesvw.time-machine` | Time Machine | `1.0.0` | [omarchy-time-machine](https://github.com/jankeesvw/omarchy-time-machine) |
| `omaplug` | Plugin Manager | `1.6.6` | [omaplug](https://github.com/fross100/omaplug) |

其他已安装但没有被这份配置引用的插件，不因本次配置备份而一并纳入。用户插件目录整体没有导入；本轮也没有核对其全部本机源码修改。

系统级依赖快照为 `omarchy 4.0.4-1`、`quickshell 0.3.1-1`。新版本的配置兼容性应另行核对。

## 恢复次序

在另一台 Omarchy 机器恢复完整界面时：

1. 准备兼容的 Omarchy shell，以及需要启用的第三方插件。
2. 特别确认 `lock-explorer` 及菜单替代方案可用，因为此配置停用了默认 `omarchy.lock` 和 `omarchy.menu`。
3. 查看 `chezmoi diff ~/.config/omarchy/shell.json`，按新机器实际依赖调整配置后再应用。
4. 验证状态栏、菜单入口和锁屏行为。账号认证和插件自己的偏好或数据按各插件另外恢复。

只恢复 `shell.json` 不等于恢复上述插件及其运行状态。非 Omarchy 机器需先决定是否部署该文件；当前仓库还没有按操作系统或机器自动忽略它的规则。

## 导入与日常维护

本次由用户执行：

```bash
chezmoi add ~/.config/omarchy/shell.json
chezmoi diff ~/.config/omarchy/shell.json
git -C ~/dotfiles status --short --untracked-files=all
```

保存现有配置且 diff 为空时，无需 `apply` 或重启 shell。

后续编辑源配置：

```bash
EDITOR=nvim chezmoi edit ~/.config/omarchy/shell.json
chezmoi diff ~/.config/omarchy/shell.json
chezmoi apply ~/.config/omarchy/shell.json
```

Omarchy shell 会在实际配置保存后热加载；修改时按本机 Omarchy 指引操作。如果界面工具直接修改实际 JSON，核对差异后可针对这个普通文件重新 `chezmoi add`。

不要将导入一个界面文件扩大为导入整个 `~/.config/omarchy`。本轮不包含插件代码、缓存、会话或认证文件。

## 验证记录

只读检查确认：

- 本轮只新增 `home/dot_config/omarchy/private_shell.json`。
- 源文件与实际文件逐字节一致，JSON 解析成功，实际权限为 `600`。
- 目标 `chezmoi diff` 无输出。
- 八个第三方插件的本机 manifest 存在，已核对名称、版本与 Git 来源。
- 屏保和锁屏等待时间分别为 `150`、`300` 秒；保留了既有布局和插件启停设置。

本轮没有修改界面偏好、应用实际配置、安装插件、重启 shell 或触发锁屏测试。其他机器恢复与完整插件功能测试尚未进行。配置及相关文档已提交为 `ef563a3`，用户完成推送；2026-10-12 只读核对确认远程与本地一致，继续导入下一项前工作区干净。
