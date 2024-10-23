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


### Hardware specifications:

| Type         | Spec                                        | Status      |
| :----------- | :------------------------------------------ | :---------- |
| Laptop       | `Lenovo IdeaPad 3 15IML05 U1`               | Working     |
| BIOS Version | `LENOVO DXCN39WW (10/13/2021) `             | Working     | 
| CPU          | `DualCore Intel Core i3-10110U  `           | Working     |
| Chipset      | `Intel Comet Point-LP, Intel Comet Lake-U ` | Working     | 
| Graphics     | `Intel(R) UHD Graphics `                    | Working     | 
| Audio        | `Realtek ALC 230 `                          | Working     |
| Ram          | `4/16GB - 2666 Mhz DDR4 Lexar`              | Working     |
| Storage      | `CS1030 250GB M.2 NVMe SSD PNY`             | Working     | 
| WiFi         | `Intel(R) Wireless-AC 9560 `                | Working     | 
| Bluetooth    | `Intel(R) Wireless Bluetooth(R)  `          | Working     | 
| Touchpad     | `I2C ELAN0001 [PnP - MSFT0001]  `           | Working     |
| Keyboard     | `Standard PS/2 Keyboard  `                  | Working     | 
| Webcam       | `Integrated Camera  `                       | Working     | 
| Battery      | `46270 mWh `                                | Working     | 


### Geekbench:

