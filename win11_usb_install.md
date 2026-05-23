# 💿 Windows 11: The Portable SSD Installation (USB Bypass)

I needed a dedicated Windows 11 environment for a secure certification (Linguaskill). Instead of dual-booting my main internal drive, I opted for a high-speed portable SSD. 

The problem? Windows Setup is designed to be a "Yes-Man" for internal disks but a "Guard Dog" for external ones. To get it running on USB, I had to jump over two major hurdles: an installer bug and a hardcoded policy block.

---

## 🧠 The Intuition: "Bypassing the Gatekeeper"

Normally, the Windows Installer (WinPE) performs a "Hardware Check" before letting you pick a drive. If it sees a "Removable" bit or a USB controller, it refuses to install. 

The logic we use is to **ignore the GUI entirely**. We manually apply the Windows system image (WIM/ESD) to the disk—much like a master key—then manually tell the motherboard's firmware where the boot files are.

### 🏗️ Architecture: GUI vs. CLI

```text
  ┌────────────────────────┐         ┌────────────────────────┐
  │   Option A: GUI        │         │   Option B: CLI        │
  │ (The Standard Path)    │         │ (The Sovereign Path)   │
  └───────────┬────────────┘         └───────────┬────────────┘
              │                                  │
      [ Hardware Check ] ──→ [ FAIL ]    [ Diskpart / DISM ] ──→ [ SUCCESS ]
              │                                  │
     "USB not supported"                Manual blocks mapping
```

---

## 🛠️ The Fix

### Phase 1: The "0x80070001" Bug (24H2 Workaround)
On recent Windows 11 ISOs (v24H2), the new setup engine often crashes with an "Unexpected Error".

**The Reaction:**
Don't panic and don't re-create your USB stick yet. 
- Restart the PC.
- At the start of the installer, look for a link: **"Use previous version of setup"** (or "Go back to the old installer").
- This uses the legacy (and stable) Win11 setup engine which bypasses this specific crash.

### Phase 2: The USB Block (The manual "Disk Surgery")
If you get the message: *"Setup does not support installation to disks connected through a USB port"*, follow this path.

**Problem:** The GUI is hardlocked.
**Action:** Press **`Shift + F10`** (or `Shift + Fn + F10`) to open the Command Prompt.

#### 1. Disk Preparation (`diskpart`)
We must create the structure Windows expects for a UEFI boot.
```cmd
diskpart
list disk
select disk 1          (Verify it's your SSD!)
create partition efi size=100
format quick fs=fat32 label="System"
assign letter=S        (S for System)
create partition msr size=16
create partition primary
format quick fs=ntfs label="Windows"
assign letter=W        (W for Windows)
exit
```

**Why this matters:**
- **EFI (GPT):** This is the mailbox for the motherboard (UEFI).
- **MSR:** Reserved space for Windows partition management.
- **Primary:** The actual C: drive where your files live.

#### 2. Image Ingestion (`DISM`)
We manually "pour" Windows onto the disk.
Find your USB letter (`E:` in this example) and identify your target version (Index 1 is usually Pro/Home).
```cmd
dism /Apply-Image /ImageFile:E:\sources\install.wim /Index:1 /ApplyDir:W:\
```

**Key Parameters:**
- `/ImageFile`: The source of truth (the OS compressed into a file).
- `/Index`: Picks the edition (Home/Pro) inside the WIM.
- `/ApplyDir`: The destination partition.

#### 3. Firmware Wiring (`BCDBOOT`)
Finally, we tell the SSD's EFI partition how to launch the Windows we just applied.
```cmd
bcdboot W:\Windows /s S: /f UEFI
```

**The Logic:**
- `W:\Windows`: Where the OS is.
- `/s S:`: Where the boot files should go (EFI partition).
- `/f UEFI`: Ensures it boots in modern UEFI mode, not legacy BIOS.

---

## 🧪 Result: A Sovereign Portable OS

1. Close the terminal.
2. Exit the installer.
3. Unplug the USB stick (the installer) but leave the SSD.
4. **Reboot.**

Windows will start the "Getting ready" phase. You have successfully bypassed the USB restriction. 🚀

> **Pro Tip:** Disable "Fast Startup" in Windows Power Settings immediately after login. This prevents NTFS locks when moving the SSD back to a Linux environment!
