---
title: 使用 MacMini UTM 虚拟机，启动 Home Assistant OS
date: 2025-11-29
draft: true
categories:
  - Home Assistant
description: '在 MacMini 上通过 UTM 虚拟机安装和配置 Home Assistant OS，实现智能家居控制中心的搭建，包括网络代理配置、HACS 插件安装以及小米、美的设备接入。'
---

## 前置准备

### 梯子和 TUN 模式

为了能够顺利访问 Home Assistant 的插件商店和下载各种集成，建议配置支持 TUN 模式的代理工具。

**为什么需要 TUN 模式？**

- 普通代理模式只能代理应用层流量
- TUN 模式可以接管系统级别的网络流量，让虚拟机内的所有网络请求都通过代理
- 确保 Home Assistant OS 能够访问 GitHub、官方插件仓库等资源

**推荐的代理工具：**
- **Clash Verge**（推荐，支持 Apple Silicon）
- **ClashX Pro**
- **Surge**

**配置步骤：**
1. 安装代理工具
2. 在设置中开启 TUN 模式
3. 确保代理规则包含虚拟机的网络流量
4. 测试虚拟机内是否能正常访问外网

### 安装 UTM

UTM 是 macOS 上的开源虚拟机软件，特别适合 Apple Silicon 芯片。

**下载方式：**

方式一：官网下载
```bash
# 访问官网下载
https://mac.getutm.app/
```

方式二：通过 Homebrew 安装（推荐）
```bash
brew install --cask utm
```

**UTM 的优势：**
- 原生支持 Apple Silicon（M1/M2/M3 等）
- 开源免费
- 性能优秀
- 支持 QCOW2 等多种镜像格式

### 下载 Home Assistant OS 镜像

**官方下载地址：**
```
https://www.home-assistant.io/installation/macos
```

**选择正确的镜像格式：**
- UTM/KVM 使用：选择 **QCOW2** 格式
- 文件名示例：`haos_ova-13.2.qcow2.xz`
- 下载完成后解压得到 `.qcow2` 文件

**镜像版本建议：**
- 生产环境：选择最新稳定版（Stable）
- 尝鲜体验：可以选择测试版（Beta）

## 安装配置

### 在 UTM 中创建虚拟机

1. **打开 UTM，创建新虚拟机**
   - 点击"+"按钮 → 选择"虚拟化"（Apple Silicon）或"模拟"（Intel）
   - 操作系统类型选择"Linux"

2. **配置虚拟机资源**
   - **内存：** 最低 2GB，推荐 4GB
   - **CPU 核心：** 最低 2 个，推荐 4 个
   - 注：资源分配根据 MacMini 的实际配置调整

3. **导入系统镜像**
   - 在"启动"设置中，选择"导入启动镜像"
   - 选择下载的 `.qcow2` 文件

4. **配置完成**
   - 检查所有设置无误后保存
   - 虚拟机命名建议：HomeAssistant

### 磁盘分配

**默认配置：**
- 官方镜像自带虚拟磁盘，建议扩展到 **64GB** 或更大！！！
- 对于基础使用场景已经足够

**扩展方式：**
1. 在 UTM 的虚拟机设置中
2. 进入"驱动器"选项
3. 添加新的虚拟磁盘或调整现有磁盘大小
4. 启动后在 Home Assistant 中配置挂载

**数据备份建议：**
- 定期备份配置文件
- 重要数据可以映射到宿主机目录
- 使用 Home Assistant 的快照功能

### 网络配置

**UTM 网络模式选择：**

1. **共享网络模式（推荐新手）**（目前我使用的方式）
   - 优点：配置简单，开箱即用
   - 缺点：虚拟机 IP 会变化，需要通过 `homeassistant.local` 访问
   - 适用场景：个人学习、测试使用

2. **桥接模式（推荐生产环境）**
   - 优点：虚拟机获得局域网独立 IP，便于其他设备访问
   - 缺点：需要路由器支持，配置稍复杂
   - 适用场景：长期使用、需要局域网内多设备访问

**配置步骤（桥接模式）：**
1. 在 UTM 虚拟机设置中选择"网络"
2. 网络模式选择"桥接模式"
3. 选择 MacMini 的网络接口（有线网卡或 Wi-Fi）
4. 保存设置并启动虚拟机

**端口说明：**
- `8123`：Home Assistant Web 界面
- `8300`：Home Assistant 内部通讯
- `51827`：HomeKit 配对

