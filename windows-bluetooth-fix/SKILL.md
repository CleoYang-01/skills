---
name: Windows蓝牙设备连不上修复
description: 当蓝牙鼠标/键盘等 BLE 设备（如罗技 M240）在两台电脑间来回使用、反复重连后突然连不上，或"添加设备时搜得到但配对失败/连上就断"时，用一键核弹重置修好。覆盖只读诊断、通配删除重复配对记录、清理注册表配对密钥、移除蓝牙适配器级联清除孤儿节点。适用于 Windows 10/11，需要用户在弹窗点"是"提权。
version: 1.0.0
author: 屿
tags: [蓝牙, 蓝牙故障, Windows排障, 罗技, 外设, 配对失败]
---

# Windows 蓝牙设备连不上修复（核弹重置法）

## 何时用这个技能
- 蓝牙鼠标/键盘在公司和家里两台电脑之间来回拿，突然连不上这台电脑。
- 设置里"添加蓝牙设备"时**搜得到设备但配对失败、或连上又断**（B 类）；或根本搜不到（A 类）。
- 用户已在设置里尝试"删除设备"但删不掉。
- **先用手机蓝牙测一下设备本身**：手机能连上 = 硬件没坏，问题在电脑端（本技能适用）；手机也连不上 = 硬件坏了，走保修/换新，本技能不适用。

## 根因（先讲清楚，免得瞎试）
1. **重复配对记录**：BLE 设备用随机可解析私有地址（RPA），每次重连地址都变，Windows 反复写入新记录，旧的没清，连接时"选错/混乱"。
2. **孤儿 HID 节点**：删母设备时，挂在下面的"蓝牙低能耗 GATT 的 HID 设备"子节点没跟着删，残留成孤儿，新配对地址冲突被回滚（B 类主因）。
3. **注册表配对密钥(link key)残留**：旧地址的密钥残留在 `HKLM:\SYSTEM\CurrentControlSet\Services\BTHPORT\Parameters\Keys`，新地址又建一条，两边打架。

## 步骤

### 第 1 步：只读诊断（不动任何设置）
用 PowerShell 跑。**不要用 `Get-PnpDevice -Class HIDClass`**（在某些沙箱环境会直接崩溃进程、连文件都写不出），改用**全量枚举 + `Where-Object` 过滤**：

```powershell
$all = Get-PnpDevice -ErrorAction SilentlyContinue
# 蓝牙适配器
$all | Where-Object { $_.FriendlyName -match 'Bluetooth Adapter|Radio|MediaTek|Intel|Broadcom|Realtek|Qualcomm' } | ForEach-Object { "$($_.FriendlyName) | $($_.Status)" }
# 目标设备（按 FriendlyName 通配，别用固定 ID）
$all | Where-Object { $_.FriendlyName -match 'M240|Logi|Mouse' } | ForEach-Object { "$($_.FriendlyName) | $($_.Status) | $($_.InstanceId)" }
# 注册表配对密钥（需管理员才能读全，普通权限可能为空）
$keys = 'HKLM:\SYSTEM\CurrentControlSet\Services\BTHPORT\Parameters\Keys'
if (Test-Path $keys) { (Get-ChildItem $keys -ErrorAction SilentlyContinue).Count } else { 'no keys path' }
