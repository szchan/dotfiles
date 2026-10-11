# Fcitx5 / Rime 输入法配置

核查日期：2026-10-11（Asia/Singapore）。当前验证范围为本机 Omarchy/Linux。概览见 [SETUP.md](SETUP.md)。

## 纳入的五个文件

| 源文件 | 实际文件 | 保存的内容 | 当前权限 |
| --- | --- | --- | --- |
| [private_config](../home/dot_config/fcitx5/private_config) | `~/.config/fcitx5/config` | Fcitx5 行为设置，`DefaultPageSize=10`；其余注释项沿用默认值 | `600` |
| [private_profile](../home/dot_config/fcitx5/private_profile) | `~/.config/fcitx5/profile` | Default 分组、美式键盘和 Rime，默认输入法为 `rime` | `600` |
| [clipboard.conf](../home/dot_config/fcitx5/conf/clipboard.conf) | `~/.config/fcitx5/conf/clipboard.conf` | `TriggerKey` 和 `PastePrimaryKey` 值为空 | `644` |
| [xcb.conf](../home/dot_config/fcitx5/conf/xcb.conf) | `~/.config/fcitx5/conf/xcb.conf` | `Allow Overriding System XKB Settings=False` | `644` |
| [private_default.custom.yaml](../home/dot_local/share/fcitx5/rime/private_default.custom.yaml) | `~/.local/share/fcitx5/rime/default.custom.yaml` | 通过 `patch/schema_list` 选择 `rime_ice`（雾凇拼音） | `600` |

这些都是普通文件，没有 `.tmpl`。`private_` 使 chezmoi 保留目标的私有权限，不加密 Git 内容。

Rime 自定义补丁目前只有方案选择：

```yaml
patch:
  schema_list:
    - schema: rime_ice
```

沿用 `.custom.yaml` 的补丁结构追加个人偏好，避免直接修改上游方案文件或 `build/` 中的生成文件。[Rime 定制指南](https://github.com/rime/home/wiki/CustomizationGuide)

## 导入范围

本轮只导入表中的五个文件。以下内容未纳入公开仓库：

- Fcitx5 的 `conf/cached_layouts`；`notifications.conf` 当前只有注释，没有有效设置。
- Rime 的 `build/` 编译产物。
- `rime_ice.userdb/` 学习词库与 `sync/` 同步数据；它们可能包含个人输入词条，后续单独制定私密备份方案。
- Rime 的 `installation.yaml` 和 `user.yaml` 等安装标识及运行状态。
- 系统包提供的雾凇拼音方案、词典和其他上游文件。

尚未读取或发布学习词库的内容。没有使用 `--exact`，也没有整体导入 `~/.config/fcitx5` 或 Rime 用户目录。

## 恢复依赖

本机相关软件包的核查快照如下，版本不是恢复时必须固定的要求：

| 软件包 | 核查版本 | 用途 |
| --- | --- | --- |
| `fcitx5` | `5.1.22-1` | 输入法框架 |
| `fcitx5-configtool` | `5.1.15-1` | 图形配置工具 |
| `fcitx5-gtk` | `5.1.7-1` | GTK 集成 |
| `fcitx5-qt` | `5.1.15-1` | Qt 集成 |
| `fcitx5-rime` | `5.1.15-1` | Rime 前端 |
| `librime` | `1:1.17.0-5` | Rime 引擎 |
| `rime-ice-pinyin-git` | `r1001.da1fbe6-1` | 雾凇拼音方案和词典 |

已通过 `pacman -Qo` 确认 `/usr/share/rime-data/rime_ice.schema.yaml` 来自 `rime-ice-pinyin-git`，`/usr/share/rime-data/fcitx5.yaml` 来自 `fcitx5-rime`。

在其他机器恢复时，需要安装相应软件和方案，再应用配置。本次没有管理桌面自启动或输入模块环境设置，必须按目标桌面及应用的实际情况另行配置。[Fcitx5 设置说明](https://fcitx-im.org/wiki/Setup_Fcitx_5)

Fcitx5 配置路径适用于该框架；其他 Rime 前端的用户目录可能不同，不能直接把本机路径视为跨平台通用路径。恢复配置不包含个人学习词库。

## 导入和日常维护

本次由用户执行：

```bash
chezmoi add \
  ~/.config/fcitx5/config \
  ~/.config/fcitx5/profile \
  ~/.config/fcitx5/conf/clipboard.conf \
  ~/.config/fcitx5/conf/xcb.conf \
  ~/.local/share/fcitx5/rime/default.custom.yaml
```

随后只检查这五个目标：

```bash
chezmoi diff \
  ~/.config/fcitx5/config \
  ~/.config/fcitx5/profile \
  ~/.config/fcitx5/conf/clipboard.conf \
  ~/.config/fcitx5/conf/xcb.conf \
  ~/.local/share/fcitx5/rime/default.custom.yaml
```

导入现有配置且 diff 为空时，不需要再次 `apply` 或重启输入法。

后续可以用 Fcitx5 配置工具调整偏好，再针对实际变化的普通配置文件重新导入。也可以编辑源文件，例如：

```bash
EDITOR=nvim chezmoi edit ~/.local/share/fcitx5/rime/default.custom.yaml
chezmoi diff ~/.local/share/fcitx5/rime/default.custom.yaml
chezmoi apply ~/.local/share/fcitx5/rime/default.custom.yaml
```

对 Rime 补丁的实际修改需要通过前端重新部署才能生效；本次没有改变设置，因此没有执行重新部署或功能变更测试。

## 验证记录

只读检查确认：

- 新增范围恰好为上述五个文件，没有学习词库、同步数据、编译产物或缓存。
- 源文件与当前实际文件逐字节一致，五个目标的 `chezmoi diff` 无输出。
- `config`、`profile` 和 Rime 补丁通过 `private_` 保留 `600` 权限；两份 addon 配置保持 `644`。
- 五个文件均为普通文件，内容与此前审阅的输入法偏好相符。
- 核对了软件包版本和系统方案文件的归属；没有安装软件、重启输入法、重新部署 Rime 或改动桌面环境。

以上是配置导入和一致性验证；其他机器的恢复及输入法功能验证尚未完成。