## 首次启动和初始化

### 启动虚拟机

1. 在 UTM 中启动创建好的虚拟机
2. 等待 3-5 分钟，让 Home Assistant OS 完成初始化
3. 观察终端输出，看到 "Welcome to Home Assistant" 表示启动成功

### 访问 Home Assistant

**访问方式：**

方式一：通过域名访问（推荐）
```
http://homeassistant.local:8123
```

方式二：通过 IP 访问
```
http://虚拟机IP:8123
```

如何查看虚拟机 IP：
- 在 UTM 终端中输入 `ha network info`
- 或通过路由器管理界面查看

### 初始化设置

1. **创建管理员账户**
   - 设置用户名和密码
   - 建议使用强密码

2. **设置位置信息**
   - 填写家庭地址（用于天气、日出日落等自动化）
   - 选择时区：`Asia/Shanghai`

3. **分享设置**
   - 选择是否分享匿名统计信息
   - 可根据个人喜好选择

4. **完成初始化**
   - 进入主界面，开始配置智能家居

**Home Assistant 主界面示例：**

<a href="/images/homeAssistant/ha-overview.png" data-lightbox="ha-overview" data-title="Home Assistant 主界面">
  <img src="/images/homeAssistant/ha-overview.png" alt="Home Assistant 主界面">
</a>

## 装机必备 AddOn

AddOn 是 Home Assistant OS 的核心功能扩展，以下是强烈推荐安装的插件。

### 安装方式

进入：`设置` → `加载项` → `加载项商店`

**设置主界面：**

<a href="/images/homeAssistant/ha-settings.png" data-lightbox="ha-settings" data-title="Home Assistant 设置界面">
  <img src="/images/homeAssistant/ha-settings.png" alt="Home Assistant 设置界面">
</a>

**已安装的加载项列表：**

<a href="/images/homeAssistant/ha-addons.png" data-lightbox="ha-addons" data-title="Home Assistant 加载项">
  <img src="/images/homeAssistant/ha-addons.png" alt="Home Assistant 加载项">
</a>

### 核心 AddOn 推荐

**1. File Editor（文件编辑器）**
- **用途：** 在网页端直接编辑配置文件
- **推荐理由：** 无需 SSH 即可修改 configuration.yaml 等文件
- **安装后：** 在侧边栏找到 "File Editor" 即可使用

**2. Terminal & SSH**
- **用途：** 通过 SSH 或网页终端访问系统
- **推荐理由：** 高级配置、调试必备
- **配置要点：**
  - 设置 SSH 端口（默认 22）
  - 配置密码或密钥认证
  - 启动后可通过 `ssh root@homeassistant.local` 访问

**3. Samba Share（网络共享）**
- **用途：** 将 Home Assistant 配置目录共享到网络
- **推荐理由：** 在电脑上直接编辑配置文件，支持 VSCode
- **配置步骤：**
  - 安装后启动服务
  - macOS 访问：`Finder` → `前往` → `连接服务器` → `smb://homeassistant.local`

**4. Studio Code Server（可选）**
- **用途：** 完整的 VSCode 网页版环境
- **推荐理由：** 强大的代码编辑功能，支持插件、Git 等
- **注意：** 占用资源较多，配置较低的机器可以跳过

**5. Duck DNS（外网访问）**
- **用途：** 配置动态域名，实现外网访问
- **推荐理由：** 出门在外也能控制家中设备
- **配合使用：** Let's Encrypt（自动申请 HTTPS 证书）

## HACS 安装和配置

HACS（Home Assistant Community Store）是社区插件商店，提供海量的自定义集成和前端组件。

### 什么是 HACS？

- Home Assistant 的"非官方应用商店"
- 包含数千个社区开发的集成、主题、卡片等
- 小米、美的等国内设备的集成大多在 HACS 中

### 安装 HACS

**方式一：通过 Terminal SSH 安装（推荐）**

1. 打开 Terminal & SSH AddOn
2. 执行安装脚本：
```bash
wget -O - https://get.hacs.xyz | bash -
```

3. 等待安装完成
4. 重启 Home Assistant：`设置` → `系统` → `重启`

**方式二：手动下载安装**

如果网络问题无法执行脚本：
1. 从 GitHub 下载 HACS 最新版本
2. 解压到 `/config/custom_components/hacs/` 目录
3. 重启 Home Assistant

