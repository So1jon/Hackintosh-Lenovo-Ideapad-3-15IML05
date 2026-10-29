<p></p>
<p align="center"><img src="https://i.imgur.com/HJnpvwQ.png" width="200" height="48"/> EFI</p>
<p align="center">
  <a href="https://github.com/acidanthera/OpenCorePkg">
  <img src="https://img.shields.io/badge/OpenCore-1.0.8-informational.svg">
 </a>
  <img src="https://img.shields.io/badge/macOS-Tahoe%2026.7.2-blue.svg">
  <img src="https://img.shields.io/badge/macOS-Sonoma%2014-lightgrey.svg">
  <img src="https://img.shields.io/badge/CPU-i3--10110U%20Comet%20Lake-0071C5.svg">
</p>

<a href="https://github.com/So1jon">
    <img src="https://img.shields.io/github/followers/So1jon?label=So1jon&logo=GitHub&style=social" />
</a>

[![GitHub all releases](https://img.shields.io/github/downloads/So1jon/Hackintosh-Lenovo-Ideapad-3-15IML05/total?style=flat&logo=github&logoColor=white&color=1A91FF)](https://github.com/So1jon/Hackintosh-Lenovo-Ideapad-3-15IML05/releases)

# Lenovo IdeaPad 3 15IML05 Hackintosh

**OpenCore EFI, DSDT analysis and a full working log for macOS Sonoma 14 and macOS Tahoe 26**

Intel Core i3-10110U (Comet Lake-U) | Intel UHD 620 | 20 GB RAM | Intel Wireless-AC 9560 | current: **macOS Tahoe 26.7.2**, OpenCore 1.0.8, SMBIOS `MacBookPro16,2` | last update **2026-10-29**

> **In short.** Everything that matters works on Tahoe 26.7.2: graphics, audio, Wi-Fi, Bluetooth, USB, trackpad, Fn keys, sleep and wake, and OTA updates. Getting there took four Tahoe-specific fixes on top of the Sonoma EFI (xHCI `device-id`, the renamed USB port key, an Intel Bluetooth vendor-ID rewrite and `revpatch=sbvmm`) plus a CPU power profile (CPUFriend) rebuilt for the new SMBIOS. The one thing that does not work is DRM-protected media (Apple Music, full Apple TV content), a known limit of iGPU-only Hackintosh.

## Status

| Component | Sonoma 14 | Tahoe 26.7.2 | Notes |
|---|:---:|:---:|---|
| Boot, OpenCore picker | ✅ | ✅ | Graphical picker (OpenCanopy) |
| Graphics, Intel UHD 620 | ✅ | ✅ | Metal 3, 1920x1080, backlight |
| Audio (Realtek ALC 230) | ✅ | ✅ | Tahoe needs SimpleHDA (AppleHDA was removed) |
| Wi-Fi (Intel 9560 CNVi) | ✅ | ✅ | AirportItlwm 2.3.0 / 2.4.0 |
| Bluetooth (Intel 9560) | ✅ | ✅ | Tahoe needs the vendor-ID rewrite, see [Bluetooth](#7-bluetooth) |
| USB ports | ✅ | ✅ | Tahoe needs `9DED` and `usb-port-number`, see [USB](#6-usb) |
| Trackpad (I2C), keyboard, Fn keys | ✅ | ✅ | VoodooI2C, VoodooPS2, BrightnessKeys, YogaSMC |
| Battery, Lenovo features | ✅ | ✅ | YogaSMC, SMCBatteryManager |
| Sleep and wake | ✅ | ✅ | Instant wake fixed on Sonoma; Tahoe tested up to 64 min |
| macOS updates (OTA) | ✅ | ✅ | Tahoe needs `revpatch=sbvmm` on a T2 SMBIOS |
| CPU power management (CPUFriend) | ✅ | ✅ | Tahoe uses a profile rebuilt for `MacBookPro16,2` plus `-cpufbeta`, see [CPUFriend](#9-cpu-power-management-cpufriend) |
| Hibernation | ⏸️ | ⏸️ | Disabled on purpose (`hibernatemode 0`) |
| DRM media (Apple Music, Apple TV) | ❌ | ❌ | FairPlay does not work on iGPU-only Hackintosh, see [DRM](#11-drm-media-apple-music-and-apple-tv) |

✅ works | ⏸️ disabled on purpose | ❌ does not work

## Contents

1. [History and conclusions](#1-history-and-conclusions)
2. [Hardware and SMBIOS](#2-hardware-and-smbios)
3. [EFI](#3-efi)
4. [ACPI and DSDT](#4-acpi-and-dsdt)
5. [Sleep and wake](#5-sleep-and-wake)
6. [USB](#6-usb)
7. [Bluetooth](#7-bluetooth)
8. [Audio, Wi-Fi, graphics and input](#8-audio-wi-fi-graphics-and-input)
9. [CPU power management (CPUFriend)](#9-cpu-power-management-cpufriend)
10. [macOS updates](#10-macos-updates)
11. [DRM media (Apple Music and Apple TV)](#11-drm-media-apple-music-and-apple-tv)
12. [Xcode and developer tools](#12-xcode-and-developer-tools)
13. [Open items](#13-open-items)
14. [Benchmarks](#14-benchmarks)
15. [Change log](#15-change-log)

---

## 1. History and conclusions

| Date | Step | Result |
|---|---|---|
| 2026-10-06 | Sonoma 14 (OpenCore 1.0.1, `MacBookAir8,1`): sleep and ACPI work | Instant wake fixed by the USB map; no ACPI load errors |
| 2026-10-26 | Tahoe 26.7 installed next to Sonoma (OpenCore 1.0.8, `MacBookPro16,2`) | Audio, Wi-Fi, sleep, trackpad, Fn work; USB and Bluetooth do not |
| 2026-10-27 | Tahoe-specific fixes for USB and Bluetooth | USB fixed, then Bluetooth fixed |
| 2026-10-28 | `revpatch=sbvmm`, update to 26.7.2 | Update offered and installed; all fixes survived |
| 2026-10-28 | Sonoma upgraded in place | Tahoe is now the only system on the drive |
| 2026-10-29 | CPUFriend profile rebuilt for Tahoe | Profile applied (`epp 8`), survives sleep |
| 2026-10-29 | Sleep tests, Xcode, DRM analysis | This document |

**Conclusions**

- **Sonoma:** the 8-9 s instant wake was caused by the USB map (internal camera `HS07` and Bluetooth `HS10` marked as external). Setting both to `UsbConnector = 255` fixed it; the GPRW / GPE 0x6D patches turned out to be unnecessary. ACPI errors were cleaned up (`EC0 -> EC` for all tables, `SSDT-USBX` instead of `SSDT-EC-USBX`, redundant SSDTs removed).
- **Tahoe:** the same EFI base works after four changes: xHCI `device-id` `9DED` from ACPI (Tahoe dropped the old Intel xHCI classes), the per-port key `usb-port-number` (renamed by Apple), an Intel Bluetooth vendor-ID rewrite to `0x0A12` (Tahoe's `bluetoothd` accepts only allow-listed USB dongles) and `revpatch=sbvmm` (update notifications on a T2 SMBIOS).
- **CPUFriend:** the Sonoma power profile was built for `MacBookAir8,1`. On Tahoe the profile was rebuilt from Apple's own `MacBookPro16,2` data and (with the earlier kext build) only worked after `-cpufbeta`; this was proven with a marker value, not assumed (section 9).
- **Limitation:** DRM media never worked on this laptop; on Tahoe the cause is visible in the logs (FairPlay key creation fails).
- **Method:** every conclusion below is backed by logs, `ioreg`, `pmset`, Apple's own kext plists or a disassembly of Apple's daemon, and each EFI change was tested one at a time. Two wrong turns are documented on purpose (an HPET patch that broke the trackpad, a Bluetooth NVRAM key deleted by mistake).

## 2. Hardware and SMBIOS

| Parameter | Value |
|---|---|
| Laptop | Lenovo IdeaPad 3 15IML05; EC VPC2004 / IDEA2004; ACPI OEM LENOVO CB-01 |
| Processor | Intel Core i3-10110U, 2 cores / 4 threads (Comet Lake-U, 10th gen) |
| Memory | 20 GB DDR4-2667 (4 GB + 16 GB) |
| Disks | SATA SSD SSSTC CVB-8D128-HP (128 GB, boot) and 1 TB HDD WD10SPZX |
| Graphics | Intel UHD 620; `ig-platform-id 0x3EA50009` (CFL framebuffer), `device-id 0x3E9B` spoof |
| Audio | Realtek ALC 230, `layout-id 20` |
| Wi-Fi / Bluetooth | Intel Wireless-AC 9560 (CNVi); Bluetooth is the USB device `8087:0AAA` on port `HS10` |
| Trackpad | I2C HID, SYNA2BA6 (`I2C0.TPD0`) |
| Battery | 4605 mAh design, 3380 mAh now (about 73%), 104 cycles (2026-10-28) |

| | Sonoma 14 | Tahoe 26.7.2 |
|---|---|---|
| SMBIOS | `MacBookAir8,1` | `MacBookPro16,2` (13" MacBook Pro 2020, four Thunderbolt 3 ports) |
| Why | Close match for a 10th-gen U-series laptop | One of the last Intel models supported by Tahoe; it is a **T2** model, which matters for updates (section 10) |
| CPU name | Shown as "Apple M4" via RestrictEvents (`revcpuname`) | Same `revcpu` / `revcpuname` NVRAM keys |

> **Correction.** Early in the Sonoma analysis the DSDT structure suggested "Alder Lake, i7-12700". That was **wrong**: the 24 root ports and 20 Processor objects come from Lenovo's generic template; the machine is Comet Lake-U (i3-10110U).

## 3. EFI

### 3.1 Layout

| Item | Sonoma EFI (2026-10-06) | Tahoe EFI (2026-10-29) |
|---|---|---|
| OpenCore | 1.0.1 (REL-101-2024-08-05) | 1.0.8 (REL-108-2026-09-27) |
| Drivers | OpenRuntime, HfsPlus, OpenCanopy, AudioDxe, NvmExpressDxe, ResetNvramEntry; ToggleSipEntry disabled | Same set |
| SSDTs | 15 enabled (+ `SSDT-EC-USBX.aml` unused) | 16 enabled (+ `SSDT-UIAC.aml` disabled spare) |
| ACPI patches | 12 | 13 |
| Kexts | 38 config entries | 29 third-party kexts loaded (+ codeless `USBPro` and `CPUFriendDataProvider`), spares `itlwm`, `USBInjectAll` disabled |
| Size | 54 MB | 57 MB |
| Resources | OpenCanopy: Audio, Font, Image (Acidanthera), Label | Same |

### 3.2 Kexts

| Kext | Sonoma | Tahoe | Purpose |
|---|---|---|---|
| Lilu | 1.7.1 | 1.7.2 | Patching framework, loaded first |
| VirtualSMC, SMCBatteryManager, SMCProcessor, SMCSuperIO | 1.3.8 | 1.3.8 | SMC emulation, battery, sensors |
| WhateverGreen | 1.7.0 | 1.7.1 | Graphics |
| AppleALC | 1.9.6 | 1.9.8 | Audio layout |
| **SimpleHDA** | - | 1.0.0 | Audio driver for Tahoe (AppleHDA removed) |
| RestrictEvents | 1.1.6 | 1.1.6 | CPU name; `revpatch=sbvmm` on Tahoe |
| ECEnabler | 1.0.6 | 1.0.6 | EC fields (battery) |
| CpuTscSync | 1.1.2 | 1.1.2 | TSC sync |
| BrightnessKeys | 1.0.3 | 1.0.3 | Brightness keys |
| FeatureUnlock | 1.1.8 | 1.1.8 | Feature unlock |
| NVMeFix | 1.1.3 | 1.1.3 | NVMe |
| **CPUFriend** | 1.3.0 (a different build from the official release) | 1.3.0 official RELEASE (2025-07-07), run with `-cpufbeta` | CPU power profile loader (section 9) |
| **CPUFriendDataProvider** | 1.0.1 | 1.0.4 (rebuilt for `MacBookPro16,2`) | Codeless kext holding the profile data (section 9) |
| RTCMemoryFixup | 1.0.8 | 1.0.7 | RTC memory |
| BlueToolFixup | 2.6.8 | 2.7.2 | Bluetooth daemon patches (Monterey and newer) |
| IntelBluetoothFirmware | 2.5.0 | 2.5.0 (+ `0A12` personality) | Intel Bluetooth firmware upload |
| **IntelBTPatcher** | - | 2.5.0 | Intel Bluetooth HCI fixes |
| AirportItlwm | 2.3.0 | 2.4.0 (Tahoe build) | Intel Wi-Fi |
| VoodooI2C (+ VoodooInput, VoodooI2CServices, VoodooGPIO), VoodooI2CHID | 2.9.1 | 2.9.1 | I2C trackpad |
| VoodooPS2Controller, VoodooPS2Keyboard | 2.3.8 | 2.3.7 | PS/2 keyboard |
| USBPro (USB map) | 1.2 | 1.2 (+ `usb-port-number`, `Intel-BT-VendorID`) | USB port map |
| USBWakeFixup | 1.0 | 1.0 | USB wake (with `SSDT-USBW`) |
| YogaSMC | 1.5.3 | 1.5.3 | Lenovo features |
| HoRNDIS, HWPEnabler, EnergyDriver, HibernationFixup, RtWlanU, RtWlanU1827, RealtekCardReader (+ 2 helpers) | ✓ | - | Not carried over to Tahoe |

Load order on Tahoe: Lilu, VirtualSMC and plugins, WhateverGreen, AppleALC, SimpleHDA, RestrictEvents, ECEnabler, CpuTscSync, BrightnessKeys, FeatureUnlock, NVMeFix, CPUFriend, CPUFriendDataProvider, RTCMemoryFixup, BlueToolFixup, IntelBluetoothFirmware, IntelBTPatcher, AirportItlwm, VoodooInput, VoodooI2CServices, VoodooGPIO, VoodooI2C, VoodooI2CHID, VoodooPS2Controller, VoodooPS2Keyboard, USBPro, USBWakeFixup, YogaSMC. Never load `itlwm` and `AirportItlwm` together.

### 3.3 SSDTs

| SSDT | Bytes | Purpose | Sonoma | Tahoe |
|---|---:|---|:---:|:---:|
| `SSDT-AWAC` | 73 | AWAC/RTC conflict (STAS = 1: the real RTC is enabled) | ✓ | ✓ |
| `SSDT-XOSI` | 421 | `_OSI` wrapper (Darwin = Windows) | ✓ | ✓ |
| `SSDT-OSYS` | 574 | Sets OSYS (PCI1._INI); duplicates XOSI, harmless | ✓ | ✓ |
| `SSDT-PLUG` | 900 | XCPM (CPU power) | ✓ | ✓ |
| `SSDT-MCHC` | 104 | MCHC device | ✓ | ✓ |
| `SSDT-SBUS` | 190 | SBUS.BUS0 + DVL0 (SMBus) | ✓ | ✓ |
| `SSDT-DTGP` | 100 | DTGP helper method | ✓ | ✓ |
| `SSDT-MEM2` | 176 | MEM2 for the iGPU | ✓ | ✓ |
| `SSDT-PNLFCFL` | 141 | Brightness (PNLF) | ✓ | ✓ |
| `SSDT-ALS0` | 132 | Ambient light sensor (ACPI0008) | ✓ | ✓ |
| `SSDT-TPD0` | 468 | Trackpad `_CRS`/`_DSM` replacement (needs the XCRS/XTSM renames) | ✓ | ✓ |
| `SSDT-RHUB` | 110 | `_STA` for XHC.RHUB | ✓ | ✓ |
| `SSDT-USBX` | 217 | USB power properties (USBX only, no fake EC) | ✓ | ✓ |
| `SSDT-USBW` | 150 | USBW device (`_PRW = XHC._PRW`), paired with USBWakeFixup | ✓ | ✓ |
| `SSDT-VirutalNetCard` | 215 | Null Ethernet (RMNE); typo in the file name | ✓ | ✓ |
| **`SSDT-XHCID`** | 183 | xHCI `device-id` `9DED` from `_DSM` (section 6.2) | - | ✓ |

Removed during the Sonoma work: `SSDT-RCSM`, `SSDT-ECRW`, `SSDT-YVPC` (no `H_EC`), `SSDT-NO-CNVW` (did not work), `SSDT-HPET`, `SSDT-RTC` (failed to load, redundant), `SSDT-EC-USBX` (replaced by `SSDT-USBX`).

### 3.4 ACPI patches

| # | Table | Patch | Note |
|---|---|---|---|
| 1, 2, 7 | all | RTC IRQ 8, TIMR IRQ 0, IPIC IRQ 2 | |
| 3 | DSDT | `_OSI -> XOSI` | For `SSDT-XOSI` |
| 4 | DSDT | **`TPD0 _CRS -> XCRS`** | Find `20 5F 43 52 53`; `SSDT-TPD0` relies on it (see 4.3) |
| 5 | DSDT | `TPD0 _DSM -> XTSM` | |
| 6 | all | `HPET _STA -> XSTA` | No `SSDT-HPET`: HPET has no `_STA` and stays enabled (original working state) |
| 8 | DSDT | `SAT0 -> SATA` | |
| 9 | **all** | `EC0 -> EC` | The BIOS SSDT also uses `EC0_` |
| 10, 11, 12 | DSDT | `HDAS -> HDEF`, `HECI -> IMEI`, `PNLF -> XNLF` | |
| 13 | DSDT | **`XHC _DSM -> XDSM`** (Tahoe) | For `SSDT-XHCID`, see 6.2 |

### 3.5 config.plist

| Section | Sonoma | Tahoe |
|---|---|---|
| Kernel > Quirks | AppleCpuPmCfgLock, AppleXcpmCfgLock, CustomSMBIOSGuid, DisableIoMapper, DisableLinkeditJettison, PanicNoKextDump, PowerTimeoutKernelPanic | Same |
| Kernel > Patch | AppleRTC: disable RTC wake scheduling; IOAHCIBlockStorage: TRIM (3) | Same |
| Booter > Quirks | AvoidRuntimeDefrag, EnableSafeModeSlide, EnableWriteUnprotector, FixupAppleEfiImages, ProvideCustomSlide, SetupVirtualMap, SyncRuntimePermissions | Same |
| UEFI | ConnectDrivers, ReleaseUsbOwnership, RequestBootVarRouting, EnableVectorAcceleration; audio on Pci 1F,3 | Same |
| Misc | PickerMode External (OpenCanopy), Timeout 59, SecureBootModel Disabled, Vault Optional, ScanPolicy 0, HibernateMode None | Same |
| NVRAM boot-args | `-v keepsyms=1 debug=0x100 swd_panic=1` | `-v keepsyms=1 debug=0x100 swd_panic=1 -ibtcompatbeta -btlfxallowanyaddr revpatch=sbvmm -cpufbeta` |
| NVRAM other | csr-active-config 0 (SIP on); `revcpu`, `revcpuname` | Same + `bluetoothExternalDongleFailed`, `bluetoothInternalControllerInfo` (section 7) |
| PlatformInfo | `MacBookAir8,1`, UpdateSMBIOSMode Custom | `MacBookPro16,2`, UpdateSMBIOSMode Custom |
| DeviceProperties | XHC (`device-id` spoof `A2AF`), CNVi, SerialIO, IMEI, SATA, HDEF (`layout-id 20`), iGPU | Same, but the XHC `device-id` was **removed** (it comes from `SSDT-XHCID`) |

Serial number, MLB, SmUUID and ROM are present in both configs and are not shown here.

## 4. ACPI and DSDT

### 4.1 DSDT

| Parameter | Value |
|---|---|
| Signature / OEM / Table ID | DSDT / LENOVO / CB-01, OEM revision 1, ACPI revision 2 |
| Size / checksum | 245,154 bytes (245,617 in the live system after OpenCore changes) / correct |
| Compiler | INTL 20200925; recompiling gives AML byte-identical to the original |
| Objects | 22,587 opcodes, 4,021 named objects; 206 Device, 1,530 Method, 114 OpRegion, 46 PowerResource, 20 Processor, 6 Mutex |
| iasl result | 0 errors, 187 warnings, 313 remarks (typical of vendor code, left untouched) |

**Structure**

- **PCI0:** 3 PEG, GFX0, IPU0, 24 root ports, SATA (6 ports + NVM1-3), XHC (14 USB2 + 10 USB3), XDCI, HDEF (+ SoundWire), GLAN, SBUS, CNVW (CNVi Wi-Fi), PUFS/PEMC/PSDC, ISHD, IMEI.
- **LPCB:** EC (PNP0C09; inside: BAT0, VPC0 = VPC2004, ITSD), HPET, RTC, IPIC, TIMR, PS2K, PS2M. **SerialIO:** I2C0-5, SPI0-2, UA00-02; I2C0.TPD0 (trackpad, SYNA2BA6).
- **Lenovo WMI:** WMI4, WMIL, WMIU, WMTF, HKDV. Reference-code leftovers: DOCK (SADDLESTRING), WFDE/WFTE (SampleDev/TestDev), TPD0 HID "XXXX0000".
- **Power:** PEPD (S0ix), AWAC, ADP0, LID0, PWRB, SLPB, Thunderbolt code (`_GPE.XTBT`).
- **XHC:** has its own `_DSM` that starts with `ADBG ("XHCI _DSM")`; this is why `SSDT-XHCID` renames it instead of adding a second `_DSM` (section 6.2).

| Topic | Details |
|---|---|
| `_Sx` | S0 {0,0,0,0}, S1 {1,0,0,0}, **S3 {5,0,0,0}**, S4 {6,0,0,0}, S5 {7,0,0,0} |
| `_PTS` / `_WAK` | `_PTS`: Thunderbolt and S3/S4 GPIO preparation. `_WAK` (2,046 bytes): writes the OS type to the EC, bus-checks the root ports, restores LID state. OSYS is 1000, or 0x7DF via XOSI |
| GPE handlers | `_L61` PCIe hot-plug (not a wake source); `_L69` / `_L6F` / `_L26` PCIe PME / RTD3-PEG GPIO / WLAN GPIO wake; `_L62` / `_L66` / `_L72` thermal-HWP / graphics SCI / RTC alarm |
| **GPE 0x6D** | No handler; GLAN, XHC, XDCI, HDEF and CNVW all call `GPRW (0x6D, 0x03)` in `_PRW`, the OS manages it |
| USB (`_UPC`) | External: HSP1, HSP3, HSP4, SSP1, SSP2. Internal: HSP2, HSP5, HSP6, **HSP7 (camera)**, HS10 (Bluetooth). Not connectable: HSP8, HSP9, SSP3-6. Hidden: HS11-14, SS07-10 |
| iasl warnings (187) | 3115 x92, 3124 x31, 3107 x25, 3168 x20, 3073 x10, other x9 |

### 4.2 ACPI errors found and fixed (Sonoma)

| Table / error | Cause | Fix |
|---|---|---|
| BIOS SSDT: `EC0_` (AE_NOT_FOUND) | The `EC0 -> EC` patch applied only to the DSDT, while the BIOS SSDT still uses `EC0_` | Patch applied to all tables. **Worked** |
| `SSDT-EC-USBX`: `EC__` (AE_ALREADY_EXISTS) | It created a fake EC while a real EC exists; the whole table was dropped and USBX was missing | `SSDT-USBX` (USBX only). **Worked**: 2100 / 5100 |
| `SSDT-RTC`: `RTC_` (AE_ALREADY_EXISTS) | `RTC_` already exists; `SSDT-AWAC` (STAS = 1) enables it | `SSDT-RTC` removed |
| `SSDT-HPET`: `_CRS` (AE_ALREADY_EXISTS) | Patch #4 acts on `TPD0 _CRS`, not HPET (its old comment was wrong) | `SSDT-HPET` removed |
| `H_EC` (`RCSM`, `ECRW`, `YVPC`) | There is no `H_EC` in the DSDT (the EC is `LPCB.EC`) | Removed; YogaSMC works through VPC0 |

### 4.3 Current state and one mistake worth knowing

- **Sonoma (boot of 2026-10-06 03:38):** no table failed to load, no `Namespace lookup failure`.
- **Tahoe 26.7.2:** no table fails to load either (16 SSDTs). One boot-time runtime exception remains: `\_SB.PCI0.LPCB.EC.BAT0._STA` fails with `AE_AML_UNINITIALIZED_LOCAL`, together with `No device handle for EC_ID ...H_EC`. The battery reads correctly afterwards; the cause was not investigated.
- **Harmless messages:** "Unsupported module-level executable opcode 0x70" (2016 ACPICA in macOS), "no ECDT", "cannot translate ACPI object 14".

> **The HPET mistake.** An attempt to "fix" the HPET patch with a Base-scoped rename **broke the trackpad**: patch #4 actually renames `TPD0 _CRS`, and `SSDT-TPD0` depends on it. With the new patch `SSDT-TPD0` failed to load (`VoodooI2CHIDDevice = 0`); restoring the original patch (Find `20 5F 43 52 53`) brought the trackpad back.

**How the DSDT was verified:** at each stage a live DSDT dump (MaciASL) was compared with the original AML semantically (both re-decompiled with the same iasl, display-only differences filtered out). Calls to external SSDT methods are written invalidly in the dump, so they were fixed in a temporary copy that then compiled with 0 errors.

## 5. Sleep and wake

### 5.1 Instant wake on Sonoma: diagnosis

**Symptom:** the Mac woke up by itself 8-9 seconds after going to sleep; all 11 events in `pmset -g log` said `due to XDCI USBW/User`, `DriverReason:XHC`.

| Time | State | Sleep | Result |
|---|---|---|---|
| 00:29 | Original DSDT (no patch) | 8 s | Instant wake |
| 00:52 | Selective GPRW patch (GLAN, XDCI, HDEF, CNVW), receiver connected | 8 s | Instant wake |
| 00:57 | Same, receiver unplugged | 8 s | Instant wake |
| 00:59 | Same, receiver unplugged, Bluetooth off | 9 s | Instant wake |
| 01:10 | Bluetooth kexts not loaded | 8 s | Instant wake |
| 01:20 | **USB map: HS07, HS10 = 255** (+ patch, Bluetooth kext on) | 63 s | **Stable** |
| 01:48 | Same | 45 s | Stable |
| 02:08 | **USB map fixed, no patches** | 28 s | **Stable** |
| 02:37 | Same | 21 s | Stable |
| 03:42 | Final configuration (USB map + USBX, no ACPI errors) | 50 s | **Stable** |

- The only change between the last failing test (01:10) and the first stable one (01:20) was the USB map. The no-patch tests are stable too, so the GPRW patches were removed; USB wake stays available (`SSDT-USBW` + `USBWakeFixup`).
- Likely mechanism: with every port marked external, macOS applied the external-device wake policy to the internal camera and Bluetooth (`USBExternalDevice` sleep factor).
- The label "XDCI USBW" shows the wake path, not the culprit: XDCI is only an ACPI entry on GPE 0x6D and USBW is the device created by `SSDT-USBW`.
- A full `SSDT-GPRW` (GPE 0x6D off) also stopped the wake, but it disables USB wake as well.
- USB keyboard wake and long sleep were then confirmed by the user.

### 5.2 Tahoe tests

| Test | Duration | Wake time | Result |
|---|---|---|---|
| Menu sleep, 26.7 | 24 s | 2.97 s | ✅ |
| Lid-close sleep, 26.7 | 31 min | 2.94 s | ✅ |
| Menu sleep, 26.7.2 | 8 min | 2.98 s | ✅ |
| Lid-close sleep, 26.7.2 | 64 min | 2.99 s | ✅, Bluetooth dropped for ~35 s and recovered (section 7.4) |
| Lid-close sleep, 26.7.2 + CPUFriend | 12 min | not measured | ✅, CPUFriend profile kept (`epp 8`); Bluetooth dropped for ~16 s and recovered (section 7.4) |

Wake reason in every case: `PWRB XDCI USBW/User` (or `.../UserActivity Assertion`), i.e. power button or key press; no spontaneous wake. Power settings: `hibernatemode 0`, `standby 0`, `powernap 0` (`sudo pmset -a hibernatemode 0 standby 0 powernap 0`); the `disksleep` warning that `pmset` prints on battery is harmless.

## 6. USB

### 6.1 USB map (both systems)

`USBPro.kext` is a codeless map (`AppleUSBHostMergeProperties`) for `XHC` with 15 ports (`HS01-HS12`, `SS01-SS03`). The internal camera (`HS07`) and Bluetooth (`HS10`) are `UsbConnector = 255`; the others are `3`. `SSDT-USBX` sets the USB power properties (`kUSBSleepPortCurrentLimit = 2100`, `kUSBSleepPowerSupply = 5100`).

### 6.2 Three Tahoe-specific causes

| # | Symptom on Tahoe | Root cause | Fix |
|---|---|---|---|
| P1 | No USB ports; System Information shows only the controller | Sonoma used the `device-id` spoof `A2AF` (Skylake class). Tahoe removed the legacy Intel xHCI classes (SPT, LPT, PPT, WPT ...); only `AppleIntelCNLUSBXHCI` (`9DED`, `A36D`, `06ED`) and `AppleIntelICLUSBXHCI` remain | `device-id` = `9DED` |
| P2 | OpenCore picker turns into a black text menu | `9DED` placed in `DeviceProperties`; the only difference between two otherwise identical EFIs | Give it from ACPI (`SSDT-XHCID`) and keep it out of `DeviceProperties` |
| P3 | Driver loaded, ports still dead | Apple renamed the per-port key `port` to `usb-port-number` (compare `AppleUSBHostPlatformProperties` in Tahoe and Sonoma) | Both keys on all 15 ports |

**SSDT-XHCID.** The original `XHC._DSM` is renamed to `XDSM` and replaced by a `_DSM` that returns the `device-id`:

```asl
DefinitionBlock ("", "SSDT", 2, "hack", "XHCID", 0x00000000)
{
    External (_SB_.PCI0.XHC_, DeviceObj)
    External (_SB_.PCI0.XHC_.XDSM, MethodObj)    // 4 Arguments

    Method (\_SB.PCI0.XHC._DSM, 4, NotSerialized)
    {
        If ((Arg0 == ToUUID ("a0b5b7c6-1318-441c-b0c9-fe695eaf949b")))
        {
            If ((Arg2 == Zero))
            {
                Return (Buffer (One) { 0x03 })
            }

            Return (Package (0x02)
            {
                "device-id",
                Buffer (0x04) { 0xED, 0x9D, 0x00, 0x00 }
            })
        }

        Return (\_SB.PCI0.XHC.XDSM (Arg0, Arg1, Arg2, Arg3))
    }
}
```

| ACPI > Patch field | Value |
|---|---|
| Comment | `XHC: _DSM to XDSM (needed by SSDT-XHCID)` |
| Find | `5F 44 53 4D 0C 41 44 42 47 0D 58 48 43 49 20 5F 44 53 4D 00` |
| Replace | `58 44 53 4D 0C 41 44 42 47 0D 58 48 43 49 20 5F 44 53 4D 00` |
| Count / TableSignature | `1` / `DSDT` |

> The pattern includes the `ADBG ("XHCI _DSM")` call of this laptop's DSDT, so it matches exactly one place. On another BIOS, check your own DSDT first.

**Port keys in `USBPro.kext`** (all 15 ports, the map keeps working on Sonoma):

```xml
<key>HS01</key>
<dict>
    <key>UsbConnector</key><integer>3</integer>
    <key>port</key><data>AQAAAA==</data>
    <key>usb-port-number</key><data>AQAAAA==</data>
</dict>
```

## 7. Bluetooth

### 7.1 Sonoma

`IntelBluetoothFirmware` 2.5.0 + `BlueToolFixup` 2.6.8 are enough. Sonoma's `bluetoothd` accepts any USB Bluetooth vendor and uses the generic `THIRD_PARTY_DONGLE` path.

### 7.2 Why Intel fails on Tahoe

Symptoms: the switch turns off; the controller shows `BCM_4350C2`, firmware `v0 c0`, address `NULL`; `bluetoothd` restarts every ~10 s with `Transport layer initialization failed`, later `VSC Read Verbose Config Version Info failed, STATUS 712` and `HCI initialization failed`.

Findings from Tahoe's `bluetoothd` binary, compared with the working Sonoma logs:

| Finding | Detail |
|---|---|
| Internal controller detection | Tahoe compares `USB Product Name` with `Bluetooth USB Host Controller`. The Intel device has no product string. A name merge only moves the failure to the HCI stage |
| Dongle allow-list | Tahoe accepts an external dongle only for vendor `0x0A12` (CSR), `0x0A5C` (Broadcom), `0x0B05:0x190E` (ASUS) and a few MediaTek/Azurewave IDs. Intel `0x8087` is not listed, so it is treated as an internal Broadcom chip and receives Broadcom vendor commands it cannot answer |
| Name check | A device already named `Bluetooth USB Host Controller` is skipped by the dongle check, so the name merge must **not** be combined with the fix below |

### 7.3 Fix

1. `USBPro.kext`: an extra personality rewrites `idVendor` of `8087:0AAA` to `0x0A12` (CSR):

   ```xml
   <key>Intel-BT-VendorID</key>
   <dict>
       <key>CFBundleIdentifier</key><string>com.apple.driver.AppleUSBHostMergeProperties</string>
       <key>IOClass</key><string>AppleUSBHostMergeProperties</string>
       <key>IOProviderClass</key><string>IOUSBHostDevice</string>
       <key>idVendor</key><integer>32903</integer>
       <key>idProductArray</key><array><integer>2730</integer></array>
       <key>IOProviderMergeProperties</key>
       <dict>
           <key>idVendor</key><integer>2578</integer>
       </dict>
   </dict>
   ```

   (`32903` = `0x8087`, `2730` = `0x0AAA`, `2578` = `0x0A12`.)

2. `IntelBluetoothFirmware.kext/Contents/Info.plist`: copy the personality `IntelBluetoothFirmware_9560` and set `idVendor` to `2578`, so the firmware kext still matches after the rewrite.
3. NVRAM, GUID `7C436110-AB2A-4BBB-A880-FE41995C9F82`, also listed under **Delete** so both are rewritten every boot:

   | Key | Type | Value |
   |---|---|---|
   | `bluetoothExternalDongleFailed` | Data | `00` (`AA==`) |
   | `bluetoothInternalControllerInfo` | Data | 14 zero bytes (`AAAAAAAAAAAAAAAAAAA=`) |

   Tahoe skips the dongle path if `bluetoothExternalDongleFailed` is `01`.
4. Kexts: `BlueToolFixup` 2.7.2, `IntelBluetoothFirmware` 2.5.0, `IntelBTPatcher` 2.5.0, all after `Lilu`; do not load `IntelBluetoothInjector`.
5. boot-args: `-ibtcompatbeta -btlfxallowanyaddr`.

Result: `bluetoothd` logs `Found external Bluetooth dongle -- setting chipset to THIRD_PARTY_DONGLE` with vendor `0x0A12`, the controller is `PoweredOn` and finds nearby devices. Pairing and Bluetooth audio were not tested yet.

Attempts that did **not** help on their own: a newer firmware kext alone, the NVRAM keys alone (at first `bluetoothInternalControllerInfo` was deleted by mistake; the README of BlueToolFixup requires both keys), IntelBTPatcher with `-ibtcompatbeta`, `-ibtcompatdbg`, a full power-off cold boot, the name merge.

### 7.4 After sleep

Normal wake messages (also before 26.7.2): `HCI expected event` for `SET_EVT_MASK` and `LE_SET_RANDOM_ADDRESS`; the controller returns to `PoweredOn`. After the 64-minute lid sleep, `bluetoothd` hit USB pipe errors (`kIOReturnNotResponding`) and restarted twice: the first start came up before the USB device returned (`Transport layer initialization failed`), the second recovered.

The same happened after the 12 min lid sleep with CPUFriend (2026-10-29): the Bluetooth USB pipes returned `kIOReturnNotResponding` at wake (11:57:31), `Transport layer initialization failed` followed 16 s later (11:57:47) and the restarted `bluetoothd` powered the chip on within a second (11:57:48); the state is `On`, `THIRD_PARTY_DONGLE` again. So it is repeatable (two of two long lid sleeps, ~16-35 s), it self-recovers and CPUFriend is not involved (the 64 min case predates it). The Intel kexts log nothing in release builds, so the cause was not pinned down. Accepted as is.

Untested idea: `bluetoothd` may keep a stale USB handle for the whole timeout. If so, restarting it right after wake would shorten the gap:

```bash
sudo pkill bluetoothd
```

## 8. Audio, Wi-Fi, graphics and input

| Area | Sonoma | Tahoe | Notes |
|---|---|---|---|
| Audio | AppleALC (`layout-id 20`) with AppleHDA | AppleALC + **SimpleHDA 1.0.0** | Apple removed AppleHDA in Tahoe; system sounds play through the built-in speakers |
| Wi-Fi | AirportItlwm 2.3.0 | AirportItlwm 2.4.0 (Tahoe build, MinKernel 25.0.0) | `itlwm.kext` kept as a disabled spare |
| Graphics | WhateverGreen, CFL framebuffer, backlight patch | Same; Metal 3, 3 framebuffers | `device-id 0x3E9B`, `ig-platform-id 0x3EA50009` |
| Trackpad | VoodooI2C + VoodooI2CHID (`SSDT-TPD0`) | Same | Multitouch works |
| Keyboard, Fn | VoodooPS2, BrightnessKeys, YogaSMC | Same | |

## 9. CPU power management (CPUFriend)

### 9.1 What it does

`CPUFriend.kext` (a Lilu plugin) replaces the power profile that Apple's `X86PlatformPlugin` loads for the Mac's board-id. The profile data comes from the codeless `CPUFriendDataProvider.kext` (property `cf-frequency-data`, a complete plist). Inside each frequency vector there are named fields near the end; two of them are tuned here:

| Field | Meaning | Apple default | Used here |
|---|---|:---:|:---:|
| `epp` | HWP energy-performance preference; lower favours performance, `128` is balanced, `255` saves power | 128 | **8** |
| `perf-bias` | Performance bias | 5 (`MacBookAir8,1`) / 7 (`MacBookPro16,2`) | **7** |

### 9.2 The data on both systems

The Sonoma data differed from Apple's own `MacBookAir8,1` plist in exactly these two fields (checked by comparing the vectors word by word). On Tahoe the SMBIOS is `MacBookPro16,2`, so the plugin uses another base plist; the data was rebuilt from that one.

| | Sonoma 14 | Tahoe 26.7.2 |
|---|---|---|
| Base plist (X86PlatformPlugin resources) | `Mac-827FAC58A8FDFA22` (`MacBookAir8,1`) | `Mac-5F9802EFE386AA28` (`MacBookPro16,2`) |
| Frequency vectors | 1 | 2 |
| `Frequencies` map | `3600 -> 0` | `3800 -> 1`, `4100 -> 1` (the i3-10110U turbo is 4.1 GHz) |
| `CPUFloor` | 800 | 1000 |
| `ratioratelimit` | 3000000 | 100000 |
| `epp` / `perf-bias` | 8 / 7 (Apple: 128 / 5) | 8 / 7 (Apple: 128 / 7) |
| Provider version | 1.0.1 | 1.0.4 |

Board-id of the running Mac: `ioreg -rd1 -c IOPlatformExpertDevice | grep board-id`.

### 9.3 Rebuilding the data for another SMBIOS

```python
import plistlib, struct
R = '/System/Library/Extensions/IOPlatformPluginFamily.kext/Contents/PlugIns/X86PlatformPlugin.kext/Contents/Resources/'
base = plistlib.load(open(R + 'Mac-5F9802EFE386AA28.plist', 'rb'))   # board-id of your SMBIOS
want = {b'epp': 8, b'perf-bias': 7}
vectors = []
for v in base['IOPlatformPowerProfile']['FrequencyVectors']:
    b = bytearray(v)
    for off in range(7000, len(b) - 24, 4):           # named fields sit at the end of the vector
        name = bytes(b[off:off + 20]).split(b'\0')[0]
        if name in want and not any(b[off + len(name):off + 20]):
            struct.pack_into('<I', b, off + 20, want[name])   # the value follows the 20-byte name
    vectors.append(bytes(b))
base['IOPlatformPowerProfile']['FrequencyVectors'] = vectors
data = plistlib.dumps(base, fmt=plistlib.FMT_XML)
# write `data` into CPUFriendDataProvider.kext/Contents/Info.plist:
#   IOKitPersonalities > CPUFriendDataProvider > cf-frequency-data
```

Each named record is 36 bytes (20-byte name, 4-byte value, padding). In the `MacBookPro16,2` vector (7668 bytes) `epp` is at byte 7148 and `perf-bias` at 7184. `CFBundleVersion` of the provider must be purely numeric (`1.0.4`).

In `config.plist` (Kernel > Add) both kexts come right after `NVMeFix` (after `Lilu`): `CPUFriend.kext` with `Contents/MacOS/CPUFriend`, then `CPUFriendDataProvider.kext` with an empty `ExecutablePath`.

### 9.4 Problems found on Tahoe

| # | Symptom | Cause | Fix |
|---|---|---|---|
| C1 | `kmutil showloaded` lists CPUFriend, `cf-frequency-data` is on all four CPU nodes in `ioreg`, but the plugin still reports Apple's profile (`epp 128`, `ratio_change_ratelimit_ns` unchanged) | The Sonoma-era copy (labelled 1.3.0, Xcode 14.2 build, file dated 2025-01-21) did not apply the data on Tahoe without help | Boot-arg `-cpufbeta`. The official 1.3.0 build (below) was only tested with this flag |
| C2 | No `cf-frequency-data` and no `CPUFriendDataProvider` in `ioreg` at all | I had written the provider version as `1.0.3-test`; kext versions must be numeric | `1.0.3`, later `1.0.4` |
| C3 | Everything looks fine in `ioreg` but nothing changed | The data being present on the CPU nodes does **not** prove the plugin uses it | Check the plugin's live vector, or use a marker value (9.5) |

### 9.5 How it was verified

A marker was set in the data: `ratioratelimit` 100000 -> 200000. The plugin exports it as `machdep.xcpm.ratio_change_ratelimit_ns`, and the vectors it really uses are visible in `ioreg`.

```bash
sysctl machdep.xcpm.ratio_change_ratelimit_ns
```

```bash
kmutil showloaded | grep -i cpufriend
```

```bash
ioreg -rc X86PlatformPlugin -l -w0 | grep -m1 IOPlatformPowerProfile
```

(Decode the `FrequencyVectors` hex and read `epp` / `perf-bias` at the offsets above.)

| Test | CPUFriend | Boot-arg | Provider in `ioreg` | `ratio_change_ratelimit_ns` | Live `epp` | Result |
|---|---|---|---|---|---|---|
| 1 | Sonoma copy | - | yes (`epp 8` data) | 100000 | 128 | Not applied |
| 2 | Sonoma copy | - | **no** | 100000 | - | Provider rejected (version string) |
| 3 | Sonoma copy | - | yes (marker 200000) | 100000 | 128 | Not applied |
| 4 | Sonoma copy | `-cpufbeta` | yes | **200000** | **8** | Applied |
| 5 | Official 1.3.0 RELEASE | `-cpufbeta` | yes | 100000 (marker removed, as designed) | **8** | Applied |

After a 12 min lid sleep the live vector still had `epp 8` and `perf-bias 7`; USB, Bluetooth, Wi-Fi and the power settings (`hibernatemode 0`, `standby 0`, `powernap 0`) were unchanged.

### 9.6 The official release used in the final EFI

| Item | Value |
|---|---|
| Source | `github.com/acidanthera/CPUFriend`, release 1.3.0 (2025-07-07), asset `CPUFriend-1.3.0-RELEASE.zip` (78,716 bytes) |
| SHA-256 of the zip | `37645d960f0b3c958cfd0a8a041160532267ec535c4979897123df89c7dbdcde` |
| SHA-256 of `Contents/MacOS/CPUFriend` | official `36c3897684642942bd0dc60f20ecdf002aa375ce1788fa0cefc5e7282418dd76`; the earlier copy was `cc0a355776d121fa6a782bb0a728c6e070a35ad1fe5a680d5665055b8f41123d` |
| Build | official: Xcode 15.2 (15C500b); earlier copy: Xcode 14.2. Supports `-cpufoff`, `-cpufdbg`, `-cpufbeta` |
| Code signature | none (normal for Acidanthera kexts) |

The release notes say 1.3.0 adds constants for macOS 26, yet the flag was kept because a boot without it was never tested with the official build.

### 9.7 Not done / not verified

- `HWPEnabler` and `EnergyDriver` (used on Sonoma) were not carried over. Nothing broke, but whether HWP is active was not measured (no MSR access). If `epp` seems to have no effect, add `HWPEnabler` first.
- Battery life and fan behaviour with `epp 8` were not measured. To trade speed for battery, change `epp` (for example 64 or 128), rebuild the provider and bump its version.
- Whether the official build works on Tahoe without `-cpufbeta` was not tested.
- No Geekbench run on Tahoe yet, so the Sonoma scores in section 14 are not comparable.
- After every macOS point update, repeat the `sysctl` / `kmutil` check above.

## 10. macOS updates

On Tahoe, `softwareupdate -l` and System Settings offered nothing although 26.7.2 existed (it was visible in the full-installer list). `MacBookPro16,2` is a **T2** model, and with a T2 SMBIOS on non-T2 hardware macOS does not offer updates.

| Item | Detail |
|---|---|
| Fix | `RestrictEvents.kext` (already loaded) + boot-arg `revpatch=sbvmm` |
| Result | 26.7.2 (25G332, 1.37 GB) appeared and installed in about 20 minutes; `kern.hv_vmm_present` stays `0` |
| After the update | OpenCore, boot-args and Bluetooth NVRAM kept; 28 third-party kexts loaded; USB, Bluetooth (`THIRD_PARTY_DONGLE`), Wi-Fi, audio and graphics all checked |
| Space | About 1.4 GB download; 30 GB free was more than enough |
| Advice | Turn **automatic installation of macOS updates off** and update by hand after backing up the EFI: the Bluetooth fix depends on `bluetoothd` internals. Tahoe is the last macOS for Intel Macs (macOS 27 is Apple silicon only) |

## 11. DRM media (Apple Music and Apple TV)

| Item | Detail |
|---|---|
| Symptom | Apple Music is silent or stalls; Apple TV plays trailers only; the Safari web players fail too. System sounds play, so the audio path is fine |
| Logs (Tahoe) | `Key request data generation FAILED because we were unable to generate a key (err = -15842)` and `Request timed out` in Music (279 times in two hours) and TV (10 times). The HLS stream itself arrives |
| Cause | Dortania: "DRM is broken for iGPU-only systems" (the iGPU management engine lacks Apple's certificate). No verified fix for Intel UHD on Tahoe |
| Ruled out | `revpatch=sbvmm` (Apple Music never worked on this laptop), the AdGuard network filter (paused, no change), Lossless audio (turned off, no change) |
| Options | Chrome or Firefox use Widevine instead of macOS FairPlay for the web players (not tested here; Apple TV+ on the web may be limited to 720p), another device, or another service |

Do not change the SMBIOS to chase DRM: it would break the USB and Bluetooth fixes and DRM would still not be guaranteed.

## 12. Xcode and developer tools

Xcode 26.6 requires macOS Tahoe 26.2 or later. On first launch it aborted with `You have not agreed to the Xcode license agreements` (the active developer directory was still the Command Line Tools). Fix:

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

```bash
sudo xcodebuild -license accept
```

```bash
sudo xcodebuild -runFirstLaunch
```

SDKs afterwards: macOS 26.5, iOS 26.5, iOS Simulator 26.5, DriverKit 25.5. To check the SDK at any time: `xcrun --show-sdk-version` and `xcodebuild -showsdks`.

## 13. Open items

| Level | Item | Recommendation |
|---|---|---|
| ⚠️ | Automatic macOS updates were on | Turn them off; update by hand after backing up the EFI |
| ⚠️ | Tahoe point updates | Re-test USB and Bluetooth after every update (the fix depends on `bluetoothd` internals and USB property names) |
| ⚠️ | Privacy | `config.plist` contains serial, MLB, SmUUID and ROM: never share the EFI folder |
| Watch | Bluetooth drop after lid sleep | Seen after 64 min (~35 s) and 12 min (~16 s); self-recovers, accepted. Untested idea: `sudo pkill bluetoothd` after wake (section 7.4) |
| Info | Boot-time ACPI exception in `BAT0._STA` | Battery works; investigate only if battery readings misbehave |
| Info | Gatekeeper | `spctl` reports assessments disabled; check `spctl --status` if that is not intended |
| Not tested | Bluetooth pairing and audio, battery life and CPU frequency under load with `epp 8` | The profile is applied (section 9); raise `epp` if the battery drains too fast |
| Optional | `-cpufbeta` | Needed with the earlier build; not tested without it on the official 1.3.0 |
| Cleanup | Debug boot-args `-v`, `debug=0x100`, AppleDebug/ApplePanic | Remove once stable; keep `keepsyms=1` |
| Optional | Patch #6 `HPET _STA -> XSTA` without `SSDT-HPET` | Leave as is (original working state) |
| Optional | `SSDT-OSYS` and `SSDT-XOSI` both set OSYS; "Virutal" file name typo | Harmless |
| Optional | `AppleCpuPmCfgLock` and `AppleXcpmCfgLock` both on | Not needed if CFG Lock is disabled in the BIOS |

Resolved by the Tahoe EFI (Sonoma leftovers): `SSDT-EC-USBX.aml` is gone, `HibernationFixup` and the unused Realtek kexts are no longer loaded.

## 14. Benchmarks

| Information | Result | ID | Operating system | Model ID |
| --- | --- | --- | --- | --- |
| CPU Single-Core Score | 1250 | [ID 10211314](https://browser.geekbench.com/v6/cpu/10211314) | `macOS Sonoma` | `MacBookAir8,1` |
| CPU Multi-Core Score | 2065 | [ID 10211314](https://browser.geekbench.com/v6/cpu/10211314) | `macOS Sonoma` | `MacBookAir8,1` |
| iGPU OpenCL Score | 3309 | [ID 3577045](https://browser.geekbench.com/v6/compute/3577045) | `macOS Sonoma` | `MacBookAir8,1` |
| iGPU Metal Score | 4602 | [ID 3577034](https://browser.geekbench.com/v6/compute/3577034) | `macOS Sonoma` | `MacBookAir8,1` |

Tahoe benchmarks have not been run yet.

## 15. Change log

| Date | System | Change | Status |
|---|---|---|---|
| 2026-10-06 | Sonoma | GPRW -> XPRW + `SSDT-GPRW` (full wake disable) | tested, removed |
| 2026-10-06 | Sonoma | 4 Base patches (GPRW -> NPRW) + `SSDT-GPRW-USB` | tested, removed |
| 2026-10-06 | Sonoma | `_PTS -> ZPTS` patch (landed in the wrong place) | disabled |
| 2026-10-06 | Sonoma | Bluetooth kexts disabled for a test | reverted |
| 2026-10-06 | Sonoma | **USB map: HS07, HS10 `UsbConnector` 3 -> 255** | **main fix, kept** |
| 2026-10-06 | Sonoma | `SSDT-RCSM`, `YVPC`, `ECRW`, `NO-CNVW` removed; comments fixed | applied |
| 2026-10-06 | Sonoma | `EC0 -> EC` patch for all tables | applied, worked |
| 2026-10-06 | Sonoma | `SSDT-USBX` instead of `SSDT-EC-USBX` | applied, worked |
| 2026-10-06 | Sonoma | HPET Base patch (broke the trackpad) | reverted |
| 2026-10-06 | Sonoma | Original `TPD0 _CRS` patch restored; `SSDT-HPET` and `SSDT-RTC` removed | **applied, trackpad recovered** |
| 2026-10-26 | Tahoe | Tahoe EFI: OpenCore 1.0.8, `MacBookPro16,2`, SimpleHDA, AirportItlwm Tahoe build | applied |
| 2026-10-27 | Tahoe | xHCI `9DED` via `SSDT-XHCID` + `XHC _DSM -> XDSM`; `usb-port-number` | **USB works** |
| 2026-10-27 | Tahoe | Intel Bluetooth vendor-ID rewrite to `0x0A12`, NVRAM keys, IntelBTPatcher | **Bluetooth works** |
| 2026-10-28 | Tahoe | `revpatch=sbvmm`; update to 26.7.2 | **applied, update installed** |
| 2026-10-28 | Tahoe | `hibernatemode 0`, `standby 0`, `powernap 0` | applied |
| 2026-10-29 | Tahoe | CPUFriend profile rebuilt for `MacBookPro16,2` (`epp 8`, `perf-bias 7`), marker test, `-cpufbeta` | applied |
| 2026-10-29 | Tahoe | Official CPUFriend 1.3.0 RELEASE installed, marker removed | **applied, profile active** |
| 2026-10-29 | Tahoe | Sleep tests up to 64 min, Xcode 26.6 setup, DRM limitation documented | documented |

Sonoma backups were kept on the EFI partition (`config-backup-20261006-0043`, `-bt-test-0106`, `-before-usbmap-0112`, `-before-nprw-off-0141`, `USBPro Info.plist.bak-20261006-0112`).

---

### Credits:

- [Apple](https://www.apple.com) for macOS.
- [Acidanthera](https://github.com/acidanthera) for OpenCorePkg and most of the kexts (Lilu, WhateverGreen, VirtualSMC, AppleALC, RestrictEvents, CPUFriend, BrcmPatchRAM / BlueToolFixup).
- [PMheart](https://github.com/PMheart) for the original CPUFriend.
- [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for the OpenCore install and post-install guides.
- [OpenIntelWireless](https://github.com/OpenIntelWireless) for itlwm / AirportItlwm and IntelBluetoothFirmware / IntelBTPatcher.
- [laobamac](https://github.com/laobamac) for SimpleHDA (macOS 26 audio) and the Tahoe AirportItlwm build.
- [goodwin](https://github.com/goodwin) for ALCPlugFix.
- [RehabMan](https://github.com/RehabMan), [Steve Zheng](https://github.com/stevezhengshiqi), [Sniki](https://github.com/Sniki), [daliansky](https://github.com/daliansky) and [Moh_Ameen](https://github.com/ameenjuz) for patches.

### Disclaimer:

⚠️ Highly Recommend you to build the EFI for your device on your own and only use this for reference even though you have the same device as mine since, when something fails you will be on your own.

⚠️ If you want to report or rasie an issue, you must mention your device details in it. And give a detailed information about your issue(images or videos are encouraged)

⚠️ This is a technical log for one laptop, not a guarantee for other hardware or BIOS versions. The Tahoe fixes depend on macOS internals and may break with a point update.

### You can contact me through:

[![](https://img.shields.io/badge/iCloud-nusratov.sobirjon@icloud.com-informational?style=flat&logo=apple&logoColor=white&color=cbcdcc)](mailto:nusratov.sobirjon@icloud.com)[![](https://img.shields.io/badge/Telegram-@Sobirjon_Nusratov-informational?style=flat&logo=telegram&logoColor=white&color=89e2ff)](https://t.me/Sobirjon_Nusratov)[![](https://img.shields.io/badge/Facebook-Nusratov_Sobirjon-informational?style=flat&logo=facebook&logoColor=white&color=3a4dc9)](https://www.facebook.com/Sobirjon.Nusratov)
