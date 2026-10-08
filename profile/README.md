# Barley Project

**Open-source research and development for Lenovo TB330FU and TB337FC tablets.**

Barley Project is a community-driven initiative dedicated to reverse engineering, firmware research, custom ROM development, and low-level system development for Lenovo tablets powered by MediaTek chipsets.

Our goal is to improve device accessibility, extend hardware lifespan, and document the inner workings of devices that receive limited support from the manufacturer.

## Supported Devices

| Device | Codename / Platform | Status |
|---|---|---|
| Lenovo TB330FU | `barley_prc_commercial_wifi` / `barley_row_wifi` | Active research |
| Lenovo TB337FC | TB330FU-based OEM / education variant | Active research |

Hardware revisions, firmware generations, and regional variants may differ. Compatibility should never be assumed based solely on the model name.

## Research Areas

- **Bootloader Research** — MediaTek Preloader, LK, SECCFG, and secure boot mechanisms.
- **Firmware Reverse Engineering** — Partition layouts, Download Agents, and proprietary firmware formats.
- **Custom ROM Development** — Android system modification and alternative firmware porting.
- **Linux & UEFI** — Mainline Linux bring-up, device tree development, and EDK2 experimentation.
- **Recovery & Unbricking** — Flashing tools, backup methods, and device recovery.
- **Hardware Documentation** — Chipsets, board revisions, peripherals, and device-specific differences.

---

## Unlocking Your TB330FU / TB337FC Bootloader

> [!WARNING]
> **Experimental procedure. Use at your own risk.**
>
> Bootloader unlocking modifies security-related device configuration and can cause data loss, boot failure, or permanent device damage.
>
> Back up your important data and device-specific partitions before proceeding.
>
> This procedure was tested on a specific TB337FC/TB330FU hardware and firmware combination. Other TB330FU production revisions may behave differently.

Some Lenovo TB330FU and TB337FC variants do not support conventional Fastboot bootloader unlocking, or their firmware imposes additional restrictions.

We maintain a modified version of MTKClient with support for the **Carbonara exploit** and an experimental MediaTek Download Agent storage-write patch.

**Repository:** [Barley-Project/mtkclient-tb330fu-carbonara](https://github.com/Barley-Project/mtkclient-tb330fu-carbonara)

### 1. Get the Modified MTKClient

```bash
git clone https://github.com/Barley-Project/mtkclient-tb330fu-carbonara.git
cd mtkclient-tb330fu-carbonara
pip install -r requirements.txt
```

For Windows, configure a compatible USB driver such as WinUSB for the MediaTek Preloader interface.

The tested configuration uses:

```powershell
$env:MTKCLIENT_USB_BACKEND = "winusb"
```

### 2. Prepare the Required Files

You may need the following files from firmware matching your device:

- MediaTek Download Agent (`MTK_AllInOne_DA-resign.bin`)
- Device-compatible Preloader image
- Authentication file (`auth_sv5.auth`), where required

A tested DA sample is included in the repository under `vendor/TB330FU/`.

**Do not blindly reuse a Preloader, DA, or authentication file from an unrelated firmware revision.**

### 3. Unlock Using SECCFG

Our modified MTKClient supports the `--experimental-da-write` flag, which enables a guarded runtime DA storage-write patch.

In a Windows PowerShell terminal:

```powershell
$DA   = ".\vendor\TB330FU\MTK_AllInOne_DA-resign.bin"
$PL   = "PATH_TO_MATCHING_PRELOADER.bin"
$AUTH = "PATH_TO_MATCHING_AUTH.auth"

$env:MTKCLIENT_USB_BACKEND = "winusb"

python.exe .\mtk.py da seccfg unlock `
    --ptype carbonara `
    --experimental-da-write `
    --loader "$DA" `
    --preloader "$PL" `
    --auth "$AUTH" `
    --vid 0x0e8d `
    --pid 0x2000
```

Replace the placeholder paths with files matching your firmware.

Power off your device and connect it in the supported MediaTek Preloader mode while MTKClient is waiting for a connection.

On the tested device, MTKClient identified **SECCFG V4** and successfully changed the bootloader lock state.

The experimental DA patch is applied in memory during the current session; it does not permanently replace the device's Download Agent.

### 4. Verify the Unlock State

Boot into Fastboot mode and run:

```bash
fastboot getvar unlocked
```

Expected output:

```text
unlocked: yes
```

This confirms the lock state reported by the bootloader.

### What If Unlocking Fails?

**Do not repeatedly force writes or erase SECCFG.**

There may be different TB330FU hardware revisions and firmware generations with incompatible security configuration layouts or encryption behavior.

If the operation fails:

1. Record the exact device model, hardware revision, and firmware build.
2. Read and back up the existing `seccfg` partition.
3. Determine the SECCFG format and investigate its validation and encryption logic.
4. Compare the relevant implementation against the version supported by MTKClient.
5. Report the findings or contribute compatibility fixes to the repository.

A successful unlock on one TB330FU revision does **not** guarantee compatibility with every other revision.

For detailed research notes and recovery information, see:

[TB330FU Unlock Research](https://github.com/Barley-Project/mtkclient-tb330fu-carbonara/blob/main/TB330FU_UNLOCK_RESEARCH.md)

---

## 中文说明：TB330FU / TB337FC 解锁

部分联想 TB330FU 及 TB337FC 教育定制机型无法通过常规 Fastboot 命令解锁 Bootloader。

本组织维护了一个修改版 MTKClient：

[mtkclient-tb330fu-carbonara](https://github.com/Barley-Project/mtkclient-tb330fu-carbonara)

其主要功能包括：

- Carbonara 利用流程
- MediaTek Preloader 通信
- Windows WinUSB 支持
- 实验性 DA 存储写入补丁
- SECCFG Bootloader 解锁

**基本解锁方法：**

使用与设备固件匹配的 DA、Preloader 和认证文件，添加 `--experimental-da-write` 参数，并执行：

```text
python mtk.py da seccfg unlock
```

实际执行时请使用上方包含完整参数的 PowerShell 命令。

如果成功，进入 Fastboot 后执行：

```text
fastboot getvar unlocked
```

正常情况下应返回：

```text
unlocked: yes
```

**如果解锁失败，请先提取并备份自己的 `seccfg` 分区，研究其格式、加密方式和验证逻辑。**

TB330FU 可能存在不同生产批次、硬件修订及固件版本，不能保证所有机型使用相同的 SECCFG 实现。

请勿直接刷入其他设备的 SECCFG，也不要清空设备唯一数据分区。

---

## Contributing

Contributions are welcome, including:

- Device trees and kernel patches
- Bootloader and firmware research
- Hardware revision documentation
- Recovery procedures
- Bug reports and compatibility testing
- Technical documentation and translations

When reporting an issue, please include the device model, firmware version, relevant logs, and steps to reproduce.

Do not publish device-unique identifiers, private authentication files, cryptographic keys, or personal data.

## Disclaimer

Barley Project is an independent community research initiative and is **not affiliated with, endorsed by, or sponsored by Lenovo or MediaTek**.

All software and documentation are provided for educational, research, and authorized device-maintenance purposes, without warranty.

Use these tools only on devices you own or are explicitly authorized to service.

**Research. Understand. Build.**
