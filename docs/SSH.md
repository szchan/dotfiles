# SSH 与 1Password 配置

核查日期：2026-10-11（Asia/Singapore）。当前验证范围为本机 Omarchy/Linux；概览见 [SETUP.md](SETUP.md)。

## 配置结构

公开仓库只保存 1Password 引用。服务器别名、地址和登录账号存放在 1Password 的 `SSH hosts` 安全备注中，使用六个独立的 Text 自定义字段：

| 字段 | 用途 |
| --- | --- |
| `host1_alias` | 第一台服务器的 SSH 别名 |
| `host1_hostname` | 第一台服务器的地址 |
| `host1_user` | 第一台服务器的登录账号 |
| `host2_alias` | 第二台服务器的 SSH 别名 |
| `host2_hostname` | 第二台服务器的地址 |
| `host2_user` | 第二台服务器的登录账号 |

这些是独立字段，不是备注正文中的六行文字。编辑条目时通过 **Add More → Text** 添加字段，保存后通过字段菜单复制 **Copy Secret Reference**。[自定义字段说明](https://support.1password.com/custom-fields/)

两台服务器当前使用同一把已有 SSH 密钥。公钥模板引用 SSH Key 条目的 `public key` 字段；私钥不导出到文件。

| 源文件 | 部署文件 | 已验证权限 |
| --- | --- | --- |
| [private_config.tmpl](../home/private_dot_ssh/private_config.tmpl) | `~/.ssh/config` | `600` |
| [private_id_ed25519_1password.pub.tmpl](../home/private_dot_ssh/private_id_ed25519_1password.pub.tmpl) | `~/.ssh/id_ed25519_1password.pub` | `600` |
| `home/private_dot_ssh/` | `~/.ssh/` | `700` |

`private_` 控制目标权限，不加密仓库中的内容。源目录和模板自身可以是 `755`、`644`；属性前缀决定部署权限。`private_dot_ssh` 没有 `exact_` 属性，部署时不会因此清除目录中的其他文件。

## 模板与 agent 的分工

`onepasswordRead` 通过 `op` 读取指定字段。主模板读取六个服务器字段，公钥模板读取已有 SSH Key 条目的公钥，并用 `trim` 去除首尾空白。[函数说明](https://www.chezmoi.io/reference/templates/1password-functions/onepasswordRead/)

应用后，实际 `~/.ssh/config` 是包含服务器信息的普通文本；公钥文件包含实际公钥。这些运行时文件与 Git 中的引用模板是两份文件。

每个服务器条目均配置：

```sshconfig
IdentityFile ~/.ssh/id_ed25519_1password.pub
IdentitiesOnly yes
```

通用设置保留：

```sshconfig
Host *
    IdentityAgent ~/.1password/agent.sock
```

`IdentityFile` 中的公钥用于选择 agent 内对应的密钥，签名由 1Password agent 完成。[1Password 官方说明](https://www.1password.dev/ssh/agent/advanced)

只管理上述配置和公钥模板。`known_hosts`、旧备份、原有公钥和 1Password 自动管理的 `~/.ssh/1Password/` 不属于这次导入；本轮不改变它们的管理方式。

## 恢复依赖

完整恢复前需要：

- 安装 1Password 桌面应用、1Password CLI（`op`）及 chezmoi。
- 在桌面应用中登录对应账号，启用 CLI 集成和 SSH agent。
- 可以访问源模板引用的保险库、`SSH hosts` 条目及 SSH Key 条目；密钥需要可由 agent 使用。
- 当前模板适用于 Linux 的 `~/.1password/agent.sock` 路径。macOS 等其他系统需先确认实际路径并修改模板。

本仓库初始化模板已声明 `[onepassword] mode = "account"` 和 `prompt = false`。该设置关闭 chezmoi 自己的登录提示逻辑；桌面集成的授权由 `op` 与 1Password 应用处理。

条目名称和字段名称参与当前引用路径。重命名、移动或删除条目后，需要检查引用；不通过把实际值写回 Git 来解决读取失败。

## 日常维护与验证

编辑 SSH 配置使用源模板：

```bash
EDITOR=nvim chezmoi edit ~/.ssh/config
chezmoi diff ~/.ssh/config ~/.ssh/id_ed25519_1password.pub
chezmoi apply ~/.ssh/config ~/.ssh/id_ed25519_1password.pub
chezmoi diff ~/.ssh/config ~/.ssh/id_ed25519_1password.pub
```

服务器信息变更在 1Password 字段中维护，再预览并应用。公钥更换需要同步选择的 SSH Key 引用及服务器的授权公钥；只修改本机模板不等于更新服务器。

不要直接重新导入渲染后的配置或公钥来覆盖模板。`chezmoi diff`、`chezmoi cat` 的输出可能包含实际服务器信息，检查时避免将其写入公开文档和日志。

本机解析检查可以使用 `ssh -G <别名>`，它不连接服务器。真实登录测试由用户执行，成功后用 `exit` 返回本机。

## 授权复用

1Password 默认按“应用＋密钥”请求授权，并记住到 1Password 锁定；可在 **Settings → Developer → SSH agent 高级设置**调整授权范围和记忆时长。同一应用再次使用同一把密钥连接另一台服务器，可以复用授权。[授权说明](https://www.1password.dev/ssh/agent/authorization)

本机具体的高级授权选项尚未核对。本次第一次登录出现授权、第二次没有出现，符合授权复用行为，不能据此断言本机采用了哪一组选项。

## 验证记录

Agent 已只读检查：

- 两份模板均成功从 1Password 读取，渲染内容与实际文件逐字节一致。
- 全局 `chezmoi diff` 无输出。
- `~/.ssh` 权限为 `700`，配置与新公钥为 `600`。
- 新公钥是有效的 ED25519 公钥，其密钥内容与原公钥一致。
- 两台服务器的 `ssh -G` 解析成功；服务器连接信息保持原值，通用公钥路径、`IdentitiesOnly yes` 和 agent 路径符合预期。
- 两个源模板没有实际服务器地址或私钥，保留了 1Password 引用。

用户确认两台服务器的真实 SSH 登录均成功：第一次弹出 1Password 授权，第二次没有弹出。Agent 未自行建立远程登录会话。

模板及相关文档已提交为 `89603c1`，用户完成推送；随后核对 GitHub `main` 与本地提交一致、工作区干净。其他机器的恢复测试尚未进行。
