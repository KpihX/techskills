# Nautilus as default file manager (Ubuntu GNOME)

**Date:** 2026-05-22  
**Context:** Nemo (Cinnamon file manager) was installed manually; default for `inode/directory` was `nemo.desktop`. Switched back to GNOME Nautilus and purged Nemo packages.

## Set Nautilus as default (user, no root)

```bash
xdg-mime default org.gnome.Nautilus.desktop inode/directory
gsettings set org.gnome.desktop.default-applications.file-manager exec 'nautilus %U'
```

Edit `$HOME/.config/mimeapps.list`:

- `[Default Applications]` → `inode/directory=org.gnome.Nautilus.desktop`
- `[Added Associations]` → same for `inode/directory`

Verify:

```bash
xdg-mime query default inode/directory   # org.gnome.Nautilus.desktop
gio mime inode/directory
```

## Purge Nemo (requires sudo)

```bash
sudo apt purge -y nemo nemo-fileroller nemo-data libnemo-extension1
sudo apt autoremove -y
```

Packages removed: `nemo`, `nemo-data`, `nemo-fileroller`, `libnemo-extension1`.  
Left installed: `nautilus`, `nautilus-data`, `python3-nautilus` (GNOME stack).

## Notes

- `cinnamon-desktop-data` / `cinnamon-l10n` may remain as stray libs; they do not force Nemo as default.
- Drive archive paths mentioning "Nemo the Default File Manager" are unrelated KB exports, not workstation policy.
- Right-click / Nautilus scripts: see Gdrive note `Intégrer Warp à Nautilus, Clic Droit` if needed.
