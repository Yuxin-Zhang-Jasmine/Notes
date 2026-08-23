# Dell Precision 3591：Ubuntu 22.04 → 24.04 升级与故障修复记录

> 时间：2026-08-22 16:43 至 2026-08-23 11:50（Asia/Shanghai）<br>
> 设备：Dell Precision 3591<br>
> 目标：从 Ubuntu 22.04.5 LTS 原地升级到 Ubuntu 24.04.4 LTS，并恢复 NVIDIA、网络、固件更新以及 ChatGPT/Codex 桌面环境。

## 阅读说明

这是一份面向未来自己的实机维护记录，由以下三个会话按主线时间和故障主题合并而成：

- `Ubuntu安装建议`
- `Branche · Ubuntu安装建议`
- `登录失败处理`

分支会话与主会话有大量重复内容，本文已去重，只保留分支中新增的系统体检和 `firmware-updater` 补装记录。

为了适合公开到 GitHub，本文已省略或替换用户名、主机名、Wi-Fi SSID、Clash 订阅信息、节点名称、Machine ID、设备 GUID 等信息。私网地址也做了模糊化。

> [!CAUTION]
> 本文中的版本号和版本锁定命令是这台机器在 **2026-08-22/23** 的历史记录，不是适用于所有电脑的通用教程。以后再次操作时，应先检查当前系统、内核和仓库中实际存在的版本。

## 目录

