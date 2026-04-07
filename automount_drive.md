# 💾 Universal Auto-Mount for External Disks

I was hitting a classic friction point on my workstation: every time I plugged in my **KpihX-Backup** drive, I had to manually click its icon in the file manager for it to actually mount. 

The problem? My automated backup scripts (running at 22:00) don't have fingers to click icons. They just saw a missing drive and failed. 

Here is how I fixed it using the "Lazy Mounting" power of `systemd`.

---

## 🧠 The Intuition: "Don't mount, just watch"

Normally, Ubuntu's default behavior (`udisks2`) is reactive: it sees the USB, but it waits for an explicit user request. 

We want to flip the script. Instead of waiting for a click, we want the system to **watch** the mount point. As soon as anything (a script, a terminal, or you) tries to enter that folder, the system should instantly "snap" the disk into place.

### 🏗️ Architecture: The "Trap" Logic

```text
  ┌─────────────┐       ┌──────────────┐      ┌───────────────┐
  │ 🔌 USB Plug │ ──→   │  🐧 Kernel   │ ──→  │ ⚖️  Systemd    │
  └─────────────┘       └──────────────┘      └───────────────┘
                                                      │
                                                      ▼
                      ┌────────────────────────────────────────────────┐
                      │  🔍 Watcher: /media/kpihx/KpihX-Backup         │
                      │  (Empty folder acting as a sensor/trap)        │
                      └────────────────────────────────────────────────┘
                                        │
                                        │  Access detected! (ls, cd, cp)
                                        ▼
                      ┌────────────────────────────────────────────────┐
                      │  ⚡ Auto-Mount triggered immediately           │
                      │  (UUID matched -> /dev/sda1 -> Mounted!)       │
                      └────────────────────────────────────────────────┘
```

---

## 🛠️ The Fix

### 1. Identify your hardware DNA
First, we need the UUID (Unique ID) of the partition. Labels can be duplicated; UUIDs are forever.

```bash
# Plugin your disk and find its UUID
ls -l /dev/disk/by-uuid/ | grep sda1
# OR
lsblk -f
```

Example result: `05e59157-6516-473c-bc33-d486ee1327bb`

### 2. Create a Permanent Home
For `systemd` to place its "sensor", the directory must exist and be static.

```bash
sudo mkdir -p /media/kpihx/KpihX-Backup
sudo chown $USER:$USER /media/kpihx/KpihX-Backup
```

> [!IMPORTANT]
> GNOME usually creates and deletes these folders dynamically. By creating it manually, you take ownership back.

### 3. Set the Trap in `fstab`
Backup your configuration then edit `/etc/fstab`:

```bash
sudo cp /etc/fstab /etc/fstab.bak
sudo nano /etc/fstab
```

Add this line at the bottom:
```text
UUID=YOUR_UUID_HERE /media/kpihx/KpihX-Backup ext4 nosuid,nodev,nofail,x-systemd.automount,x-gvfs-show 0 2
```

**Key Options:**
- `nofail`: System won't hang at boot if the disk is unplugged (CRITICAL for USB).
- `x-systemd.automount`: The magic "lazy mount" trigger.
- `x-gvfs-show`: Keeps the icon visible in your sidebar.

### 4. Activate the Magic
Reload the system configuration:

```bash
sudo systemctl daemon-reload
```

---

## 🧪 Verification: The "No-Click" Test

1. Unmount the disk if it's there.
2. Unplug and replug the disk.
3. Open a terminal and run `ls /media/kpihx/KpihX-Backup`.

| Before Access | After `ls` |
| :--- | :--- |
| Folder exists but feels empty | Folder instantly populates with your files |
| Unit: `active (waiting)` | Unit: `active (running)` |

**It just works.** Your scripts will now trigger the mount as soon as they try to read the disk. No fingers required. 🚀