| Information           | Result | ID Information                                                 | Operating system  | Model ID        |
| --------------------- | ------ | -------------------------------------------------------------- | ----------------- | --------------- |
| CPU Single-Core Score | 1034   | [ID 8181935](https://browser.geekbench.com/v6/cpu/8181935)     | `macOS Sonoma`    | `MacBookAir8,1` |
| CPU Multi-Core Score  | 2063   | [ID 8181935](https://browser.geekbench.com/v6/cpu/8181935)     | `macOS Sonoma`    | `MacBookAir8,1` |
| iGPU OpenCL Score     | 3402   | [ID 2896309](https://browser.geekbench.com/v6/compute/2896309) | `macOS Sonoma`    | `MacBookAir8,1` |
| iGPU Metal Score      | 4598   | [ID 2896295](https://browser.geekbench.com/v6/compute/2896295) | `macOS Sonoma`    | `MacBookAir8,1` |


### Kext Used:

| Kext                                                                                       | Info                                                                                                                                                                                                                                                                                                       |
| :----------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Lilu.kext](https://github.com/acidanthera/Lilu)                                           | Arbitrary kext and process patching on macOS.                                                                                                                                                                                                                                                              |
| [VirtualSMC.kext](https://github.com/acidanthera/VirtualSMC)                               | SMC Emulator Layer.                                                                                                                                                                                                                                                                                        |
| [WhateverGreen.kext](https://github.com/acidanthera/WhateverGreen)                         | Various patches necessary for certain ATI/AMD/Intel/Nvidia GPUs. This is needed for Intel HD 520.                                                                                                                                                                                                          |
| [ECEnabler.kext](https://github.com/1Revenger1/ECEnabler)                                  | Allows reading Embedded Controller fields over 1 byte long, vastly reducing the amount of ACPI modification needed (if any) for working battery status.                                                                                                                                                    |
| [CpuTscSync.kext](https://github.com/acidanthera/CpuTscSync)                               | It is a Lilu plugin, combining functionality of VoodooTSCSync and disabling xcpm_urgency if TSC is not in sync. It should solve kernel panics after wake.                                                                                                                                                  |
| [AirportItlwm.kext](https://github.com/OpenIntelWireless/itlwm)                            | Intel Wi-Fi Drivers for macOS.                                                                                                                                                                                                                                                                             |
| [HoRNDIS.kext](https://github.com/jwise/HoRNDIS)                                           | Android USB tethering driver for Mac OS X                                                                                                                                                                                                                                                                  |
| [HWPEnabler.kext](https://github.com/goodwin/HWPEnable)                                    | HWP is a technology introduced in Skylake which lets the CPU select its own stepping speed without the usage of the CPU Multiplier. Additionally it trottles/boosts itself much faster, which improoves overall CPU performance. With enabled HWP you dont need to create SSDTs with CPU P-States anymore. |
| [RTCMemoryFixup.kext](https://github.com/acidanthera/RTCMemoryFixup)                       | open source kernel extension providing a way to emulate some offsets in your CMOS (RTC) memory                                                                                                                                                                                                             |
| [SMCBatteryManager.kext](https://github.com/acidanthera/VirtualSMC)                        | Battery Status Monitoring.                                                                                                                                                                                                                                                                                 |
| [SMCProcessor.kext](https://github.com/acidanthera/VirtualSMC)                             | Processor Temp Monitoring.                                                                                                                                                                                                                                                                                 |
| [SMCSuperIO.kext](https://github.com/acidanthera/VirtualSMC)                               | Fan Reading.                                                                                                                                                                                                                                                                                               |
| [BlueToolFixup.kext](https://github.com/acidanthera/BrcmPatchRAM)                          | Required for macOS 12 or newer, as in macOS 12 Apple has changed parts of the Bluetooth stack from kernel-space to user-space                                                                                                                                                                              |
| [IntelBluetoothFirmware.kext](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) | Intel Bluetooth Drivers for macOS.                                                                                                                                                                                                                                                                         |
| [IntelBTPatcher.kext](https://github.com/zxystd/IntelBTPatcher)                            | A Lilu base patcher that fix Intel Bluetooth on Bigsur, Catalina, Mojave, High sierra etc, tested with Bigsur and Catalina all working good.                                                                                                                                                               |
| [RestrictEvents.kext](https://github.com/Mieze/RTL8111_driver_for_OS_X)                    | Lilu Kernel extension for blocking unwanted processes causing compatibility issues on different hardware and unlocking the support for certain features restricted to other hardware.                                                                                                                      |
| [NVMeFix.kext](https://github.com/acidanthera/NVMeFix)                                     | NVMeFix is a set of patches for the Apple NVMe storage driver, IONVMeFamily. Its goal is to improve compatibility with non-Apple SSDs.                                                                                                                                                                     |
| [FeatureUnlock.kext](https://github.com/acidanthera/FeatureUnlock)                         | Add Sidecar support to unsupported models                                                                                                                                                                                                                                                                  |
| [BrightnessKeys.kext](https://github.com/acidanthera/BrightnessKeys)                       | Handler for brightness keys without DSDT patches                                                                                                                                                                                                                                                           |
| [UTBMap.kext](https://github.com/USBToolBox/kext)                                          | Contains USB port mappings.                                                                                                                                                                                                                                                                                |
| [AppleALC.kext](https://github.com/acidanthera/AppleALC)                                   | For Audio.                                                                                                                                                                                                                                                                                                 |
| [VoodooI2C.kext](https://github.com/VoodooI2C/VoodooI2C)                                   | For I2C Touchpad.                                                                                                                                                                                                                                                                                          |
| [VoodooI2CHID.kext](https://github.com/VoodooI2C/VoodooI2C)                                | For ELAN Touchpad.                                                                                                                                                                                                                                                                                         |
| [VoodooPS2Controller.kext](https://github.com/RehabMan/OS-X-Voodoo-PS2-Controller)         | Contains updated Voodoo PS/2 Controller, improved Keyboard & Synaptics TouchPad.                                                                                                                                                                                                                           |
| [HibernationFixup.kext](https://github.com/acidanthera/HibernationFixup)                   | A Lilu plugin intended to fix hibernation compatibility issues.                                                                                                                                                                                                                                            |
| [YogaSMC.kext](https://github.com/zhen-zen/YogaSMC)                                        | ACPI driver for OEM hardware.                                                                                                                                                                                                                                                                              |


### SSDT Used:

| Kext                 | Info                                       | Refrence Link                                                                                                             |
| :------------------- | :----------------------------------------- | :------------------------------------------------------------------------------------------------------------------------ |
| SSDT-AWAC.aml        | Fixing System Clocks                       | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac.html)                                          |
| SSDT-EC.aml          | Fixing Embedded Controller                 | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix.html)                                        |
| SSDT-ECRW.aml        | YogaSMC-ACPI driver for OEM hardware       | [Link](https://github.com/zhen-zen/YogaSMC)                                                                               |
| SSDT-YVPC.aml        | YogaSMC-ACPI driver for OEM hardware       | -                                                                                                                         |
| SSDT-PNLFCFL.aml.aml | Fixing Backlight                           | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/backlight.html)                                       |
| SSDT-EC-USBX.aml     | Fixes EC and USB Power Supply              | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix.html#fixing-embedded-controller-ssdt-ecusbx) |
| SSDT-RHUB.aml        | Fixing RHUB                                | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Universal/rhub-methods/ssdttime.html)                         |
| SSDT-GPRW.aml        | GPRW/UPRW/LANC Instant Wake Patch          | [Link](https://dortania.github.io/OpenCore-Post-Install/usb/misc/instant-wake.html)                                       |
| SSDT-TPD0.aml        | Fixing Trackpads                           | [Link](https://voodooi2c.github.io/#GPIO%20Pinning/GPIO%20Pinning)                                                        |
| SSDT-HPET.aml        | Fixing IRQ Conflicts                       | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Universal/irq.html)                                           |
| SSDT-XOSI.aml        | Fixing Trackpads                           | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/trackpad.html)                                        |
| SSDT-MEM2.aml        | Adds MEM2 ACPI Device to IGPU              | -                                                                                                                         |
| SSDT-PLUG.aml        | Enables native CPU Power Management (XCPM) | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Universal/plug.html)                                          |
| SSDT-SBUS.aml        | Adds missing MCHC Device                   | [Link](https://dortania.github.io/Getting-Started-With-ACPI/Universal/smbus.html)                                         |
| SSDT-RCSM.aml        | -                                          | -                                                                                                                         |
| SSDT-ALS0.aml        | Fixing SMBus support                       | [Link](https://github.com/Acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/Source/SSDT-ALS0.dsl)                      |


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

[![](https://img.shields.io/badge/iCloud-nusratov.sobirjon@icloud.com-informational?style=flat&logo=apple&logoColor=white&color=cbcdcc)](mailto:nusratov.sobirjon@icloud.com)[![](https://img.shields.io/badge/Telegram-@Nusratov_Sobirjon-informational?style=flat&logo=telegram&logoColor=white&color=89e2ff)](https://t.me/Nusratov_Sobirjon)[![](https://img.shields.io/badge/Facebook-Nusratov_Sobirjon-informational?style=flat&logo=facebook&logoColor=white&color=3a4dc9)](https://www.facebook.com/Sobirjon.Nusratov)


