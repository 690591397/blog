---
title: 使用 Zigbee2MQTT + Sonoff ZBDongle-E 接入 Aqara 单键墙壁开关
date: 2025-12-13
draft: true
categories:
  - Home Assistant
tags:
  - Zigbee
  - Zigbee2MQTT
  - Sonoff
  - Aqara
description: '在 Home Assistant OS（UTM 虚拟机）中使用 Zigbee2MQTT + Sonoff ZBDongle-E 接入 Aqara 单键墙壁开关：配置要点、配对流程、验证与常见问题排查。'
---

## 前置文章（必读）

本文默认你已经把 Home Assistant OS 跑起来，并且网络/基础环境都 OK。前置参考：

- [使用 MacMini UTM 虚拟机，启动 Home Assistant OS]({{< relref "posts/homeAssistant/macos.md" >}})

## 你需要的东西

- **协调器**：Sonoff ZBDongle-E（EFR32 系列，Zigbee2MQTT 通常使用 `ezsp` 适配器）
- **设备**：Aqara 单键墙壁开关（接线款，具体型号不同但流程类似）
- **Home Assistant**：运行在 HA OS（本文场景：UTM 虚拟机）

### 安全提醒

墙壁开关涉及市电接线与断电操作：

- **务必断电**再操作与安装
- **不确定接线**就请电工处理

## 1. 把 ZBDongle-E 接到 Home Assistant（UTM USB 透传）

<a href="/images/homeAssistant/ha-share-usb.png" data-lightbox="ha-share-usb" data-title="UTM USB 共享设置">
  <img src="/images/homeAssistant/ha-share-usb.png" alt="UTM USB 共享设置">
</a>

<a href="/images/homeAssistant/ha-usb-auth.png" data-lightbox="ha-usb-auth" data-title="USB 授权">
  <img src="/images/homeAssistant/ha-usb-auth.png" alt="USB 授权">
</a>

<a href="/images/homeAssistant/ha-sonoff-connect.png" data-lightbox="ha-sonoff-connect" data-title="Sonoff 连接成功">
  <img src="/images/homeAssistant/ha-sonoff-connect.png" alt="Sonoff 连接成功">
</a>

## 2. 安装并配置 MQTT（Mosquitto Broker）

Zigbee2MQTT 依赖 MQTT Broker。这里以 Home Assistant Add-on 的 **Mosquitto broker** 为例（最省事）。

### 2.1 安装 Mosquitto broker Add-on

1. 进入 Home Assistant：**Settings（设置） → Add-ons（加载项）**
2. 打开 Add-on Store（加载项商店），搜索并安装 **Mosquitto broker**
3. 安装后进入该 Add-on：
   - 打开 **Start on boot**（开机自启）
   - 打开 **Watchdog**（看门狗，建议开）
   - 点击 **Start** 启动
4. 切到 **Log**，确认没有持续报错

### 2.2 创建 MQTT 账号（给 Zigbee2MQTT 用）

1. 进入：**Settings（设置） → People（人员） → Users（用户）**
2. 创建一个专门给设备使用的账号（例如 `mqtt`），设置强密码
3. 回到 Mosquitto broker Add-on 的配置页面，把这个账号加入允许列表（不同版本的 UI 文案略有差异，但核心是：让 Mosquitto 允许该用户登录）

### 2.3 确认 Home Assistant 已启用 MQTT 集成

1. 进入：**Settings → Devices & services（设备与服务）**
2. 如果 MQTT 未自动出现：点击 **Add integration（添加集成）**，搜索 **MQTT** 并按向导完成（broker 选择本机/默认）

### 关于 `adapter: ezsp`

ZBDongle-E 常见芯片与固件栈会走 EZSP 协议，因此 Zigbee2MQTT 多数场景使用 `ezsp`。如果你后面遇到启动报错（例如握手失败/不停重连），优先检查：

- **串口路径是否正确**
- **是否被别的集成占用**（比如同时开了 ZHA 占用同一个串口）
- **协调器固件/协议栈与 Z2M 兼容性**

## 3. 安装并配置 Zigbee2MQTT（对接 Mosquitto + 串口）

### 3.1 安装 Zigbee2MQTT Add-on

1. 进入：**Settings → Add-ons → Add-on Store**
2. 搜索并安装 **Zigbee2MQTT**
3. 安装后进入该 Add-on：
   - 打开 **Start on boot**、**Watchdog**
   - 暂时先不要启动（先把连接信息填好）

### 3.2 在 UI 配置 Zigbee2MQTT 连接信息

在 Zigbee2MQTT Add-on 的配置页面里，按界面字段填写（不需要贴配置代码，但需要把关键项填对）：

- **MQTT 连接**：
  - **Server**：使用 Mosquitto Add-on 的地址（常见为 `mqtt://core-mosquitto:1883`）
  - **User / Password**：填写你在「2.2」创建的 MQTT 账号与密码
- **Serial（串口）**：
  - **Port**：优先选择 `/dev/serial/by-id/...`（找不到再用 `/dev/ttyUSB0`）
  - **Adapter**：选择 `ezsp`
