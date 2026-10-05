<p></p>
<p align="center"><img src="https://i.imgur.com/HJnpvwQ.png" width="200" height="48"/> EFI</p>
<p align="center">
  <a href="https://github.com/acidanthera/OpenCorePkg">
  <img src="https://img.shields.io/badge/OpenCore-1.0.1-informational.svg">
 </a>
</p>

<a href="https://github.com/So1jon">
    <img src="https://img.shields.io/github/followers/So1jon?label=So1jon&logo=GitHub&style=social" />
</a> 

[![GitHub all releases](https://img.shields.io/github/downloads/So1jon/Hackintosh-Lenovo-Ideapad-3-15IML05/total?style=flat&logo=github&logoColor=white&color=1A91FF)](https://github.com/So1jon/Hackintosh-Lenovo-Ideapad-3-15IML05/releases)

# Final report: OpenCore EFI and DSDT

Lenovo laptop, Intel Core i3-10110U (Comet Lake-U), 20 GB RAM | SMBIOS MacBookAir8,1 | macOS 14 | OpenCore 1.0.1 | 2026-10-06

## Overall conclusion

The system is stable. The main problem (the Mac waking up by itself 8-9 seconds after going to sleep) is **solved**. The cause was in the USB map: the internal camera and Bluetooth ports were marked as external USB. On the latest boot (03:38) there are **no ACPI load errors**; the trackpad, Wi-Fi, Bluetooth, YogaSMC and USB power (USBX) all work. USB keyboard wake and long sleep were tested by the user and **work**.

## Final conclusion

- **Root cause:** in the USB map (`USBPro.kext`) every port was marked as external USB (`UsbConnector = 3`), including the internal camera (`HS07`) and Bluetooth (`HS10`). Changing both to `255` (internal) stopped the wake-up 8-9 seconds after sleep.
- **The GPRW / GPE 0x6D patches turned out to be unnecessary.** They were tested and removed; as a result USB wake was preserved (`SSDT-USBW` + `USBWakeFixup`).
- **Additional fixes:** USBX was enabled (`SSDT-USBX`), the `EC0 -> EC` patch now applies to all tables, and non-working or redundant SSDTs were removed. No ACPI errors remain at boot.
- **Final state:** sleep and wake (including USB keyboard and long sleep, per the user's confirmation), trackpad, Wi-Fi, Bluetooth, YogaSMC and battery all work.
- **Optional leftovers:** delete `SSDT-EC-USBX.aml` from the folder, review `HibernationFixup` and the unused Realtek kexts, remove the debug boot-args (`-v`, `debug`), and keep the EFI folder private (it contains serial, MLB, UUID, ROM).

## Status panel (live system, boot 03:38:14)

| Component | Status | Evidence |
|---|---|---|
| Sleep (instant wake) | **Resolved** | After the USB map fix: sleeps of 63, 45, 28, 21 and 50 s; the 8 s wake-up did not return |
| ACPI load errors | **None (0)** | No `Namespace lookup failure` / `while loading table` in the kernel log |
| Trackpad (I2C) | Working | `VoodooI2CHIDDevice = 1`, `MultitouchHIDEventDriver = 1`, `AppleActuatorDevice = 1` |
| USB power (USBX) | Working | `kUSBSleepPortCurrentLimit = 2100` (15 ports), `kUSBSleepPowerSupply = 5100` |
| Wi-Fi | Working | AirportItlwm 2.3.0 loaded, connected to "Beeline" (CNVi) |
| Bluetooth | Working | IntelBluetoothFirmware + BlueToolFixup, State: On |
| YogaSMC / battery | Working | IdeaSMC, IdeaVPC, IdeaWMIBattery active; battery 100% |
| USB keyboard wake | Working | Confirmed by the user (2.4G receiver) |
| Long sleep | Working | Confirmed by the user |
| Hibernation | Not tested | `HibernateMode None`, `hibernatemode 0` |

> **Note:** the last two tests are based on the user's observation. When this report was prepared, the `pmset` log (after the 03:38 boot) showed only the 50 s sleep at 03:42 (`Sleep Count: 1`), so these tests are not independently confirmed in the log.

## 1. Platform

| Parameter | Value | Source |
|---|---|---|
| Processor | Intel Core i3-10110U, 2 cores / 4 threads (Comet Lake-U, 10th gen) | sysctl |
| Memory / disk | 20 GB RAM; boot disk SATA SSD (SSSTC CVB-8D128-HP), plus 1 TB HDD (WD10SPZX) | sysctl, diskutil |
| Graphics | Intel UHD, ig-platform-id `0x3EA50009` (CFL framebuffer), device-id `0x3E9B` spoof | ioreg, config |
| Device | Lenovo laptop; EC: VPC2004 / IDEA2004; ACPI OEM: LENOVO CB-01 | DSDT |
| SMBIOS | MacBookAir8,1 (processor name shows as "Apple M4" via RestrictEvents) | config, NVRAM |
| macOS | Darwin 23.6.0 (macOS 14 Sonoma) | uname |
| Bootloader | OpenCore 1.0.1 (REL-101-2024-08-05) | NVRAM |

> **Correction.** Early in the analysis, judging by the DSDT structure, the machine was guessed to be "Alder Lake, i7-12700". That was **wrong**: the 24 root ports and 20 Processor objects in the DSDT come from Lenovo's generic template; the real machine is Comet Lake-U (i3-10110U).

## 2. DSDT analysis

| Parameter | Value |
|---|---|
| Signature / OEM / Table ID | DSDT / LENOVO / CB-01, OEM revision 1, ACPI revision 2 |
| Size / checksum | 245,154 bytes (245,617 in the live system after OC changes) / correct |
| Compiler | INTL 20200925; recompiling gives AML byte-identical to the original |
| Objects | 22,587 opcodes, 4,021 named objects (iasl); 206 Device, 1,530 Method, 114 OpRegion, 46 PowerResource, 20 Processor, 6 Mutex |
| Result | 0 errors, 187 warnings, 313 remarks (typical of vendor code, left untouched) |

**Structure**

- **PCI0:** 3 PEG, GFX0, IPU0, 24 root ports, SATA (6 ports + NVM1-3), XHC (14 USB2 + 10 USB3), XDCI, HDEF (+ SoundWire), GLAN, SBUS, CNVW (CNVi Wi-Fi), PUFS/PEMC/PSDC, ISHD, IMEI.
- **LPCB:** EC (PNP0C09; inside: BAT0, VPC0 = VPC2004, ITSD), HPET, RTC, IPIC, TIMR, PS2K, PS2M. **SerialIO:** I2C0-5, SPI0-2, UA00-02; I2C0.TPD0 (trackpad, SYNA2BA6).
- **Lenovo WMI:** WMI4, WMIL, WMIU, WMTF, HKDV. Reference-code leftovers: DOCK (SADDLESTRING), WFDE/WFTE (SampleDev/TestDev), TPD0 HID "XXXX0000".
- **Power:** PEPD (S0ix), AWAC, ADP0, LID0, PWRB, SLPB, Thunderbolt code (`_GPE.XTBT`).

**Sleep states, GPE and USB**

| Topic | Details |
|---|---|
| `_Sx` | S0 {0,0,0,0}, S1 {1,0,0,0}, **S3 {5,0,0,0}**, S4 {6,0,0,0}, S5 {7,0,0,0} |
| `_PTS` / `_WAK` / `_PIC` | `_PTS`: Thunderbolt and S3/S4 GPIO preparation. `_WAK` (2,046 bytes): writes the OS type (OSTY) to the EC, bus-checks the RPxx ports, restores LID state. OSYS is 1000, or 0x7DF via XOSI |
| `_L61` | PCIe hot-plug (6 KB); not a wake source |
| `_L69` / `_L6F` / `_L26` | PCIe PME / RTD3-PEG GPIO / WLAN GPIO wake |
| `_L62` / `_L66` / `_L72` | Thermal-HWP / graphics SCI / RTC alarm |
| **GPE 0x6D** | No handler; GLAN, XHC, XDCI, HDEF and CNVW all call `GPRW (0x6D, 0x03)` in `_PRW`, the OS manages it |
| USB (`_UPC`) | External: HSP1, HSP3, HSP4, SSP1, SSP2. Internal: HSP2, HSP5, HSP6, **HSP7 (camera)**, HS10 (Bluetooth). Not connectable: HSP8, HSP9, SSP3-6. Hidden: HS11-14, SS07-10 |
| Warnings (187) | 3115 x92, 3124 x31, 3107 x25, 3168 x20, 3073 x10, other x9 |

## 3. The sleep problem: diagnosis and fix

**Symptom.** The Mac wakes up by itself 8-9 seconds after going to sleep. The `pmset` log shows the same cause for all 11 events: `due to XDCI USBW/User`, detail `DriverReason:XHC`.

### 3.1 Tests (`pmset -g log`)

| Time | State | Sleep | Result |
|---|---|---|---|
| 00:29 | Original DSDT (no patch) | 8 s | Instant wake (XDCI USBW) |
| 00:52 | Selective GPRW patch (GLAN, XDCI, HDEF, CNVW), receiver connected | 8 s | Instant wake (USBW) |
| 00:57 | Same + receiver unplugged | 8 s | Instant wake |
| 00:59 | Same + receiver unplugged + Bluetooth off | 9 s | Instant wake |
| 01:10 | Bluetooth kexts not loaded | 8 s | Instant wake |
| 01:20 | **USB map: HS07, HS10 = 255** (+ patch, BT kext enabled) | 63 s | **Stable** |
| 01:48 | Same state | 45 s | Stable |
| 02:08 | **USB map fixed, no patches** | 28 s | **Stable** |
| 02:37 | Same state | 21 s | Stable |
| 03:42 | **Final configuration** (USB map + USBX, no ACPI errors) | 50 s | **Stable** |

Note: in the 02:08 and 02:37 rows the no-patch state was determined from the copied `config.plist` and the XDCI wake label. The full `SSDT-GPRW` (GPE 0x6D off for everything) also stopped the wake-up (65 s, 26 s), but it also disables USB wake.

### 3.2 Cause

- The **only new change** between the last test with the instant wake (01:10) and the first stable test (01:20) was the USB map (HS07 and HS10: 3 -> 255). The no-patch tests are stable too: the patches are not needed.
- Mechanism (assumption): in the map every port counts as external USB, so macOS applies the external-device wake policy to the internal camera and Bluetooth (the `USBExternalDevice` sleep factor in the kernel log).
- **The "XDCI USBW" label:** XDCI does not exist as a PCI device (not in ioreg); it is just an ACPI entry attached to GPE 0x6D. USBW is the device created by `SSDT-USBW` (`_PRW = XHC._PRW`). The label shows the wake path, not the culprit.
- Ruled-out suspects: GLAN/XDCI/HDEF/CNVW, the 2.4G receiver, the Bluetooth radio and the Bluetooth firmware kext.

### 3.3 How it was verified

At each stage a live DSDT dump was taken with MaciASL and compared with the original AML **semantically** (both tables re-decompiled with the same iasl, display-only differences filtered out). Calls to an external SSDT method were written invalidly in the dump (syntax error), so they were fixed in a temporary copy, which then compiled with 0 errors.

## 4. EFI contents

| Folder | Contents | Size |
|---|---|---|
| `EFI/BOOT` | `BOOTx64.efi` (2024-08-05) | 24 KB |
| `EFI/OC` | `OpenCore.efi` (2024-08-05), `config.plist` (51,144 bytes) | ~0.7 MB |
| `OC/ACPI` | 16 files: 15 enabled SSDTs + 1 unused (`SSDT-EC-USBX.aml`) | ~4 KB |
| `OC/Drivers` | OpenRuntime, HfsPlus, OpenCanopy, AudioDxe, NvmExpressDxe, ResetNvramEntry (enabled); ToggleSipEntry (disabled) | 444 KB |
| `OC/Kexts` | 34 kext bundles (38 config entries, all enabled) | 51 MB |
| `OC/Resources` | OpenCanopy: Audio (74), Font (4), Image (Acidanthera), Label (22); Tools is empty | 1.3 MB |

Total 54 MB, no `._` (AppleDouble) files. Drivers come from the same release as OpenCore (2024-08-05; HfsPlus 2024-01-27). Every enabled file exists.

### 4.1 Kexts (in load order, all enabled)

| # | Kext | Version | Purpose |
|---|---|---|---|
| 1 | Lilu | 1.7.1 | kext patching framework |
| 2 | VirtualSMC | 1.3.8 | SMC emulation |
| 3 | WhateverGreen | 1.7.0 | graphics |
| 4 | ECEnabler | 1.0.6 | EC fields (battery) |
| 5 | CpuTscSync | 1.1.2 | TSC sync |
| 6 | AirportItlwm | 2.3.0 | Intel Wi-Fi (active) |
| 7 | HoRNDIS | 9.2 | Android USB tethering |
| 8 | HWPEnabler | 1.2 | HWP |
| 9 | RTCMemoryFixup | 1.0.8 | RTC memory |
| 10 | SMCBatteryManager | 1.3.8 | battery |
| 11 | SMCProcessor | 1.3.8 | CPU sensors |
| 12 | SMCSuperIO | 1.3.8 | SuperIO |
| 13 | BlueToolFixup | 2.6.8 | Intel Bluetooth (Monterey+) |
| 14 | IntelBluetoothFirmware | 2.5.0 | Intel BT firmware |
| 15 | RestrictEvents | 1.1.6 | CPU name and others |
| 16 | EnergyDriver | 3.7.0 | Intel energy |
| 17 | CPUFriend | 1.3.0 | CPU power |
| 18 | CPUFriendDataProvider | 1.0.1 | CPUFriend data |
| 19 | NVMeFix | 1.1.3 | NVMe |
| 20 | FeatureUnlock | 1.1.8 | feature unlock |
| 21 | BrightnessKeys | 1.0.3 | brightness keys |
| 22 | USBPro (USB map) | 1.2 | **USB port map (HS07, HS10 = 255)** |
| 23 | AppleALC | 1.9.6 | audio |
| 24 | VoodooInput | 1.1.6 | VoodooI2C plugin |
| 25 | VoodooI2CServices | 1.0 | VoodooI2C plugin |
| 26 | VoodooGPIO | 1.1 | VoodooI2C plugin |
| 27 | VoodooI2C | 2.9.1 | I2C trackpad (active) |
| 28 | VoodooI2CHID | 1.0 | I2C HID |
| 29 | VoodooPS2Controller | 2.3.8 | PS/2 keyboard |
| 30 | VoodooPS2Keyboard | 2.3.8 | PS/2 plugin |
| 31 | HibernationFixup | 1.5.5 | hibernation (not needed now) |
| 32 | USBWakeFixup | 1.0 | USB wake (paired with SSDT-USBW) |
| 33 | RtWlanU | 1830.32.b27 | Realtek USB Wi-Fi (not connected now) |
| 34 | RtWlanU1827 | 1827.4.b36 | Realtek USB Wi-Fi (other variant) |
| 35 | RealtekCardReader | 0.9.7 | card reader |
| 36 | RealtekCardReaderFriend | 1.0.4 | helper |
| 37 | GenericCardReaderFriend | 1.0.4 | helper |
| 38 | YogaSMC | 1.5.3 | Lenovo features (active) |

The order is correct (Lilu first, VoodooInput/VoodooGPIO before VoodooI2C). The kexts in the folder and the config match completely.

### 4.2 SSDTs (15 enabled, all load without errors)

| SSDT | Bytes | Purpose |
|---|---|---|
| `SSDT-AWAC` | 73 | AWAC/RTC conflict (STAS = 1: the real RTC is enabled) |
| `SSDT-XOSI` | 421 | `_OSI` wrapper (Darwin = Windows) |
| `SSDT-OSYS` | 574 | sets OSYS (PCI1._INI); duplicates XOSI, harmless |
| `SSDT-PLUG` | 900 | XCPM (CPU power) |
| `SSDT-MCHC` | 104 | MCHC device |
| `SSDT-SBUS` | 190 | SBUS.BUS0 + DVL0 (SMBus) |
| `SSDT-DTGP` | 100 | DTGP helper method |
| `SSDT-MEM2` | 176 | MEM2 for the iGPU |
| `SSDT-PNLFCFL` | 141 | Brightness (PNLF) |
| `SSDT-ALS0` | 132 | Ambient light sensor (ACPI0008) |
| `SSDT-TPD0` | 468 | Trackpad: `_CRS`/`_DSM` replacement (relies on the XCRS/XTSM renames) |
| `SSDT-RHUB` | 110 | `_STA` for XHC.RHUB (replacement for macOS) |
| `SSDT-USBX` | 217 | USB power properties (USBX only; does not create an EC) |
| `SSDT-USBW` | 150 | USBW device (`_PRW = XHC._PRW`), paired with USBWakeFixup |
| `SSDT-VirutalNetCard` | 215 | Null Ethernet (RMNE); typo in the file name (Virutal) |

Removed (from config and folder): `SSDT-RCSM`, `SSDT-ECRW`, `SSDT-YVPC` (no H_EC), `SSDT-NO-CNVW` (did not work), `SSDT-HPET`, `SSDT-RTC` (failed to load, redundant). Left in the folder: `SSDT-EC-USBX.aml` (not in config, replaced by `SSDT-USBX`; can be deleted).

### 4.3 ACPI > Patch (12, all enabled)

| # | Signature | Note |
|---|---|---|
| 1, 2, 7 | all tables | RTC IRQ 8, TIMR IRQ 0, IPIC IRQ 2 |
| 3 | DSDT | `_OSI -> XOSI` |
| 4 | DSDT | **`TPD0 _CRS -> XCRS`** (Find `20 5F 43 52 53`; for the trackpad, `SSDT-TPD0` relies on it) |
| 5 | DSDT | `TPD0 _DSM -> XTSM` |
| 6 | all tables | `HPET _STA -> XSTA` (there is no `SSDT-HPET`, so HPET has no `_STA`: it is always enabled; the original working state) |
| 8 | DSDT | `SAT0 -> SATA` |
| 9 | **all tables** | `EC0 -> EC` (the BIOS SSDT also uses `EC0_`, so it applies to all tables) |
| 10, 11, 12 | DSDT | `HDAS -> HDEF`, `HECI -> IMEI`, `PNLF -> XNLF` |

### 4.4 config.plist: other sections

| Section | Key settings |
|---|---|
| Kernel > Quirks | AppleCpuPmCfgLock, AppleXcpmCfgLock, CustomSMBIOSGuid, DisableIoMapper, DisableLinkeditJettison, PanicNoKextDump, PowerTimeoutKernelPanic |
| Kernel > Patch | AppleRTC: disable RTC wake scheduling; IOAHCIBlockStorage: TRIM (3) |
| Booter > Quirks | AvoidRuntimeDefrag, EnableSafeModeSlide, EnableWriteUnprotector, FixupAppleEfiImages, ProvideCustomSlide, SetupVirtualMap, SyncRuntimePermissions |
| UEFI | ConnectDrivers, ReleaseUsbOwnership, RequestBootVarRouting, EnableVectorAcceleration; Audio enabled (Pci 1F,3) |
| Misc | PickerMode External (OpenCanopy), LauncherOption Full, Timeout 59; SecureBootModel Disabled, Vault Optional, ScanPolicy 0, HibernateMode None |
| NVRAM | boot-args: `-v keepsyms=1 debug=0x100 swd_panic=1`; csr-active-config 0 (SIP enabled); revcpu/revcpuname |
| PlatformInfo | Generic, MacBookAir8,1, UpdateSMBIOSMode Custom; serial/MLB/UUID/ROM are present (not shown) |
| DeviceProperties | XHC (device-id spoof), CNVi, SerialIO, IMEI, SATA, HDEF (layout-id 20), iGPU (backlight, framebuffer patch) |

## 5. ACPI load errors: found and resolved

The first boot log (kernel, `AppleACPIPlatform`) showed several SSDTs failing to load. The current boot has **no errors at all**.

| Table / error | Cause | Fix |
|---|---|---|
| BIOS SSDT: `EC0_` (AE_NOT_FOUND) | The `EC0 -> EC` patch applied only to the DSDT, while the BIOS SSDT still uses `EC0_` | The patch signature was changed to all tables. **Worked** |
| `SSDT-EC-USBX`: `EC__` (AE_ALREADY_EXISTS) | The SSDT tries to create a fake EC while a real EC exists; the whole table was dropped and USBX was missing | `SSDT-USBX` (USBX only). **Worked**: 2100 / 5100 |
| `SSDT-RTC`: `RTC_` (AE_ALREADY_EXISTS) | `RTC_` already exists in the DSDT; `SSDT-AWAC` (STAS = 1) enables it, AppleRTC is active | `SSDT-RTC` removed (redundant) |
| `SSDT-HPET`: `_CRS` (AE_ALREADY_EXISTS) | Patch #4 actually acted on `TPD0 _CRS`, not HPET (its comment was wrong); HPET `_CRS` was not renamed | `SSDT-HPET` removed (original state kept) |
| H_EC (`RCSM`, `ECRW`, `YVPC`) | There is no `H_EC` in the DSDT (the EC is named `LPCB.EC`) | Removed; YogaSMC works through VPC0 |

> **A mistake in the analysis and its correction.** An attempt to "fix" the HPET patch (a Base-scoped HPET rename) **broke the trackpad**: patch #4 actually renamed `TPD0 _CRS`, and `SSDT-TPD0` relies on that. With the new patch `TPD0 _CRS` was no longer renamed, `SSDT-TPD0` failed to load (`VoodooI2CHIDDevice = 0`). Once the original patch was restored (Find `20 5F 43 52 53`) the trackpad recovered (`VoodooI2CHIDDevice = 1`). Evidence: the live DSDT had `TPD0.XCRS`, while HPET `_CRS` was in place.

Harmless messages: "Unsupported module-level executable opcode 0x70" (a limitation of the 2016 ACPICA in macOS), "no ECDT", "cannot translate ACPI object 14".

## 6. Findings and recommendations

### 6.1 Resolved

| Finding | How it was resolved |
|---|---|
| Instant wake (8-9 s) | USB map: camera (HS07) and Bluetooth (HS10) `UsbConnector` 3 -> 255 |
| USBX inactive (USB sleep power 0) | `SSDT-USBX` instead of `SSDT-EC-USBX` |
| BIOS SSDT could not find `EC0_` | `EC0 -> EC` patch applied to all tables |
| H_EC SSDTs (`RCSM`, `ECRW`, `YVPC`), `SSDT-NO-CNVW`, `SSDT-RTC`, `SSDT-HPET` | Removed (non-working or redundant) |
| Trackpad regression (my HPET patch) | The original `TPD0 _CRS` patch was restored |
| Unneeded GPRW/NPRW/ZPTS patches, wrong comments | Removed / corrected |

### 6.2 Open items

| Level | Finding | Recommendation |
|---|---|---|
| Info | USB keyboard wake and long sleep were confirmed by the user but are not recorded in the `pmset` log. | Optional: confirm with `pmset -g log` after the next sleep. |
| Low | `SSDT-EC-USBX.aml` is still in the folder (not in config). | Delete it. |
| Low | Patch #6 (`HPET _STA -> XSTA`) is left without `SSDT-HPET`: HPET has no `_STA` and is always enabled (the original working state). | Leave it, or disable it if not needed. |
| Low | `RtWlanU` and `RtWlanU1827` are both enabled; no Realtek USB device is connected now. | If you use a dongle, keep the matching one. |
| Low | `HibernationFixup` is enabled, `HibernateMode None`. | Not needed, can be disabled. |
| Low | Debug mode: `-v`, `keepsyms=1`, `debug=0x100`, AppleDebug/ApplePanic. | Remove once stable. |
| Low | `SSDT-OSYS` and `SSDT-XOSI` both set OSYS; "Virutal" typo in a file name. | Harmless, optional. |
| Info | AppleCpuPmCfgLock and AppleXcpmCfgLock are both enabled; SecureBoot/Vault/ScanPolicy are the usual values. | Not needed if CFG Lock is disabled in the BIOS. |
| Private | `config.plist` contains the serial number, MLB, SmUUID and ROM. | Do not share the EFI folder with others. |

Verified successfully: every enabled SSDT, driver and kext file exists; kext order is correct; OpenCore and drivers are from one release; SIP is enabled; no `._` files in the copy; no ACPI errors in the boot log.

## 7. Appendix: change log

| Change | File | Status |
|---|---|---|
| GPRW -> XPRW + `SSDT-GPRW` (full wake disable) | `config.plist`, `SSDT-GPRW.aml` | tested, removed |
| 4 Base patches (GPRW -> NPRW) + `SSDT-GPRW-USB` | `config.plist`, `SSDT-GPRW-USB.aml` | tested, removed |
| `_PTS -> ZPTS` patch (landed in the wrong place) | `config.plist` | disabled |
| Temporarily disabling the Bluetooth kexts (test) | `config.plist` | reverted |
| **USB map: HS07, HS10 `UsbConnector` 3 -> 255** | `USBPro.kext/Contents/Info.plist` | **main fix, kept** |
| `SSDT-RCSM`, `YVPC`, `ECRW`, `NO-CNVW` removed; comments | `config.plist`, `ACPI/` | applied |
| `EC0 -> EC` patch for all tables | `config.plist` | applied, worked |
| `SSDT-USBX` instead of `SSDT-EC-USBX` | `config.plist`, `ACPI/SSDT-USBX.aml` | applied, worked |
| HPET Base patch (my mistake: broke the trackpad) | `config.plist` | reverted |
| Original `TPD0 _CRS` patch restored; `SSDT-HPET` and `SSDT-RTC` removed | `config.plist`, `ACPI/` | **applied, trackpad recovered** |

Backups are on the EFI partition (in the `OC` folder): `config-backup-20261006-0043`, `-bt-test-0106`, `-before-usbmap-0112`, `-before-nprw-off-0141` and `USBPro Info.plist.bak-20261006-0112`. They are not in the Downloads copy. The earlier PDF (`EFI-Yakuniy-Hisobot.pdf`) contains a wrong conclusion about the HPET error; this report replaces it.


| Information           | Result | ID Information                                                 | Operating system  | Model ID        |
| --------------------- | ------ | -------------------------------------------------------------- | ----------------- | --------------- |
| CPU Single-Core Score | 1250   | [ID 10211314](https://browser.geekbench.com/v6/cpu/10211314)   | `macOS Sonoma`    | `MacBookAir8,1` |
| CPU Multi-Core Score  | 2065   | [ID 10211314](https://browser.geekbench.com/v6/cpu/10211314)   | `macOS Sonoma`    | `MacBookAir8,1` |
| iGPU OpenCL Score     | 3309   | [ID 3577045](https://browser.geekbench.com/v6/compute/3577045) | `macOS Sonoma`    | `MacBookAir8,1` |
| iGPU Metal Score      | 4602   | [ID 3577034](https://browser.geekbench.com/v6/compute/3577034) | `macOS Sonoma`    | `MacBookAir8,1` |




### Credits:

- [Apple](https://www.apple.com) for macOS.
- [Acidanthera](https://github.com/acidanthera) for most of the kexts.
- [goodwin](https://github.com/goodwin) for ALCPlugFix.
- [RehabMan](https://github.com/RehabMan) for some patches.
- [Steve Zheng](https://github.com/stevezhengshiqi) for some patches.
- [Sniki](https://github.com/Sniki) for some patches.
- [daliansky](https://github.com/daliansky) for some patches.
- [Moh_Ameen](https://github.com/ameenjuz) for some patches.
- [OpenIntelWireless](https://github.com/OpenIntelWireless) for Intel WiFi drivers.


### Disclaimer:

⚠️ Highly Recommend you to build the EFI for your device on your own and only use this for reference even though you have the same device as mine since, when something fails you will be on your own.

⚠️ If you want to report or rasie an issue, you must mention your device details in it. And give a detailed information about your issue(images or videos are encouraged)


### You can contact me through:

[![](https://img.shields.io/badge/iCloud-nusratov.sobirjon@icloud.com-informational?style=flat&logo=apple&logoColor=white&color=cbcdcc)](mailto:nusratov.sobirjon@icloud.com)[![](https://img.shields.io/badge/Telegram-@Sobirjon_Nusratov-informational?style=flat&logo=telegram&logoColor=white&color=89e2ff)](https://t.me/Sobirjon_Nusratov)[![](https://img.shields.io/badge/Facebook-Nusratov_Sobirjon-informational?style=flat&logo=facebook&logoColor=white&color=3a4dc9)](https://www.facebook.com/Sobirjon.Nusratov)


