# Windows 蓝牙设备连不上修复（核弹重置法）

一个 WorkBuddy 技能，解决蓝牙鼠标/键盘（如罗技 M240）在多台电脑间来回重连后突然连不上、或"搜得到但配对失败/连上就断"的问题。

## 适用场景

- 蓝牙 BLE 设备在公司/家里两台电脑间来回拿，突然连不上这台电脑
- 设置里"添加蓝牙设备"搜得到但配对失败、或连上又断（B 类）；或根本搜不到（A 类）
- 在设置里删设备删不掉

## 三层根因

1. 重复配对记录：BLE 设备用随机私有地址（RPA），每次重连地址都变，Windows 反复写新记录、旧的没清
2. 孤儿 HID 节点：删母设备时挂在下面的 HID 子节点没跟着删，残留成孤儿导致新配对地址冲突被回滚（B 类主因）
3. 注册表配对密钥残留：旧地址密钥残留在 BTHPORT\Parameters\Keys，新地址又建一条，两边打架

## 安装（WorkBuddy）

把本目录放进 WorkBuddy 技能目录后重启即可加载：

- 用户级：`~/.workbuddy/skills/windows-bluetooth-fix/`
- 项目级：`<项目>/.workbuddy/skills/windows-bluetooth-fix/`

## 使用

一句话流程：手机先测硬件没坏 → 跑只读诊断 → 双击 reset_bt.bat（自动提权）→ 重启 → 长按配对键重连。完整步骤见 SKILL.md。

## 关键坑（本技能已规避）

- .bat 必须纯 ASCII（中文会按 GBK 解析成乱码，切碎命令）
- 删除用通配（FriendlyName 匹配），别用固定 InstanceId
- Get-PnpDevice -Class HIDClass 会直接崩溃进程，改用全量枚举 + Where-Object 过滤

## 免责

核弹重置会移除蓝牙适配器并级联删掉其下子设备，重启后其他已配对设备（手机/耳机/平板）可能需重新配对一次。仅供排障，操作即视为知情。