- **Frontend**：开启（这样你能直接打开 Zigbee2MQTT Web UI）
- **Permit join**：默认关闭（安全起见，配对时再临时打开）

填完后回到 Zigbee2MQTT Add-on 点击 **Start**，再去 **Log** 确认它能成功连接 MQTT 且串口初始化成功（没有反复重连/报错）。

## 4. 允许入网（Permit join）

配对前先在 Zigbee2MQTT UI 做两件事：

1. 打开 Zigbee2MQTT 前端（Add-on 页面通常会有 “Open web UI”）
2. 点击 **Permit join（允许加入）**，给一个 1-5 分钟的窗口

配对完成后建议立刻关闭 Permit join，避免邻居设备误入网。

## 5. 配对 Aqara 单键墙壁开关

不同 Aqara 墙壁开关的配对动作略有差异，但最常见的是：

- 给开关上电
- **长按开关上的按钮约 5 秒**进入配对（指示灯闪烁或有提示音，视型号而定）
- 在 Zigbee2MQTT 的设备列表里观察到新设备加入

加入成功后建议立刻做三件事：

- **重命名**：改成你家里能看懂的名字（例如 `aqara_wall_switch_entrance`）
- **绑定房间/区域**（Home Assistant 侧更好管理）
- **看 exposes**：确认暴露了哪些能力（`state`、`action`、`power_outage_memory` 等因型号而异）

## 6. 验证是否工作（最小闭环）

在 Zigbee2MQTT 里打开该设备的日志/状态，重点看：

- **linkquality**：越高越稳（低则考虑加路由器设备或调整位置）
- **state / action**：按下开关是否有消息上报

在 Home Assistant 里，你通常会看到：

- 一个或多个实体（entity），例如开关实体（如果支持）
- 或者设备事件（如果是 action 上报型）

这一步的目标是：**按一下开关，HA 里能“看见”变化**。

## 7. 一个最小自动化示例（全程 UI，不写 YAML）

### 方案 A：设备有 `action`（优先）

当设备会上报 `action`（例如 single / double / hold），可以用 MQTT 触发自动化（UI 支持）。

1. 进入：**Settings → Automations & Scenes（自动化与场景） → Create automation（创建自动化）**
2. 选择 **Start with an empty automation（空白）**
3. Trigger（触发器）选择 **MQTT**
4. 填写：
   - **Topic**：`zigbee2mqtt/你的设备名`（设备名就是你在 Zigbee2MQTT 里给它改的 friendly name）
5. Action（动作）选择你想控制的设备，例如：
   - **Light: Toggle**，选择目标灯 `light.living_room`
6. 保存后测试：按开关，看灯是否跟着切换

提示：如果你想区分“单击/双击/长按”，可以在 Zigbee2MQTT 里先观察该设备上报的 action 值，然后在自动化 UI 里用条件（Choose / And）分别匹配不同的 payload。

### 方案 B：设备只暴露 `state`（更通用）

如果你的开关在 HA 里就是一个可用的 `switch.xxx` 实体，直接用实体状态触发更简单、也更直观。

1. 进入：**Settings → Automations & Scenes → Create automation**
2. Trigger 选择 **State**，实体选择你的开关实体（例如 `switch.aqara_wall_switch_entrance`）
3. Action 选择 **Light: Toggle**（或你需要的其他动作），目标选择要控制的灯
4. 保存并测试

## 8. 常见问题排查（高频）

### 7.1 允许入网但设备始终加不进来

- **确认 Permit join 已开启**，且未超时关闭
- **靠近协调器配对**（先贴近成功入网，再移到目标位置）
- **排查是否有其他 Zigbee 网络干扰**（比如你曾用 ZHA/deCONZ 建过网，设备被绑定在旧网里）
- **重置设备**：按型号执行 reset（通常是长按到指示灯特定闪烁节奏）

### 7.2 偶发掉线/延迟很高

- **看 `linkquality`**：偏低就说明链路差
- **加 Zigbee 路由器设备**：比如带中继能力的插座/灯（注意是否支持路由）
- **调整协调器位置**：远离 USB3.0 干扰源、金属机箱、路由器天线等

### 7.3 Zigbee2MQTT 启动报错或不停重连

- **确认串口路径**：优先用 `/dev/serial/by-id/...`
- **确认没有被占用**：ZHA 与 Zigbee2MQTT 不要同时抢同一个串口
- **确认适配器类型**：ZBDongle-E 通常使用 `ezsp`
- **固件兼容性**：如果你近期升级/刷写过固件，优先回看 Zigbee2MQTT 的官方建议版本与你的固件栈是否匹配

## 下一步

当开关稳定在线后，你可以继续做：

- 把开关的 `action`（单击/双击/长按）映射到不同场景
- 在 Zigbee2MQTT 里做分组（group）或绑定（binding）优化延迟
- 增加路由器设备，提升全屋 Zigbee 网的稳定性