### 配置 HACS

1. **添加 HACS 集成**
   - `设置` → `设备与服务` → `添加集成`
   - 搜索"HACS"并添加

2. **GitHub 授权**
   - 点击配置，会跳转到 GitHub
   - 使用 GitHub 账号登录并授权（需要提前注册 GitHub 账号）
   - 复制授权码，粘贴回 Home Assistant

3. **选择类别**
   - 集成（Integration）：必选
   - 主题（Theme）：可选
   - 前端（Frontend）：可选

4. **完成配置**
   - 配置完成后，侧边栏会出现"HACS"选项

### HACS 使用

- 点击侧边栏"HACS"
- 选择"集成"类别
- 搜索并安装需要的集成
- 安装后重启 Home Assistant

**HACS 社区商店界面：**

<a href="/images/homeAssistant/ha-hacs.png" data-lightbox="ha-hacs" data-title="HACS 社区商店">
  <img src="/images/homeAssistant/ha-hacs.png" alt="HACS 社区商店">
</a>

## 接入设备

通过 Home Assistant 的集成功能，可以轻松接入各种智能家居设备。

**设备与服务界面：**

<a href="/images/homeAssistant/ha-integrations.png" data-lightbox="ha-integrations" data-title="Home Assistant 设备与服务">
  <img src="/images/homeAssistant/ha-integrations.png" alt="Home Assistant 设备与服务">
</a>

### 接入小米设备

推荐使用 **Xiaomi Miot Auto** 集成，支持小米米家全系列设备。

**安装步骤：**

1. **通过 HACS 安装集成**
   - 打开 HACS → 集成
   - 搜索"Xiaomi Miot Auto"
   - 点击下载并重启 Home Assistant

2. **添加集成**
   - `设置` → `设备与服务` → `添加集成`
   - 搜索"Xiaomi Miot Auto"

3. **登录小米账号（推荐）**
   - 选择"通过小米账号登录"
   - 输入米家 App 使用的小米账号和密码
   - 自动发现并添加所有米家设备

4. **手动添加设备（可选）**
   - 如果不想提供账号密码
   - 可以通过设备 IP + Token 的方式添加
   - Token 获取方式：
     - 使用小米米家 App 抓包
     - 使用第三方工具如"小米米家设备 Token 提取器"

**支持的设备类型：**
- 智能灯泡、灯带、吸顶灯
- 智能插座、开关
- 扫地机器人、空气净化器
- 传感器（温湿度、门窗、人体等）
- 小爱音箱
- 其他米家生态链设备

### 接入美的设备

推荐使用 **Midea AC LAN** 集成，支持美的空调、除湿机等设备。

**安装步骤：**

1. **通过 HACS 安装集成**
   - 打开 HACS → 集成
   - 搜索"Midea AC LAN"
   - 点击下载并重启 Home Assistant

2. **添加集成**
   - `设置` → `设备与服务` → `添加集成`
   - 搜索"Midea AC LAN"

3. **配置方式**

**方式一：自动发现**
   - 选择"自动发现设备"
   - 等待扫描局域网内的美的设备
   - 选择要添加的设备

**方式二：手动添加**
   - 输入设备 IP 地址
   - 输入设备 ID（通过美的官方 App 查看）
   - 输入 Token（首次配置自动获取）

4. **完成配置**
   - 设备添加成功后，可以在主界面看到设备卡片
   - 支持温度调节、模式切换、风速调节等功能

**支持的设备类型：**
- 美的空调（挂机、柜机、中央空调）
- 美的除湿机
- 美的新风系统
- 其他美的智能家电

**注意事项：**
- 确保美的设备已连接到与 Home Assistant 相同的局域网
- 部分设备需要在美的官方 App 中开启"局域网控制"功能
- 如果无法发现设备，检查路由器是否开启了 AP 隔离

## 下一步

安装配置完成后，你可以：

1. **创建自动化**：根据时间、状态等触发条件自动控制设备
2. **配置仪表板**：自定义主界面，添加各种卡片和布局
3. **接入更多设备**：空调、电视、窗帘、摄像头等
4. **语音控制**：接入小爱同学、HomeKit、Google Assistant 等
5. **外网访问**：配置 DDNS 和内网穿透，随时随地控制家中设备

Home Assistant 是一个功能强大且高度可定制的智能家居平台，慢慢探索，你会发现无限可能！