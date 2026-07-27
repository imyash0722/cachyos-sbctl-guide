# CachyOS Limine Secure Boot Fix

This repository/document outlines the solution for a specific boot panic encountered in CachyOS when setting up Secure Boot using `sbctl`. 

When `sbctl` signs the kernel and initramfs to enable Secure Boot, their cryptographic hashes change. CachyOS configures the Limine bootloader to strictly verify these BLAKE2B hashes to prevent tampering. As a result, signing the files causes a hash mismatch, and Limine will panic and halt the boot process.

---

## System Specifications

The environment where this issue occurred and was resolved:

* **OS:** CachyOS x86_64
* **Kernel:** Linux 7.1.4-1-cachyos
* **Bootloader:** Limine 12.5.2 (x86_64)
* **Shell:** zsh 5.9.2
* **Window Manager:** KWin (Wayland)
* **Terminal:** konsole 26.4.3

---

## The Issue

After running `sbctl` to generate keys and sign the system files, rebooting results in the following error:

```text
linux: Loading kernel `boot():/.../linux-cachyos/vmlinuz#...`
PANIC: Blake2b hash for URI `boot():/.../linux-cachyos/vmlinuz#...` does not match!
```

### UEFI Input Glitch (The Secondary Problem)
Attempting to edit the Limine boot entry by pressing `E` to delete the hashes resulted in a firmware glitch. Pressing `Backspace` or `Delete` printed `~` characters, and using terminal shortcuts like `Ctrl+H` printed glitched red `h` blocks instead of deleting text. 

**The Cause:** A hardware conflict with the mouse in the motherboard's pre-boot UEFI environment interfered with the keyboard driver. Resolving the mouse connection restored normal keyboard functionality in the Limine editor.

---

## The Solution

Here is the step-by-step process used to bypass the panic and permanently fix the boot configuration.

### Step 1: Bypassing the Panic in Limine
To boot into the system without a Live USB, the hardcoded hashes must be temporarily removed from the boot entry.

1. At the Limine boot menu, highlight the CachyOS entry and press **`E`** to edit.
2. Navigate to the `path:` line (which points to `vmlinuz`).
3. Delete the `#` symbol and the entire alphanumeric hash string at the very end of the line.
4. Navigate to the `module_path:` line (which points to `initramfs`).
5. Delete the `#` symbol and the entire hash string at the end of this line as well.
6. Press **`F10`** to boot into the OS.

### Step 2: Permanently Updating the Hashes
Once successfully booted into the CachyOS desktop, the system must be told to recalculate the new hashes for the Secure Boot-signed files and update `/boot/limine.conf`.

1. Open a terminal.
2. Run the automated CachyOS configuration script:
   ```bash
   sudo limine-enroll-config
   ```
3. The script will output confirmation that it has updated the configuration and signed the Limine EFI binaries (e.g., `Signed /boot/EFI/limine/limine_x64.efi`). 

### Step 3: Cleaning up `sbctl` (Optional but Recommended)
After a system update, old kernels tracked by `sbctl` might be deleted, leaving "ghost" entries in the database.

1. Verify the Secure Boot status and file tracking:
   ```bash
   sbctl status
   sudo sbctl verify
   ```
2. If `sbctl verify` reports missing files (marked with a red `‼`) inside `/boot/.../limine_history/`, remove them from the tracking database to keep the environment clean:
   ```bash
   sudo sbctl remove-file <path-to-missing-file>
   ```

Secure Boot is now fully enabled, and the system will automatically update the Limine hashes during future kernel updates via pacman hooks.
