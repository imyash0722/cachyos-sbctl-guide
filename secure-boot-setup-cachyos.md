# Secure Boot Setup Guide for CachyOS

This guide explains how to properly set up Secure Boot on CachyOS using `sbctl` (Secure Boot Manager). Setting this up correctly will ensure your system is secured against unauthorized pre-boot execution while preventing Limine bootloader panics.

## Prerequisites

Before starting, you must prepare your motherboard's UEFI/BIOS:
1. Reboot your computer and enter your BIOS/UEFI settings.
2. Navigate to the **Secure Boot** section.
3. Clear/delete the existing factory Secure Boot keys. This will put your motherboard into **Setup Mode**.
4. Save changes and boot back into CachyOS.

---

## Step 1: Verify Setup Mode

Once you are back in CachyOS, open your terminal and verify that `sbctl` recognizes your system is in Setup Mode.

```bash
sbctl status
```

You should see `Setup Mode: ✓ Enabled`. If it says Disabled, you need to go back into your BIOS and make sure the factory keys are fully cleared.

---

## Step 2: Create Custom Keys

Next, generate your own custom cryptographic keys that will be used to sign your boot files.

```bash
sudo sbctl create-keys
```

---

## Step 3: Enroll Keys (with Microsoft Compatibility)

Now, enroll your newly created keys into your motherboard's firmware. 

*Note: The `-m` flag is highly recommended as it enrolls Microsoft's certificates alongside your own. This ensures compatibility with third-party hardware like dedicated GPUs (Option ROMs) and prevents display issues during boot.*

```bash
sudo sbctl enroll-keys -m
```

---

## Step 4: Sign the Boot Files

With your keys enrolled, you need to sign all the necessary boot files (your kernel, initramfs, and the Limine bootloader itself). The following command automatically finds and signs everything required:

```bash
sudo sbctl sign -s -a
```

---

## Step 5: Update Limine Hashes (CRITICAL)

Because `sbctl` just modified your boot files by signing them, their BLAKE2B cryptographic hashes have changed. If you reboot now, **Limine will trigger a panic** because the files do not match its recorded hashes.

To calculate the new hashes and update your Limine configuration automatically, run:

```bash
sudo limine-enroll-config
```

---

## Step 6: Reboot and Verify

Reboot your computer. If everything was done correctly, CachyOS will boot normally without any panics.

Once you are back on your desktop, verify that Secure Boot is actively enforcing:

```bash
sbctl status
```

You should now see:
* `Setup Mode: ✓ Disabled`
* `Secure Boot: ✓ Enabled`

Your CachyOS installation is now fully secured with Secure Boot! Future kernel updates will be automatically signed and hashed via CachyOS pacman hooks.
