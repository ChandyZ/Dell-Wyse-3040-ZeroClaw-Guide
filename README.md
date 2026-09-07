# 低配置设备 Debian 13.6 和 ZeroClaw 部署教程

> **创建时间**：2026-09-01 20:46  
> **更新时间**：2026-09-03 23:28  
> **原文链接**：[https://github.com/ChandyZ/ChandyZ-3040/edit/main/README.md]

---

## 目录

- [背景介绍](#背景介绍)
- [硬件与系统](#硬件与系统)
- [准备工作](#准备工作)
  - [1) U盘制作](#1-u盘制作)
  - [2) BIOS 设置](#2-bios-设置)
- [Debian 13.6 安装全过程](#debian-136-安装全过程)
- [安装后必做的基础设置](#安装后必做的基础设置)
- [SSH 远程连接](#ssh-远程连接)
- [安装 ZeroClaw（预编译二进制）](#安装-zeroclaw预编译二进制)
- [ZeroClaw 首次配置（Quickstart）](#zeroclaw-首次配置quickstart)
  - [1) 基础配置流程](#1-基础配置流程)
  - [2) 界面汉化](#2-界面汉化)
- [接入限速 API 场景参数设置](#接入限速-api-场景参数设置)
- [ZeroClaw Lark 接入配置](#zeroclaw-lark-接入配置)
  - [1) 飞书开放平台准备](#1-飞书开放平台准备)
  - [2) 创建 lark 基础配置](#2-创建-lark-基础配置)
  - [3) 获取用户 open_id](#3-获取用户-open_id)
  - [4) 核心配置结构](#4-核心配置结构)
- [卸载与重新安装](#卸载与重新安装)
- [后记](#后记)
- [参考资源](#参考资源)

---

## 背景介绍

一台低配置设备，长期运行一个轻量 AI Agent。

以 **Dell Wyse 3040 + Debian 13.6 命令行系统 + ZeroClaw 0.8.4** 为例。

---

## 硬件与系统

| 项目 | 规格 |
|------|------|
| **设备型号** | Dell Wyse 3040 |
| **CPU** | Intel Atom x5-Z8350（64位，amd64） |
| **内存** | 2GB |
| **存储** | 16GB eMMC |

> **安装完成后，未执行任务状态下磁盘和内存使用情况（其他设备可参考占用情况）**

### 磁盘使用情况

| 项目 | 数值 |
|------|------|
| 总容量 | 13 GB |
| 已用空间 | 1.4 GB |
| 可用空间 | 11.6 GB |
| 使用率 | 12% |

### 内存（RAM）情况

| 项目 | 数值 |
|------|------|
| 总内存 | 1.8 GiB |
| 已用 | 51 MiB |
| 空闲 | 1.2 GiB |
| 缓存/缓冲 | 448 MiB |
| 可用 | 1.5 GiB |
| Swap | 789 MiB（未使用） |

**目标系统**：纯命令行 Debian 13.6（Trixie），无桌面，只装必要组件。

---

## 准备工作

### 1) U盘制作

- 准备一个至少 **4GB** 的 U 盘（会被清空，请提前备份）
- 下载 **Debian 13.6 amd64 netinst ISO**

> **下载地址**：<https://www.debian.org/CD/netinst/>  
> 进入页面找到「网络安装 CD 映像」，选择你需要的版本进行下载

#### Debian 13.6.0-amd64 版本文件 SHA256 校验

```text
ce0eeee7b51fdcdbed1e5116668c1fee27e528767bdf488e5f115a67b225e5dfd0afca1d456aaa9408ceb6b8527521ff7b6b5d62fdbe6f8c5faaf8df56a96292  debian-13.6.0-amd64-netinst.iso
64abfd17995054650374a7bb52be805389fd2369234b27af272efa6be24ad5697b332b3cf9ac790be0478bc5814d12d13a64b086ca7e54d521f107210cd818a3  debian-edu-13.6.0-amd64-netinst.iso
d2f76dccec6b9348a30a03e8c266b8c1f7837d9afaedb51bdf06672cde902d9789f42288c446b159a1d16d5e098e4e388716589641d0431515a158aac89cd1a9  debian-mac-13.6.0-amd64-netinst.iso
```

- 下载写入工具 **Rufus（Windows）**

> **下载地址**：<https://rufus.ie/>  
> 选择便携版本：`rufus-4.15p.exe`（约 1.9 MB，免安装直接运行）

#### 用 Rufus 将 ISO 文件写入 U 盘：

1. 插入 U 盘，双击打开 `rufus-4.15p.exe`
2. **设备**：确认选择的是你的 U 盘
3. **引导类型**：点击「选择」，找到并选中下载好的 `debian-13.6.0-amd64-netinst.iso`
4. **分区方案**：选择 **GPT**
5. **目标系统类型**：选择 **UEFI（非 CSM）**
6. **文件系统**：选择 **FAT32**（默认）
7. **簇大小**：保持默认即可
8. 点击 **开始** 按钮
9. 弹出提示时选择 **以 ISO 镜像模式写入**
10. 再次确认设备无误，点击 **OK** 2次开始写入
11. 等待进度条走完（约 2-5 分钟），显示 **「准备就绪」** 即完成
12. 点击 **关闭** 退出

> ⚠️ **注意**：必须做成 **UEFI 可启动** 的 U 盘

将 Wyse 3040 连接显示屏、鼠标、键盘，并插入 U 盘。

---

### 2) BIOS 设置

1. 开机按 **F2** 进入 BIOS，选择 **Unlock**，输入默认密码：**Fireport**
2. 在 **System Configuration** 目录下的 **USB Configuration** 勾选 **Enable USB Boot Support**
3. 保存退出后，开机按 **F12**，选择 **UEFI BOOT : 你制作并插入的 U 盘**

---

## Debian 13.6 安装全过程

启动后会看到安装菜单，请按以下选择：

1. 选择 **Install**。**不要选 Graphical install**，内存不够容易出问题
2. **语言、地区、键盘**：按自己习惯选择即可
3. **主机名**：随便起一个
4. **域名**：直接留空（按回车即可）

### 设置账号

- **设备名**：随便填
- **root 密码**：自己设置并记住，二次输入确认
- **用户名**：自己起
- **密码**：自己设置并记住（后面要用）

### 分区方式

- 选择：**向导 - 使用整个磁盘**
- 分区方案：**将所有文件放在同一个分区中**（推荐新手使用）
- 选择 **「完成分区操作并将修改写入磁盘」**

### 软件选择界面

- **仓库镜像所在国家**：选择设备所在的地址即可
- **仓库镜像站点**：都可以
- **代理**：没有，继续下一步

### 软件选择**只勾选以下两项**：

- [x] **SSH server**
- [x] **标准系统工具**

> ⚠️ **其他全部不要勾选**，尤其是：所有桌面环境（GNOME、Xfce 等）和 Web server

安装完成后拔掉 U 盘，确认重启，系统会进入命令行登录界面。

---

## 安装后必做的基础设置

刚装好的系统非常精简，缺少 `curl` 和 `sudo`，需要先补全。

### 1. 在机器上登录（用刚才创建的用户名和密码）

### 2. 切换到 root

```bash
su -
```

输入 root 密码（安装时设置的）。

### 3. 安装必要工具

```bash
apt update
apt install -y curl sudo
```

### 4. 把普通用户加入 sudo 组

```bash
usermod -aG sudo 用户名
```

### 5. 退出 root

```bash
exit
```

现在普通用户已经可以使用 `sudo` 了，如果不能用就重启设备。

---

## SSH 远程连接

### 1. 在 Wyse 3040 上查看 IP 地址

```bash
ip a
```

会看到类似 `xxx.xxx.xxx.xxx` 的地址，记下来。最后的 `xxx` 可能是 1-3 位数字，不是固定三位数。

### 2. 在你自己的电脑上打开终端

```bash
ssh 用户名@IP地址
```

### 3. 第一次连接会提示确认指纹

输入 `yes` 回车，然后输入密码即可登录。

> 连接成功后，以后所有操作都可以在自己电脑上远程完成。

---

## 安装 ZeroClaw（预编译二进制）

在 SSH 里执行：

```bash
curl -fsSL https://raw.githubusercontent.com/zeroclaw-labs/zeroclaw/master/install.sh | sh
```

安装过程会自动下载对应平台的预编译二进制。

### 如果下载失败，可尝试以下操作：

1. 在自己电脑上，用工具查询 `raw.githubusercontent.com` 的 IP 地址，记录下 IP 地址
2. 将下面的 `xxx.xxx.xxx.xxx` 替换为查到的 IP 地址并执行：

```bash
echo "xxx.xxx.xxx.xxx raw.githubusercontent.com" | sudo tee -a /etc/hosts
```

即可下载安装。安装完成后验证：

```bash
zeroclaw --version
```

如果显示版本号（例如 `zeroclaw 0.8.4`），说明安装成功。

---

## ZeroClaw 首次配置

### 1) 基础配置流程

```bash
zeroclaw quickstart
```

按提示填写：

| 配置项 | 建议/说明 |
|--------|-----------|
| **模型提供方** | 选择你用的云端 API 厂商 |
| **别名** | 自己取 |
| **模型** | 先选一个（后续可改配置文件） |
| **API Key 和 Base URL** | 填入对应信息 |
| **风险配置文件** | 推荐 **Balanced（平衡模式）**<br>Locked Down 太严格，YOLO 太放开 |
| **记忆后端** | 推荐 **sqlite**（最轻量，不需要额外数据库） |
| **通道** | 不要添加，后期再添加 |
| **对等组** | 先保持 **无** |
| **agent 身份** | 给这个 agent 起一个名字 |
| **系统提示词** | 可以先跳过，以后再写 |

全部填完后选择 **「创建 agent」**。

---

### 2) 界面汉化

下载中文翻译包：

```bash
zeroclaw locales fetch zh-CN
```

> 如果失败了，多试几次。

下载完成后，启动 TUI：

```bash
zerocode
```

操作步骤：
1. 选择 **config**
2. 点击 **zerocode**，选择 **locale**
3. 双击 **中文**
4. 按住 `Ctrl+C`，回车退出
5. 再次启动 TUI：`zerocode`

界面文字应变为中文。

---

## 接入限速 API 场景参数设置

如果你使用 **OpenRouter**、**nvidia nim** 等提供的免费模型，可参考以下参数配置 `runtime_profiles`，适合限速场景：

1. 启动 TUI 后，选择 **「配置」**
2. 找到 **sections 子目录 Foundation**，选择 **runtime_profiles**
3. 点击 `[+ Add]`，回车 2 次，输入新的配置名称（自己取一个）
4. 输入完成后，进入 **Fields** 界面，按下表设置：

```toml
agentic = true
max_actions_per_hour = 200
max_cost_per_day_cents = 0
max_delegation_depth = 2
max_tool_iterations = 30
shell_timeout_secs = 6000
strict_tool_parsing = true
delegation_timeout_secs = 900
agentic_timeout_secs = 1800
max_history_messages = 200
max_context_tokens = 128000         # 也可设为 256000 或更多，视模型支持和本地空间而定
compact_context = true
parallel_tools = false
max_system_prompt_chars = 64000
max_tool_result_chars = 64000
keep_tool_context_turns = 8
memory_recall_limit = 10
```

> **thinking.default_level**  
> 这个思考强度根据你的任务需求来选择。其他设置保持默认。

### 应用配置到 Agent

1. 找到 **sections 子目录 Foundation**，选择 **Agents**
2. 点击 **General**，找到 **Fields** 下的 **runtime_profiles** 双击
3. 点击刚设置好的配置，回车两次确认

---

## ZeroClaw Lark 接入配置

### 1) 飞书开放平台准备

- 创建企业自建应用
- 开通必要权限（接收消息、发送消息、获取用户信息等）
- 配置事件订阅（长连接 websocket 或 webhook）
- 发布版本并确保应用可用

> 以上直接选择飞书（Lark）开放平台的 **自建应用**（针对 opencode 和 hermes 的），即可一键成功创建。

创建成功后，获取 **app_id** 和 **app_secret**。

---

### 2) 创建 lark 基础配置

1. 启动 TUI 后，选择 **「配置」**
2. 找到 **sections 子目录 Foundation**，选择 **Channels**
3. 双击 **lark**，点击 `[+ Add]`，回车 2 次，输入新的配置名称（自己取一个）
4. 输入完成后，进入 **Behavior** 界面：
   - 将 `enabled` 改为 `true`
5. 点击 **Connection**，将刚获取的 `app_id` 和 `app_secret` 分别输入对应位置

---

### 3) 获取用户 open_id

把机器人拉进需要的群，或直接私聊测试。随便发一条消息给机器人，然后在 **日志** 里查找警告或错误：

```text
MS: Ignoring (not in peer group)
```

找到描述内容中的 `"sender_open_id": "ou_xxxx"`，记下 `"ou_xxxx"`。

> **open_id 格式**：`ou_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

---

### 4) 核心配置结构

按下 `Ctrl + C` 回车退出 TUI。

编辑配置文件：

```bash
nano ~/.zeroclaw/config.toml
```

找到 Lark 相关配置内容，按如下结构配置：

```toml
[channels.lark.自定义]
app_id = "cli_xxxxxxxx"
app_secret = "xxxxxxxx"
encrypt_key = ""
verification_token = ""
```

接着配置 **Peer Groups（最关键）**：

将下面自定义内容替换为自己输入过和获取的内容，粘贴在 `[channels.lark.自定义]` 整个模块下方：

```toml
[peer_groups.lark_users]
channel = "lark.前文自定义内容"
agents = ["前文自定义的agent姓名"]
external_peers = ["前文获取的open_id"]
```

填写并粘贴完成后，检查完整 lark 模块应如下：

```toml
[channels.lark]

[channels.lark.自定义]
enabled = true
app_id = "自定义"
app_secret = "自定义"
receive_mode = "websocket"

[peer_groups.lark_users]
channel = "lark.前文自定义内容"
agents = ["前文自定义的agent姓名"]
external_peers = ["前文获取的"]
```

改完后 `Ctrl + S` 保存，`Ctrl + X` 退出。再执行：

```bash
zeroclaw service restart
```

这样就配置好了。可用 `zerocode` 指令启动 TUI 去检查日志，无报错就成功了。

---

## 卸载与重新安装

如果需要卸载后重装：

### 停止服务

```bash
zeroclaw service stop 2>/dev/null || true
zeroclaw service uninstall 2>/dev/null || true
```

### 删除二进制

```bash
rm -f ~/.cargo/bin/zeroclaw
rm -f ~/.cargo/bin/zerocode
```

### 删除配置和数据（可选，会清空之前的配置）

```bash
rm -rf ~/.zeroclaw
```

### 重新安装

```bash
curl -fsSL https://raw.githubusercontent.com/zeroclaw-labs/zeroclaw/master/install.sh | sh
```

---

## 后记

本教程基于 **2026 年 8 月底至 9 月初** 在真实 **Dell Wyse 3040** 上的完整操作记录整理。

整套流程在 **2GB 内存机器上可成功跑通**，系统保持极简后，ZeroClaw 运行压力很小，适合长期运行。

后续可以继续完善：如需多人使用，往 `external_peers` 追加 `open_id` 即可。无多余账号，未测试。

---

## 参考资源

| 编号 | 资源 | 链接 |
|------|------|------|
| [1] | ZeroClaw 官方仓库 | <https://github.com/zeroclaw-labs/zeroclaw> |
| [2] | ZeroClaw 官方 Peer Groups 文档 | `docs/book/src/channels/peer-groups.md` |
| [3] | Debian 13.6 下载 | <https://www.debian.org/releases/trixie/> |

---

> 如有问题或建议，欢迎在 GitHub Issues 中交流。
