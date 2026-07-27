# CachyOS Limine Secure Boot Guide & Panic Fix

This repository contains complete documentation for configuring Secure Boot (`sbctl`) on CachyOS, along with a step-by-step fix for the Limine bootloader `BLAKE2B hash mismatch` panic that can occur during the setup process.

Because CachyOS configures the Limine bootloader to strictly verify the cryptographic hashes of your boot files, manually signing your kernel with `sbctl` changes these hashes and triggers a security panic. This repository provides the proper setup sequence to avoid this issue, as well as the rescue steps if you are already stuck on the panic screen.

## Repository Contents

### 1. [Secure Boot Setup Guide for CachyOS](secure-boot-setup-cachyos.md)
A complete, start-to-finish guide on properly enabling and configuring Secure Boot on CachyOS. 
* Putting your motherboard in Setup Mode.
* Creating and enrolling custom keys with Microsoft compatibility.
* Signing the boot files.
* **Crucially:** Updating Limine hashes using `limine-enroll-config` to prevent boot panics.

### 2. [CachyOS Limine Secure Boot Fix (Panic Rescue)](limine-cachyos-panic.md)
A troubleshooting guide detailing how to bypass and permanently fix the Limine hash mismatch panic if you are currently unable to boot.
* How to temporarily remove the hardcoded hashes in the Limine pre-boot editor.
* How to fix UEFI hardware glitches (mouse/keyboard input interference) that prevent you from typing in the bootloader.
* How to permanently recalculate the hashes once booted to secure the system.
* How to clean up the `sbctl` tracking database.

---
*Created to help the CachyOS community easily navigate Secure Boot and Limine hash verification.*
