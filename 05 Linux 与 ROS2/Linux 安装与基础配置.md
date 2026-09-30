# Linux 安装与基础配置

**Linux 安装与基础配置**

面向 2026 GSing ROS2 Humble 工程的 Ubuntu 22\.04 教程

![image1\.png](../media/05%20Linux%20与%20ROS2/Linux%20安装与基础配置/image1.png)

图 1  Ubuntu 22\.04 安装主流程

|**版本结论  **本项目 README 指定 Ubuntu 22\.04 \+ ROS2 Humble。不要因为官网已有更新版本就直接安装最新版，否则 ROS2 与依赖可能不兼容。|
|---|

# 网页教程与参考资料

- [Ubuntu 22\.04\.5 LTS 官方发布页](https://releases.ubuntu.com/22.04/) — 本项目对应的桌面镜像与校验文件

- [Ubuntu Desktop 官方安装教程](https://ubuntu.com/desktop/docs/en/latest/tutorial/install-ubuntu-desktop/) — 制作启动盘、启动与安装器步骤

- [Ubuntu Desktop 下载页](https://ubuntu.com/download/desktop) — 系统需求与其他安装方式

- [ROS2 Humble 官方文档](https://docs.ros.org/en/humble/) — ROS2 Humble 安装与使用入口

- [Linux 系统下载与安装视频](https://www.bilibili.com/video/BV133411P7Cs/) — 入门视频

# 1\. 选择安装方式

|**方式**|**适用场景**|**限制**|
|---|---|---|
|实体机/双系统|雷达、串口、GPU、比赛部署|操作分区有风险，必须先备份|
|虚拟机|Linux 命令和基础 ROS2 学习|USB、GPU、实时性和网络广播可能受限|
|WSL2|Windows 上学习命令和编译|不适合作为整车传感器与实时部署环境|

# 2\. 安装前准备

- 备份 Windows 文档、代码、密钥和重要配置；确认备份可以读取。

- 准备 8GB 以上 U 盘；制作启动盘会清空 U 盘。

- 为 Ubuntu 预留至少 50GB，开发环境、ROS2、地图和模型建议 100GB 以上。

- 双系统先保存 BitLocker 恢复密钥；不要在不了解磁盘分区时手动删除 EFI 或 Windows 分区。

- 确认 CPU 架构通常为 Intel/AMD 64\-bit（AMD64）。

# 3\. 下载 Ubuntu 22\.04\.5 LTS

- 从 Ubuntu 官方 22\.04 发布页下载 ubuntu\-22\.04\.5\-desktop\-amd64\.iso。

- 同时下载 SHA256SUMS，并校验镜像，避免下载损坏。

\# Windows PowerShell：计算 ISO 的 SHA256；该命令只读取文件。
Get\-FileHash \.\\ubuntu\-22\.04\.5\-desktop\-amd64\.iso \-Algorithm SHA256
\# 将输出值与 Ubuntu 官方 SHA256SUMS 中对应文件的值逐字比较。

# 4\. 制作启动盘与启动

1. 使用 Rufus 或 balenaEtcher 选择 ISO 和正确 U 盘，再开始写入。

2. 重启电脑，在开机 Logo 出现时按厂商启动菜单键（常见 F12、F11、Esc）。

3. 选择带 UEFI 标识的 USB 设备；先进入 Try Ubuntu 可检查 Wi\-Fi、显卡、触控板和磁盘是否识别。

# 5\. 安装器关键选择

4. 语言和键盘可按习惯选择；联网后安装器可下载更新。

5. 新手双系统优先使用“与 Windows 共存”选项；不确定时不要选择“擦除磁盘”。

6. 专用比赛机可以整盘安装，但该操作会删除目标磁盘全部数据。

7. 设置用户名和强密码，记住磁盘加密密码；安装完成后拔出 U 盘并重启。

# 6\. 安装后基础配置

\# 更新软件索引与已安装软件；执行前确保网络正常。
sudo apt update
sudo apt upgrade \-y

\# 安装常用开发工具。
sudo apt install \-y git curl wget build\-essential cmake

\# 查看系统版本，确认应为 Ubuntu 22\.04。
lsb\_release \-a

\# 查看显卡和 USB 设备，为后续雷达/串口排错做准备。
lspci \| grep \-i \-E "vga\|3d"
lsusb

# 7\. 为 ROS2 Humble 做准备

- 确认系统为 Ubuntu 22\.04，再按照 ROS2 Humble 官方安装流程配置软件源。

- 安装完成后每个新终端需要 source /opt/ros/humble/setup\.bash；项目工作空间还要 source install/setup\.bash。

- 不要混用不同 ROS2 发行版的软件源和二进制包。

# 8\. 常见问题

- 看不到 U 盘：重新制作启动盘，换 USB 口，并检查 UEFI 启动菜单。

- 安装器看不到 Windows：先回 Windows 关闭快速启动，检查 BitLocker 与磁盘模式。

- 黑屏或花屏：先使用安全图形模式，安装后再处理 NVIDIA 驱动。

- 没有网络：先用有线网络或手机 USB 共享，安装后再补无线网卡驱动。