- [最终结果速览](#最终结果速览)
- [硬件与原始环境](#硬件与原始环境)
- [总体时间线](#总体时间线)
- [1. 刚格式化后的基础设置](#1-刚格式化后的基础设置)
- [2. 先完整更新 Ubuntu 22.04](#2-先完整更新-ubuntu-2204)
- [3. 补齐 Dell OEM 包并解除升级限制](#3-补齐-dell-oem-包并解除升级限制)
- [4. 正式升级 Ubuntu 22.04 → 24.04](#4-正式升级-ubuntu-2204--2404)
- [5. 第一次网络故障：已连接但没有 IPv4](#5-第一次网络故障已连接但没有-ipv4)
- [6. 安装 Clash Verge](#6-安装-clash-verge)
- [7. 升级后 NVIDIA 模块失配](#7-升级后-nvidia-模块失配)
- [8. 清理无效 OEM 软件源并检查系统](#8-清理无效-oem-软件源并检查系统)
- [9. 安装 ChatGPT、系统体检与补装 Firmware Updater](#9-安装-chatgpt系统体检与补装-firmware-updater)
- [10. Codex 登录 403 与独立发现的 libcurl 冲突](#10-codex-登录-403-与独立发现的-libcurl-冲突)
- [11. 为桌面应用配置 Clash Verge TUN](#11-为桌面应用配置-clash-verge-tun)
- [12. 固件更新与 DHCP 配置第二次覆盖](#12-固件更新与-dhcp-配置第二次覆盖)
- [13. 第二种 Wi-Fi 问号：Mihomo Fake-IP 造成误判](#13-第二种-wi-fi-问号mihomo-fake-ip-造成误判)
- [14. 旧项目在 Ubuntu 24.04 上的恢复计划](#14-旧项目在-ubuntu-2404-上的恢复计划)
- [15. 最终验证清单](#15-最终验证清单)
- [16. 有意保留的回退点](#16-有意保留的回退点)
- [17. 以后复查时的最小命令集](#17-以后复查时的最小命令集)
- [18. 关键经验](#18-关键经验)
- [19. 结论边界](#19-结论边界)

## 最终结果速览

| 项目 | 升级前/故障状态 | 最终确认状态 |
|---|---|---|
| 操作系统 | Ubuntu 22.04.5 LTS | Ubuntu 24.04.4 LTS |
| 内核 | 更新后为 `6.8.0-138-generic` | `6.8.0-138-generic` |
| Secure Boot | Enabled | Enabled |
| 独立显卡 | RTX 2000 Ada，升级后模块失配 | 正常，驱动 `580.173.02` |
| 核显 | Intel Meteor Lake-P / Arc，`i915` | 正常 |
| 网络 | 升级后无 IPv4、无默认路由 | DHCP、DNS、浏览器及联网检测均正常 |
| NetworkManager | 被强制使用外部 `dhclient` | 使用内置 `dhcp=internal` |
| APT/dpkg | OEM 源警告、旧包较多 | `apt-get check` 正常，`dpkg --audit` 无输出 |
| systemd | 曾保留非关键失败记录 | `systemctl --failed` 为 0 |
| BIOS/System Firmware | `1.22.0` | `1.25.1` |
| UEFI CA | 2011 | 2023 |
| UEFI dbx | `20241101` | `20260402` |
| Clash Verge | 未安装 | `2.5.2`，后续启用 Mihomo TUN |
| ChatGPT Linux 应用 | 未安装 | `26.818.41705` 安装成功 |
| Codex 登录 | 本地回调出现地区 403 | 终端 API 请求路径与 `curl` 依赖已恢复；会话中未明确确认最终登录成功 |

## 硬件与原始环境

- 电脑：Dell Precision 3591
- 架构：amd64
- 根分区：`/dev/nvme0n1p3`
- 根分区容量：约 1.9 TB，升级前仅使用约 15 GB
- 无线接口：`wlp0s20f3`
- 独立显卡：NVIDIA RTX 2000 Ada Generation Laptop GPU，约 8 GB 显存
- 核显：Intel Meteor Lake-P / Intel Arc
- Dell/OEM 组件：
  - `linux-oem-22.04d`
  - `oem-somerville-aron-meta`
  - `manage-distro-upgrade 2.2~22.04jiayi1`
  - `jiayi`、`somerville-aron` OEM 软件源组件

原 GitHub 项目的环境记录为：

```text
Python 3.9.7
PyTorch 1.10.1
CUDA Runtime 11.3.1
torchvision 0.11.2
```

会话中项目名称同时出现过 `DRUS` 和环境名/路径 `ddrm`，尚未核实是否指同一个项目。

## 总体时间线

| 时间 | 阶段 | 结果 |
|---|---|---|
| 08-22 16:43 起 | 刚格式化后的基础设置 | 跳过 Ubuntu Pro，修复 Chrome 软件源签名问题 |
| 17:26–17:49 | 补齐 Ubuntu 22.04 更新 | 完成 MOK 注册，Secure Boot 与 NVIDIA 正常 |
| 17:49–18:10 | 处理 Dell OEM 限制 | 移除 `manage-distro-upgrade`，恢复 `Prompt=lts` |
| 19:00–20:18 | 22.04 → 24.04 原地升级 | 成功进入 Ubuntu 24.04.4，保留 477 个旧包作为回退 |
| 20:19–21:46 | 第一次网络故障 | 改用 NetworkManager 内置 DHCP，恢复 IPv4 和默认路由 |
| 之后 | Clash 与 NVIDIA | Clash 可用；精确替换错误的 Jammy NVIDIA 内核模块 |
| 22:49 后 | APT/OEM 收尾 | 禁用无效 OEM 源，不执行 `autoremove` |
| 23:24–23:41 | 桌面应用与系统体检 | 安装 ChatGPT、Git、`firmware-updater`；系统检查正常 |
| 23:56–次日 00:09 | Codex 403 与 `curl` 故障 | 清除 Dell DCA Enabler，恢复系统 `libcurl` |
| 08-23 上午 | 固件更新 | BIOS/System Firmware、UEFI CA、UEFI dbx 全部成功 |
| 10:29–11:20 左右 | DHCP 配置再次覆盖 | 使 `internal` 稳定生效，清除 DCA 包配置，重启后未再生成 |
| 11:30–11:50 | Wi-Fi 问号但实际能上网 | 修复 Mihomo Fake-IP 对 Ubuntu 联网检测的误判 |

---

## 1. 刚格式化后的基础设置

### 1.1 Ubuntu Pro

首次设置中的 Ubuntu Pro 页面选择了：

```text
Skip for now
```

当时没有启用 Ubuntu Pro，这不影响普通 LTS 更新。

### 1.2 修复 Google Chrome 软件源签名

`apt update` 最初遇到 Google Chrome 仓库缺少签名密钥。历史上执行了：

```bash
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub \
  | sudo tee /etc/apt/trusted.gpg.d/google.asc >/dev/null

sudo apt update
```

随后截图中红色仓库错误消失。

下载源通过“软件和更新”选择较快镜像，后续实际使用华为云 Ubuntu 镜像。升级前没有先装代理，以减少升级过程中的网络变量。

## 2. 先完整更新 Ubuntu 22.04

没有直接开始跨版本升级，而是先更新当前系统：

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

更新涉及内核、Dell OEM、NVIDIA、GRUB 和基础系统组件。

### 2.1 MOK 与 Secure Boot

NVIDIA DKMS 更新触发 MOK 注册。操作顺序为：

```text
Enroll MOK
→ Continue
→ Yes
→ 输入临时 MOK 密码
→ Reboot
```

蓝色 MOK 页面输入密码时不显示字符或星号，属于正常行为。

重启后的关键状态：

```text
Ubuntu 22.04.5 LTS
6.8.0-138-generic
SecureBoot enabled
NVIDIA Driver Version: 580.173.02
GPU: NVIDIA RTX 2000 Ada
```

`nvidia-smi` 中的 `CUDA Version: 13.0` 表示驱动支持的最高 CUDA 兼容版本，不代表系统已经安装 CUDA Toolkit 13.0。

## 3. 补齐 Dell OEM 包并解除升级限制

### 3.1 更新 OEM 过渡包

当时还有以下 OEM 包待更新：

```text
linux-headers-oem-22.04d
linux-image-oem-22.04d
linux-oem-22.04d
oem-somerville-aron-meta
```

先模拟：

```bash
sudo apt -s install linux-oem-22.04d oem-somerville-aron-meta
```

确认不会卸载软件包后，正式执行：

```bash
sudo apt install \
  linux-oem-22.04d \
  linux-headers-oem-22.04d \
  linux-image-oem-22.04d \
  oem-somerville-aron-meta

sudo reboot
```

### 3.2 找到 `do-release-upgrade` 被禁止的原因

检查命令：

```bash
dpkg-divert --list | grep release-upgrades
apt-cache policy manage-distro-upgrade
grep -Ev '^[[:space:]]*(#|$)' /etc/update-manager/release-upgrades
```

关键证据：

```text
manage-distro-upgrade 已接管 /etc/update-manager/release-upgrades
[DEFAULT]
Prompt=never
```

Dell OEM 包 `manage-distro-upgrade` 通过 `dpkg-divert` 阻止发行版升级。

先模拟卸载：

```bash
sudo apt -s purge manage-distro-upgrade
```

模拟结果只移除该包本身，没有连带移除 OEM 或 NVIDIA 关键组件。随后正式执行：

```bash
sudo apt purge manage-distro-upgrade
```

再次验证：

```bash
dpkg-divert --list | grep release-upgrades
grep -Ev '^[[:space:]]*(#|$)' /etc/update-manager/release-upgrades
sudo do-release-upgrade -c
```

结果恢复为：

```text
[DEFAULT]
Prompt=lts

有新版本“24.04.4 LTS”可供使用
```

> 这一步实质上绕过了 Dell OEM 原先设置的升级限制，因此升级后需要额外验证指纹、摄像头、蓝牙、音频、休眠唤醒、外接显示器等硬件功能。

## 4. 正式升级 Ubuntu 22.04 → 24.04

### 4.1 磁盘检查

```bash
df -h /
```

当时结果约为：

```text
/dev/nvme0n1p3  1.9T  15G  1.8T  1%  /
```

### 4.2 启动升级

```bash
sudo do-release-upgrade
```

升级程序明确显示目标为 Ubuntu 24.04 LTS “Noble Numbat”，没有升级到开发版或非 LTS 版本。

最终软件包摘要中约有 155 个旧包将被删除。先按 `d` 查看详情，检查到：

- 被删除的主要是 Jammy 旧库；Noble 正安装对应的 `t64` 新库。
- PulseAudio 迁移到 PipeWire。
- 新的 `linux-generic`、GRUB、GNOME、NetworkManager、内核和固件会被安装或更新。
- NVIDIA 580、DKMS 和显卡库被保留/升级。
- `ubuntu-desktop`、`network-manager`、GRUB/shim、NVIDIA 580 和 Dell/OEM 元包没有出现在关键删除项中。

随后：

```text
q  # 退出详情
y  # 开始正式升级
```

升级下载速度约 11.5 MB/s，总过程预估 40–80 分钟。当时建议始终插电、保持开盖、禁用自动挂起，并避免运行第二个 APT 或更新器；会话没有完整确认自动挂起设置最终是否确实关闭。

### 4.3 升级中的非致命提示

以下提示均未阻断升级：

- Thunderbird 迁移为 Snap，确认继续。
- 中文界面把“6分43秒”误译为“6月43秒”。
- `fontconfig-config` 暂时未安装，随后成功覆盖。
- DKMS 目录非空警告。
- `gnome-remote-desktop`、`polkitd` 用户/组在包替换阶段暂时不存在。
- `cron` 旧包移除后，新包随即安装。
- `acpid.socket canceled`。
- Firefox 最终出现 `Snap installation complete`。
- 创建 `dhcpcd` 系统用户时无法创建 `/usr/lib/dhcpcd` 主目录。
- Python 3.12 对 gedit、HPLIP、Rhythmbox 旧插件给出 `SyntaxWarning`。
- GRUB 两次重新生成都以 `done` 结束，并保留：
  - `6.8.0-138-generic`
  - `6.5.0-1023-oem` 回退内核
- EFI 模式下 Memtest86+ 不可用、`os-prober` 未执行。
- `firmware-updater` Snap 因当时 DNS 暂时不可用而下载失败，不影响发行版升级。

升级尾声询问是否删除 477 个陈旧软件包时，选择默认：

```text
N
```

原因是磁盘空间充足，先保留旧内核和旧组件作为回退点。看到“系统升级完成”后输入 `y`，由升级程序自动重启。

结果：成功进入 Ubuntu 24.04.4 桌面。

## 5. 第一次网络故障：已连接但没有 IPv4

### 5.1 症状

升级重启后：

- Wi-Fi 和有线网络都可能显示“已连接”，但图标带问号。
- 有线显示 1,000 Mb/s 只证明物理链路存在。
- 浏览器不能访问互联网。
- `ping 223.5.5.5` 返回“网络不可达”。
- 无有效 IPv4 地址。
- `ip route` 为空。
- Wi-Fi 有 IPv6，因此 NetworkManager 仍可能显示连接已激活。

这不是单纯 DNS 故障。

### 5.2 诊断

```bash
ping -c 3 223.5.5.5
getent hosts www.baidu.com
ip route
ls -l /etc/resolv.conf
cat /etc/resolv.conf
systemctl is-active systemd-resolved

nmcli device status
ip -4 addr
ip route
```

进一步检查：

```bash
ip -4 addr show wlp0s20f3
ip route

nmcli -g \
  ipv4.method,ipv4.may-fail,ipv4.dhcp-client-id,802-11-wireless.cloned-mac-address \
  connection show "YOUR_WIFI_CONNECTION"

sudo /usr/sbin/NetworkManager --print-config \
  | grep -iE 'dhcp=|dns='

command -v dhclient

sudo journalctl -b -u NetworkManager \
  --since "30 minutes ago" --no-pager \
  | grep -Ei 'wlp0s20f3|dhcp4|dhcp|timeout|fail|error' \
  | tail -n 100
```

关键输出：

```text
ipv4.method = auto
ipv4.may-fail = yes
dns=systemd-resolved
dhcp=dhclient
/usr/sbin/dhclient
```

家庭 Wi-Fi 和手机热点都正常发放 DHCP 租约，但日志紧接着出现：

```text
DHCPACK ...
execute (/usr/libexec/nm-dhcp-helper, ...): Permission denied
```

结论：

- 网卡、驱动、路由器和热点均正常。
- 外部 `dhclient` 已收到租约。
- `nm-dhcp-helper` 因权限或安全策略原因无法把 IPv4 和默认路由写入系统；具体执行机制没有进一步定位。

### 5.3 第一次切换到内置 DHCP

先创建配置：

```bash
sudo install -d -m 755 /etc/NetworkManager/conf.d

printf '[main]\ndhcp=internal\n' \
  | sudo tee /etc/NetworkManager/conf.d/99-dhcp-internal.conf

sudo systemctl restart NetworkManager
sudo nmcli connection up "YOUR_WIFI_CONNECTION"
```

检查仍显示 `dhcp=dhclient`，说明配置被后加载的文件覆盖。

查找所有 DHCP 配置：

```bash
sudo grep -RInsE \
  '^[[:space:]]*dhcp[[:space:]]*=' \
  /etc/NetworkManager \
  /run/NetworkManager \
  /usr/lib/NetworkManager \
  /var/lib/NetworkManager \
  2>/dev/null

sudo /usr/sbin/NetworkManager --print-config | sed -n '1,35p'
```

发现：

```text
/etc/NetworkManager/conf.d/99-dhcp-internal.conf: dhcp=internal
/etc/NetworkManager/conf.d/dhcp-client.conf: dhcp=dhclient
```

后者覆盖了前者。

采用可恢复方式禁用旧配置：

```bash
sudo mv \
  /etc/NetworkManager/conf.d/dhcp-client.conf \
  /etc/NetworkManager/conf.d/dhcp-client.conf.disabled

sudo systemctl restart NetworkManager
sudo nmcli connection up "YOUR_WIFI_CONNECTION"
```

验证：

```bash
sudo /usr/sbin/NetworkManager --print-config | grep -i '^dhcp='
ip -4 addr show wlp0s20f3
ip route
ping -c 3 223.5.5.5
ping -c 3 www.baidu.com
```

结果：

```text
dhcp=internal
inet 192.168.1.x/24
default via <LAN_GATEWAY> dev wlp0s20f3
```

IP 和域名 ping 均为 0% 丢包，浏览器恢复正常。

## 6. 安装 Clash Verge

网络恢复后安装 Clash Verge Rev `2.5.2`：

```bash
cd ~/下载
sudo apt install ./Clash.Verge_2.5.2_amd64.deb
```

本地 `.deb` 出现 `_apt` 无法以低权限读取下载目录的提示，但安装已成功，不是错误。

初始安全配置：

- 系统代理：开启
- TUN/虚拟网卡：先关闭
- 局域网连接：关闭
- DNS 覆写：关闭
- 开机自启：暂不启用
- 建议使用规则模式并选择可用节点；会话只确认之后 Clash 已可用，未由截图单独验证此项
- 不使用 `sudo` 启动 Clash

用户确认 Clash 已经可以使用。

## 7. 升级后 NVIDIA 模块失配

### 7.1 症状

```text
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver
```

同时：

- Secure Boot 仍启用。
- Intel `i915` 仍能驱动桌面。
- `lspci` 可识别 RTX 2000 Ada。
- `lsmod` 中没有 NVIDIA/nouveau 模块。
- `modinfo` 可找到 NVIDIA 580.173.02 模块文件。
- `sudo modprobe nvidia` 返回 `Invalid argument`。

### 7.2 根因

`dkms status`：

```text
nvidia/580.173.02, 6.8.0-138-generic, x86_64:
installed (WARNING! Diff between built and installed module!)
```

包版本出现混合：

- 内核和头文件：Noble `6.8.0-138.138`
- NVIDIA 用户态：`580.173.02-0ubuntu0.24.04.1`
- 实际内核模块/对象：`6.8.0-138.138~22.04.1`，即 Jammy 残留

内核日志：

```text
nvidia: no symbol version for module_layout
nvidia: disagrees about version of symbol sme_me_mask
nvidia: Unknown symbol sme_me_mask (err -22)
```

模块签名为 Canonical 签名，因此根因不是 Secure Boot 拒签，而是 22.04 模块与 24.04 内核 ABI 不匹配。

### 7.3 精确修复

> 以下版本锁定命令仅记录当时实际修复。未来不要不经检查直接照抄。

```bash
sudo apt update

sudo apt install \
  linux-modules-nvidia-580-6.8.0-138-generic=6.8.0-138.138 \
  linux-objects-nvidia-580-6.8.0-138-generic=6.8.0-138.138 \
  linux-signatures-nvidia-6.8.0-138-generic=6.8.0-138.138 \
  linux-modules-nvidia-580-generic-6.8

sudo depmod -a
sudo update-initramfs -u -k 6.8.0-138-generic
sudo reboot
```

实际变更：

- 升级 3 个错误的 `~22.04.1` NVIDIA 包。
- 新装 `linux-modules-nvidia-580-generic-6.8`。
- 移除 3 个旧过渡包：
  - `linux-modules-nvidia-535-generic-hwe-22.04`
  - `linux-modules-nvidia-535-oem-22.04d`
  - `linux-modules-nvidia-580-generic-hwe-22.04`

重启后：

- `nvidia-smi` 恢复。
- `nvidia`、`nvidia_drm`、`nvidia_modeset`、`nvidia_uvm` 均已加载。
- `nvidia-smi` 中可见 Xorg 打开了 NVIDIA 设备；这不代表混合显卡桌面的主渲染一定切换到了独显。
- Secure Boot 仍为 Enabled。

修复后 `dkms status` 仍可能显示：

```text
WARNING! Diff between built and installed module!
```

本次该警告未阻断功能：`nvidia-smi` 正常、`nvidia*` 模块已加载，实际使用的是 Ubuntu 预编译签名模块。未来仍应结合当前包版本、已加载模块和 `nvidia-smi` 综合判断，不能仅凭这一行认定驱动再次损坏。

## 8. 清理无效 OEM 软件源并检查系统

### 8.1 禁用 24.04 上不存在的 OEM 组件源

`apt update` 仍访问：

```text
oem.archive.canonical.com ... jiayi
dell.archive.canonical.com ... somerville-aron
```

这些组件在 Noble 仓库中不存在，并伴随 `Signed-By` 警告。真正生效的是两个 `.sources` 文件；`.distUpgrade` 和 `.save` 只是升级备份。

采用可恢复方式禁用：

```bash
sudo mv \
  /etc/apt/sources.list.d/oem-jiayi-meta.sources \
  /etc/apt/sources.list.d/oem-jiayi-meta.sources.disabled

sudo mv \
  /etc/apt/sources.list.d/oem-somerville-aron-meta.sources \
  /etc/apt/sources.list.d/oem-somerville-aron-meta.sources.disabled

sudo apt update
```

结果：OEM 组件和 `Signed-By` 警告消失，Ubuntu Noble 主源及 Google Chrome 源正常。

### 8.2 不执行批量自动清理

只做模拟：

```bash
sudo apt -s autoremove --purge
```

模拟将删除约 317 个包，其中不仅有旧 NVIDIA 535、Python 3.10 和旧 OEM 内核，也包含 ACPI/电源管理、输入法、字体、gedit、Samba、桌面与回退组件。因此没有执行真正的：

```bash
sudo apt autoremove --purge
```

### 8.3 包和服务体检

清除非关键的旧失败记录：

```bash
sudo systemctl reset-failed casper-md5check.service
systemctl --failed
```

最终为：

```text
0 loaded units listed
```

正确的包状态检查是：

```bash
sudo apt-get check
sudo dpkg --audit
```

两者均没有错误。曾输入的 `sudo apt check` 在当前版本返回“无效的操作”，这不表示包系统损坏。

`lsb_release -ds` 显示：

```text
No LSB modules are available.
Ubuntu 24.04.4 LTS
```

第一行只是未安装旧 LSB 兼容模块的提示，可以忽略。

## 9. 安装 ChatGPT、系统体检与补装 Firmware Updater

本节按当晚实际顺序记录：先安装 ChatGPT，随后做完整系统体检，最后补装发行版升级期间因 DNS 问题而失败的 Firmware Updater。

### 9.1 安装 ChatGPT Linux 桌面应用

下载文件：

```text
~/下载/chatgpt_amd64.deb
```

文件约 371 MB，版本 `26.818.41705`。

右键“打开方式 → 软件安装”时窗口约一秒后闪退，因此改用终端：

```bash
cd ~/下载
ls -lh chatgpt*.deb
sudo apt install ./chatgpt_amd64.deb
```

安装时同时加入：

```text
git
git-man
liberror-perl
```

关键成功输出：

```text
正在设置 chatgpt (26.818.41705) ...
```

最后的 `_apt` 下载目录权限提示不影响安装。没有 `E:` 错误，无需重启。

验证：

```bash
dpkg -s chatgpt | grep -E '^(Status|Version):'
```

预期：

```text
Status: install ok installed
Version: 26.818.41705
```

启动命令：

```bash
chatgpt
```

会话只确认安装成功，没有明确记录应用启动和登录最终成功。

### 9.2 完整系统体检

实际执行：

```bash
lsb_release -ds
uname -r
mokutil --sb-state
nvidia-smi
sudo dpkg --audit
sudo apt update
apt list --upgradable
systemctl --failed
```

关键结果：

```text
Ubuntu 24.04.4 LTS
6.8.0-138-generic
SecureBoot enabled
NVIDIA Driver Version: 580.173.02
GPU: NVIDIA RTX 2000 Ada
可升级软件包：0
失败的 systemd 服务：0
dpkg --audit：无输出
```

APT 源已指向 Noble；Ubuntu 华为云主源、updates、security、backports 以及 Google Chrome 源均可正常刷新。

### 9.3 补装升级期间失败的 Firmware Updater

第一次检查：

```bash
snap list firmware-updater
```

结果：

```text
错误：未找到匹配的 snap
```

随后安装并复查：

```bash
sudo snap install firmware-updater
snap list firmware-updater
sudo apt-get check
```

结果：

```text
firmware-updater  0+git.5645b80  226  latest/stable  canonical✓
```

`apt-get check` 正常完成。`firmware-updater` 是 Snap，ChatGPT 的包名是 `chatgpt` 且通过 Debian 包安装；本次未观察到二者冲突。

## 10. Codex 登录 403 与独立发现的 `libcurl` 冲突

### 10.1 初始错误

Codex 登录已跳转到本地回调：

```text
localhost:1455
```

但页面显示：

```text
403 Country, region, or territory not supported
```

本地回调端口本身正常。排查重点转向：浏览器、桌面应用和终端是否走相同网络出口。

### 10.2 安装 `curl` 后出现符号错误

系统当时没有 `curl`：

```bash
sudo apt install curl
```

安装后却无法启动：

```text
curl: symbol lookup error:
curl: undefined symbol: curl_global_trace, version CURL_OPENSSL_4
```

先用 `wget` 检查出口，再看动态库：

```bash
wget -qO- https://ipinfo.io/country
echo

ldd /usr/bin/curl | grep libcurl
```

关键结果：

```text
SG
libcurl.so.4 => /usr/lib/dcaenabler/libcurl.so.4
```

终端出口已经是新加坡；真正问题是 Ubuntu 的 `curl` 错误加载了 Dell DCA Enabler 自带的旧 `libcurl`。

确认包：

```bash
dpkg -l | grep -i dca
```

相关包：

```text
dca-enabler           1.6.1-8
dca-enabler-packages  1.6.1-8
```

`libdca0` 是 DTS 音频解码库，与 Dell DCA 无关，保留。

确认电脑不由公司通过 Dell Wyse 集中管理后，执行：

```bash
sudo apt remove dca-enabler dca-enabler-packages
sudo ldconfig
sudo systemctl daemon-reload
```

验证：

```bash
ldd /usr/bin/curl | grep libcurl
curl --version
curl -sS -o /dev/null -w '%{http_code}\n' \
  https://api.openai.com/v1/models
```

结果：

```text
libcurl.so.4 => /lib/x86_64-linux-gnu/libcurl.so.4
curl 8.5.0 ... security patched: 8.5.0-2ubuntu10.12
401
```

无凭据访问 `/v1/models` 返回 `401` 是预期结果。它表明这次请求已到达 OpenAI API，且没有返回同样的地区 403；它不能证明登录令牌交换路径、账户或其他端点均可用。

此后要求关闭旧的 `localhost:1455` 页面、完全退出应用并发起一次全新登录；但三个会话中没有明确记录“最终登录成功”，因此不能把它写成已验证结论。

## 11. 为桌面应用配置 Clash Verge TUN

终端已经得到 `SG + 401`，但桌面应用仍可能没有走同一路径，因此为 Clash Verge 配置 TUN。此处的 Mihomo 版本截图显示为 `v1.19.29`。

> [!IMPORTANT]
> 此配置只用于本人位于服务支持地区或使用获授权网络时统一本机应用路径，不应被理解为绕过地区限制的方法。反复从不支持地区访问也可能带来账户风险。

采用的设置：

| 项目 | 值 |
|---|---|
| 代理模式 | 排查阶段临时全局 |
| TUN 栈 | `System` |
| 虚拟网卡名 | `Mihomo` |
| 自动设置全局路由 | 开启 |
| 自动选择出口接口 | 开启 |
| 自动重定向 | 关闭 |
| 严格路由 | 关闭 |
| DNS 劫持 | `any:53` |
| MTU | `1500` |
| 排除网段 | 空 |

同时开启：

- System Proxy
- TUN Mode

再次验证：

```bash
curl -s https://ipinfo.io/country
echo
curl -sS -o /dev/null -w '%{http_code}\n' \
  https://api.openai.com/v1/models
```

预期仍为：

```text
SG
401
```

还应在 Clash Verge 的“连接”页面确认 OpenAI/ChatGPT 流量经过代理，而不是 `DIRECT`。

若启用 TUN 后完全断网，紧急回退顺序是：先关闭 `TUN Mode`，再关闭 `System Proxy`；不要在尚未恢复基础网络时继续修改防火墙。

## 12. 固件更新与 DHCP 配置第二次覆盖

### 12.1 固件更新顺序

Firmware Updater 显示三项：

1. System Firmware `1.22.0 → 1.25.1`
2. UEFI CA `2011 → 2023`
3. UEFI dbx `20241101 → 20260402`

采取的顺序：

1. 接原装电源，电量高于 50%，保存工作。
2. 更新 System Firmware，重启。
3. 更新 UEFI CA，重启。
4. 更新 UEFI dbx，重启。
5. 更新过程中不合盖、不强制关机、不拔电源。

最终：

- BIOS/System Firmware：`1.25.1`
- UEFI CA：2023
- UEFI dbx：`20260402`
- Secure Boot：Enabled
- `fwupdmgr get-updates`：`No updates available`
- 固件历史报告上传成功；之后选择不再自动上传报告

### 12.2 BIOS 重启后再次真实断网

更新 System Firmware 并重启后，Wi-Fi 又显示已连接但不能联网。

发现此前禁用的文件旁边又出现了新的：

```text
/etc/NetworkManager/conf.d/dhcp-client.conf
```

内容仍为：

```ini
[main]
dhcp=dhclient
```

该文件创建于前一晚约 22:49:53。BIOS 固件不会在 Linux `/etc` 中创建它；更合理的解释是重启让 NetworkManager 重新读取了此前已经存在的覆盖配置。

调查：

```bash
dpkg-query -S /etc/NetworkManager/conf.d/dhcp-client.conf
dpkg -l | grep -E 'dca-enabler|dca-enabler-packages'
stat /etc/NetworkManager/conf.d/dhcp-client.conf

sudo grep -Rns --include='*.conf' \
  '^[[:space:]]*dhcp[[:space:]]*=' \
  /etc/NetworkManager /run/NetworkManager \
  /usr/lib/NetworkManager /var/lib/NetworkManager \
  2>/dev/null
```

结果：

- 文件不属于任何已登记的软件包。
- 所有者为 root。
- 权限异常宽松，为 `666`。
- 两个 DCA 包当时处于 `rc`：程序已移除，但包配置记录尚未清除。

**已证实的直接原因：** `dhcp-client.conf` 覆盖 `dhcp=internal`，导致外部 `dhclient` 再次生效。

**未证实的来源判断：** DCA 使用 DHCP/DNS 发现且有残留，因此高度可疑；但没有日志或脚本证据证明文件一定由 DCA 创建。不能把这一点写成定论。

### 12.3 使 `internal` 稳定生效并经重启验证

```bash
sudo mv \
  /etc/NetworkManager/conf.d/dhcp-client.conf \
  /etc/NetworkManager/conf.d/dhcp-client.conf.saved-20260822

sudo mv \
  /etc/NetworkManager/conf.d/99-dhcp-internal.conf \
  /etc/NetworkManager/conf.d/zz-dhcp-internal.conf

sudo chmod 644 \
  /etc/NetworkManager/conf.d/zz-dhcp-internal.conf

sudo systemctl restart NetworkManager
```

验证：

```bash
sudo /usr/sbin/NetworkManager --print-config | grep -i '^dhcp='
ip route
ping -c 3 223.5.5.5
```

结果：

```text
dhcp=internal
3 packets transmitted, 3 received, 0% packet loss
```

随后清除 DCA 包配置，但保留诊断文件：

```bash
sudo chmod 600 \
  /etc/NetworkManager/conf.d/dhcp-client.conf.saved-20260822 \
  /etc/NetworkManager/conf.d/dhcp-client.conf.disabled

sudo dpkg --purge dca-enabler dca-enabler-packages
sudo systemctl daemon-reload
```

验证：

```bash
dpkg -l | grep -E 'dca-enabler|dca-enabler-packages'
ldd /usr/bin/curl | grep libcurl
sudo /usr/sbin/NetworkManager --print-config | grep -i '^dhcp='
```

结果：

- DCA 包查询无输出。
- `libcurl` 仍指向 Ubuntu 系统库。
- `dhcp=internal` 保持。
- `/var/log/dcae` 和 `/etc/dcae/config` 因目录非空而保留。
- 两个 DHCP 备份文件有意保留。

重启后再次检查：

```bash
sudo /usr/sbin/NetworkManager --print-config | grep -i '^dhcp='

test -e /etc/NetworkManager/conf.d/dhcp-client.conf \
  && echo "异常：文件又出现了" \
  || echo "正常：没有重新生成"
```

最终：

```text
dhcp=internal
正常：没有重新生成
```

## 13. 第二种 Wi-Fi 问号：Mihomo Fake-IP 造成误判

固件和 DHCP 已正常后，Wi-Fi 图标再次出现问号，但这一次浏览器能上网。

### 13.1 诊断结果

```bash
nmcli networking connectivity check
nmcli device status
ip -4 -br addr show dev wlp0s20f3
ip route
ping -c 3 223.5.5.5
getent hosts www.baidu.com

curl --noproxy '*' -i --max-time 10 \
  http://connectivity-check.ubuntu.com/

sudo /usr/sbin/NetworkManager --print-config \
  | grep -A6 '^\[connectivity\]'
```

关键现象：

```text
nmcli networking connectivity check -> limited
wlp0s20f3 -> 已连接
Mihomo -> tun，连接（外部）
ping -> 0% packet loss
HTTP/1.1 204 No Content
x-networkmanager-status: online
物理 Wi-Fi 默认路由 metric -> 20600
```

`metric 20600` 与联网检测失败给物理连接附加约 `+20000` 惩罚的现象一致。

域名解析却返回 Mihomo Fake-IP：

```text
www.baidu.com -> fdfe:dcba:9876::f
connectivity-check.ubuntu.com -> 198.18.0.8 / fdfe:dcba:9876::*
```

绑定物理 Wi-Fi 请求时超时：

```bash
sudo curl -4 --interface wlp0s20f3 --noproxy '*' \
  -i --max-time 10 \
  http://connectivity-check.ubuntu.com/
```

```text
curl: (28) Connection timed out after 10002 milliseconds
```

结论：

- DHCP、IPv4、默认路由和互联网均正常。
- 现象与 NetworkManager 针对物理 Wi-Fi 执行联网检测一致；手工绑定物理接口时，该请求无法使用 TUN 内部的 Fake-IP 映射。
- 因此 GNOME 显示 `limited` 和问号，但实际网络没有中断。

### 13.2 Clash Verge Merge 修复

在“全局扩展覆写配置（Merge）”中保留原配置并追加：

```yaml
# Profile Enhancement Merge Template for Clash Verge

profile:
  store-selected: true

dns:
  fake-ip-filter:
    - connectivity-check.ubuntu.com
```

操作：

1. 打开“订阅”。
2. 进入“全局扩展覆写配置（Merge）”，不是 Script，也不是只读“当前配置”。
3. 保存后点击一次订阅卡片重新应用。
4. 关闭虚拟网卡模式，等待约 3 秒，再重新开启。
5. 在“当前配置”确认出现 `fake-ip-filter`。

最终验证：

```bash
sudo resolvectl flush-caches
getent ahosts connectivity-check.ubuntu.com

sudo curl -4 --interface wlp0s20f3 --noproxy '*' \
  -i --max-time 10 \
  http://connectivity-check.ubuntu.com/

nmcli networking connectivity check
ip route
```

结果：

- 检测域名恢复为真实 Ubuntu IPv4/IPv6，不再是 `198.18.*` 或 `fdfe:dcba:9876::*`。
- 绑定物理 Wi-Fi 的请求返回 `HTTP/1.1 204 No Content`。
- `nmcli networking connectivity check` 返回：

```text
full
```

一次最终 `ip route` 输出只显示：

```text
198.18.0.0/30 dev Mihomo proto kernel scope link src 198.18.0.1
```

这是 Mihomo TUN 使用策略路由时可能出现的正常表现；当浏览器、HTTP 204 和 `nmcli ... full` 均正常时，不应仅因传统主路由表里看不到普通默认路由就继续“修复”。

Merge 规则只让 Ubuntu 联网检测域名获取真实 IP，不影响其他网站继续通过代理，通常也不会被订阅更新覆盖。

## 14. 旧项目在 Ubuntu 24.04 上的恢复计划

项目环境尚未真正重建或运行验证。原 `environment.yml` 记录的核心版本为：

```text
Python 3.9.7
PyTorch 1.10.1
CUDA Runtime 11.3.1
torchvision 0.11.2
```

会话一处称项目为 `DRUS`，环境名/路径则为 `ddrm`；两者关系待核实。已确定的恢复策略：

- 不直接使用 Ubuntu 24.04 自带的 Python 3.12。
- 用 Miniforge/Conda 创建隔离的 Python 3.9 环境。
- 不需要为了旧项目在系统级安装 CUDA 11.3；优先使用 Conda 环境中的 CUDA Runtime。
- 不要用最新版 PyTorch 覆盖项目原来的 1.10.1。
- 从 `environment.yml` 删除机器绑定路径，例如：

```yaml
prefix: /home/OLD_USER/miniconda3/envs/ddrm
```

- 让 PyCharm 指向恢复后的 `ddrm` Conda 解释器。
- 需要实际测试 RTX 2000 Ada 与旧 PyTorch/CUDA 组合；`nvidia-smi` 正常不等于项目环境已经兼容。

## 15. 最终验证清单

### 已明确验证

- [x] Ubuntu 24.04.4 LTS 正常启动
- [x] 内核 `6.8.0-138-generic`
- [x] Secure Boot Enabled
- [x] NVIDIA RTX 2000 Ada 正常
- [x] NVIDIA 驱动 `580.173.02`
- [x] NetworkManager 使用 `dhcp=internal`
- [x] IPv4、默认路由、DNS、浏览器均正常
- [x] `nmcli networking connectivity check` 为 `full`
- [x] Clash Verge `2.5.2` 可用
- [x] `curl` 使用 Ubuntu 系统 `libcurl`
- [x] OpenAI API 无凭据访问返回预期 `401`
- [x] OEM Noble 无效源已禁用
- [x] `sudo apt-get check` 无错误
- [x] `sudo dpkg --audit` 无输出
- [x] `systemctl --failed` 为 0
- [x] BIOS/System Firmware `1.25.1`
- [x] UEFI CA 2023
- [x] UEFI dbx `20260402`
- [x] `fwupdmgr get-updates` 显示无可用固件更新
- [x] ChatGPT `26.818.41705` 安装成功

### 尚未明确验证

- [ ] ChatGPT/Codex 最终登录成功
- [ ] PyCharm、Conda 和 GitHub 项目环境恢复
- [ ] 项目训练/推理实际运行
- [ ] 摄像头
- [ ] 指纹
- [ ] 蓝牙
- [ ] 音频输入/输出
- [ ] 睡眠与唤醒
- [ ] 外接显示器和扩展坞
- [ ] 对 317 个 `autoremove` 候选的逐项审查

## 16. 有意保留的回退点

以下文件/目录当时没有删除：

```text
/etc/NetworkManager/conf.d/dhcp-client.conf.disabled
/etc/NetworkManager/conf.d/dhcp-client.conf.saved-20260822
/etc/apt/sources.list.d/oem-jiayi-meta.sources.disabled
/etc/apt/sources.list.d/oem-somerville-aron-meta.sources.disabled
/etc/apt/sources.list.d/oem-jiayi-meta.list.distUpgrade
/etc/apt/sources.list.d/oem-jiayi-meta.list.save
/etc/apt/sources.list.d/oem-somerville-aron-meta.list.distUpgrade
/etc/apt/sources.list.d/oem-somerville-aron-meta.list.save
/var/log/dcae
/etc/dcae/config
```

另外还保留了升级后的旧内核和大量旧组件。

在完成项目、硬件和至少数日稳定性验证前，不执行：

```bash
sudo apt autoremove
sudo apt autoremove --purge
```

## 17. 以后复查时的最小命令集

### 系统与固件

```bash
grep PRETTY_NAME /etc/os-release
uname -r
mokutil --sb-state
cat /sys/class/dmi/id/bios_version
fwupdmgr get-updates
```

### NVIDIA

```bash
nvidia-smi
lsmod | grep -E '^nvidia'
dkms status
```

`dkms status` 可能仍显示已知的 `Diff between built and installed module` 警告；应结合 `nvidia-smi`、实际加载模块和当前包版本判断。

### 网络

```bash
sudo /usr/sbin/NetworkManager --print-config | grep -i '^dhcp='
nmcli networking connectivity check
ip -4 -br addr show dev wlp0s20f3
ip route
ping -c 3 223.5.5.5
getent hosts www.baidu.com
getent ahosts connectivity-check.ubuntu.com

sudo curl -4 --interface wlp0s20f3 --noproxy '*' \
  -i --max-time 10 \
  http://connectivity-check.ubuntu.com/
```

如果图标有问号，先区分两类情况：

1. **浏览器也不能联网、无 IPv4/默认路由**：检查是否又出现 `dhcp-client.conf`，以及 `dhcp` 是否被改回 `dhclient`。
2. **浏览器能联网、`nmcli` 显示 `limited`**：检查 `connectivity-check.ubuntu.com` 是否又被解析成 Mihomo Fake-IP。

### 包系统

```bash
sudo apt update
sudo apt-get check
sudo dpkg --audit
systemctl --failed
```

### ChatGPT/Codex 与网络路径

```bash
dpkg -s chatgpt | grep -E '^(Status|Version):'
ldd /usr/bin/curl | grep libcurl
curl -s https://ipinfo.io/country
echo
curl -sS -o /dev/null -w '%{http_code}\n' \
  https://api.openai.com/v1/models
```

无 API 凭据时，最后一条返回 `401` 表示该次请求已到达对应 API；它不代表登录令牌交换路径、账户或其他端点均可用，也不代表 API Key 有效。

## 18. 关键经验

1. Dell OEM 系统可能通过 `manage-distro-upgrade` 和 `Prompt=never` 阻止发行版升级，不能只改一个配置文件而不检查 `dpkg-divert`。
2. Secure Boot 下 NVIDIA 失败不一定是签名问题。本次模块有 Canonical 签名，真正原因是 Jammy 模块与 Noble 内核 ABI 混装。
3. “Wi-Fi 已连接”只表示链路或连接配置已激活，不代表获得了 IPv4 和默认路由。
4. NetworkManager 配置最终以合并后的 `--print-config` 为准；文件名排序可能让后一个 `.conf` 覆盖前一个。
5. 固件更新后的重启可能只是暴露已有配置问题，不应把 Linux `/etc` 中的新文件直接归因于 BIOS。
6. Wi-Fi 问号分为“真实断网”和“联网检测误判”两类，必须用 IPv4、路由、ping、HTTP 204 和 `nmcli` 区分。
7. 桌面应用、浏览器和终端未必走同一代理路径；TUN 可以统一路径，但也会引入 Fake-IP 与系统联网检测的交互。
8. 第三方 OEM 程序携带的动态库可能覆盖系统库。本次 Dell DCA Enabler 让 `/usr/bin/curl` 加载了错误的 `libcurl`。
9. 升级后看到大批 `autoremove` 候选时不要急着清理；模拟列表中可能包含驱动、旧内核、桌面组件和回退工具。
10. `nvidia-smi` 的 CUDA 版本是驱动兼容上限，不等于项目所需 CUDA Runtime 或 Toolkit 已安装。

## 19. 结论边界

以下结论有直接命令输出支持：

- Ubuntu、内核、Secure Boot、NVIDIA、APT/dpkg、systemd、固件和网络最终状态正常。
- `dhcp-client.conf` 覆盖 `dhcp=internal` 是两次真实断网的直接原因。
- Mihomo Fake-IP 是第二次“能上网但图标有问号”的原因。
- Dell DCA Enabler 的旧 `libcurl` 是 `curl` 符号错误的原因。

以下只应保留为推断或待办：

- `dhcp-client.conf` 很可能与 DCA 或其残留脚本有关，但没有直接证据证明文件创建者。
- ChatGPT/Codex 应用已安装且网络路径已修复，但没有最终登录成功的明确记录。
- GitHub 项目理论上可通过隔离的 Conda 环境迁移，但尚未实机验证。
- Dell OEM 专属硬件功能尚未全部完成回归测试。

---

记录整理于 2026-08-23。后续如完成项目环境恢复、硬件回归测试或旧包清理，应继续在本文末尾追加新的日期、命令和验证结果。
