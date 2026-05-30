# Lenovo Legion Y7000 Hackintosh - macOS Sequoia 15.7.7

![macOS](https://img.shields.io/badge/macOS-15.7.7-Sequoia?logo=apple&labelColor=333)
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.7-blue)
![Status](https://img.shields.io/badge/Status-Stable-brightgreen)

## 🖥 硬件配置

| 部件 | 型号 |
|:----|:-----|
| **电脑** | Lenovo Legion Y7000 (Legion Y530-15ICH) |
| **CPU** | Intel Core **i5-8300H** (Coffee Lake-H) |
| **内存** | 16GB DDR4 2667MHz |
| **系统盘** | WDC SN720 NVMe SSD 512GB |
| **数据盘** | ST500LM021 HDD 500GB |
| **显卡** | Intel **UHD Graphics 630** (2048MB) |
| **声卡** | Realtek ALC (layout-id 18) |
| **有线网** | Realtek RTL8111 |
| **WiFi** | Broadcom BCM43xx (via OCLP patches) |
| **蓝牙** | Broadcom **BCM20702A0** (Foxconn USB) |
| **触控板** | I2C + PS/2 |

## ✅ 正常工作

| 功能 | 状态 |
|:----|:----:|
| **Intel UHD 630 显卡** | ✅ 硬件加速，Metal 3 |
| **WiFi** | ✅ Broadcom 已驱动 |
| **蓝牙** | ✅ 已修复（见下方说明） |
| **音频** | ✅ 内置喇叭 + 麦克风 |
| **有线网** | ✅ RealtekRTL8111 |
| **USB 端口** | ✅ 定制 USBToolBox 映射 |
| **触控板** | ✅ 多指手势 |
| **键盘** | ✅ 亮度/音量快捷键 |
| **电池** | ✅ 状态正常，AC 识别 |
| **NVMe** | ✅ TRIM 已启用 |
| **休眠/唤醒** | ✅ 正常 |
| **iMessage/Facetime** | ⚠️ 需自行生成 SMBIOS |

## 📋 需要自行配置

### 1. SMBIOS (三码)

**必须生成自己的序列号！** 用 **ProperTree** 或 **OCAT** 打开 `EFI/OC/config.plist`：

- `PlatformInfo → Generic → SystemSerialNumber`
- `PlatformInfo → Generic → MLB`
- `PlatformInfo → Generic → SystemUUID`
- `PlatformInfo → Generic → ROM`（建议设为有线网卡 MAC 地址）

推荐用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 生成，机型选择 **MacBookPro16,3**。

### 2. USB 端口映射

本 EFI 包含我的 Legion Y7000 USB 映射（UTBMap.kext），如果你的机型不同，请用 USBToolBox 重新生成。

### 3. OpenCore Legacy Patcher (OCLP) 打补丁

本 EFI 的 Broadcom WiFi 驱动依赖 **OCLP 系统补丁**。首次使用必须执行以下步骤：

**① 下载 OCLP**

从 [Dortania/OpenCore-Legacy-Patcher Releases](https://github.com/dortania/OpenCore-Legacy-Patcher/releases) 下载最新版 `OpenCore-Patcher-GUI.app.zip`

> 推荐版本：**v2.4.1** 或更新

**② 安装 OCLP**

解压后将 `OpenCore-Patcher.app` 拖入 **应用程序** 文件夹

**③ 应用 Root 补丁**

```bash
# 打开 OCLP
open /Applications/OpenCore-Patcher.app

# 点击: Post-Install Root Patch → Start Root Patching
# 等待补丁应用完成 → 重启
```

或在终端中直接运行：
```bash
/Applications/OpenCore-Patcher.app/Contents/MacOS/OpenCore-Patcher --patch_sys_vol
```

**④ 验证**

重启后 WiFi 应自动可用。可在系统信息中查看：
- Wi-Fi → 已连接
- 蓝牙 → 已开启

> ⚠️ **注意**：OCLP 补丁会修改系统根卷（/System/Library/Extensions/）。macOS 系统更新后会清除补丁，更新后需**重新运行 OCLP 打补丁**。
>
> ✅ OCLP 与本 EFI 的蓝牙修复方案（BlueToolFixup 2.6.8）兼容。

## ⚠️ 蓝牙修复说明

蓝牙芯片 **BCM20702A0** 上有一个已知问题：

- **BlueToolFixup 2.7.x** 有 bug（[GitHub #2505](https://github.com/acidanthera/bugtracker/issues/2505)），会设 `bluetoothExternalDongleFailed=01`
- 本 EFI 使用 **BlueToolFixup 2.6.8**（无此问题）
- boot-args 包含 `-btlfxbeta -btlfxboardid`
- **不要**添加 `-brcmfixup`（会破坏 WiFi）

## 🔧 Kext 清单

| Kext | 版本 | 用途 |
|:----|:---:|:-----|
| Lilu | 1.7.2 | 核心补丁框架 |
| VirtualSMC | 1.3.7 | SMC 模拟器 |
| WhateverGreen | 1.7.0 | 显卡修复 |
| AppleALC | 1.9.7 | 声卡驱动 |
| CPUFriend | 1.3.0 | CPU 电源管理 |
| NVMeFix | 1.1.3 | NVMe 稳定性 |
| RestrictEvents | 1.1.6 | 进程事件限制 |
| RealtekRTL8111 | 3.0.0 | 有线网卡 |
| AirportBrcmFixup | 2.2.0 | WiFi 修复 |
| BlueToolFixup | 2.6.8 | 蓝牙修复 |
| BrcmFirmwareData | 2.7.2 | 蓝牙固件数据 |
| BrcmPatchRAM3 | 2.7.2 | 蓝牙固件上传 |
| VoodooI2C | 2.9.1 | I2C 触控板 |
| VoodooPS2Controller | 2.3.7 | PS/2 键盘 |

## 📥 使用方法

1. 下载本 EFI
2. 用 ProperTree/OCAT 生成自己的 SMBIOS
3. 替换 UTBMap.kext（如需）
4. 放入 EFI 分区

## 📚 参考

- [OpenCore 官方文档](https://dortania.github.io/OpenCore-Install-Guide/)
- [Dortania 无线网卡指南](https://dortania.github.io/Wireless-Buyers-Guide/)
- [Acidanthera 仓库](https://github.com/acidanthera)
