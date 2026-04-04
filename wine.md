# 🍷 Wine — 32-bit Isolation and HiDPI Display

> **Machine:** KpihX-Ubuntu | **Date:** 2026-01-08

Running a Windows app under Linux is easy. Running it *cleanly* is not.
Two problems arise immediately: 32-bit apps pollute the default `~/.wine` prefix with legacy libs,
and old GUIs are unreadable on a 4K screen (96 DPI renders text microscopic).

The solution: **isolated prefixes per architecture** + **registry-level DPI tuning**.

## Core Concept: WINEPREFIX as a Container

Wine recreates a Windows environment (C: drive + registry) called a **WINEPREFIX**.
Default: `~/.wine`. Dangerous for mixed installs — one app can corrupt another's setup.

Rule: **one prefix per context** (32-bit apps get their own prefix).

## 32-bit Isolation — Aliases

Add to `~/.kshrc` (or `~/.zshrc`):

```bash
# Wine 32-bit isolation — separate prefix, forced win32 arch
alias wine32='WINEPREFIX=~/.wine32 WINEARCH=win32 wine'
alias winecfg32='WINEPREFIX=~/.wine32 WINEARCH=win32 winecfg'
alias winetricks32='WINEPREFIX=~/.wine32 WINEARCH=win32 winetricks'
```

First run of `winecfg32` creates `~/.wine32` as a clean 32-bit Windows install.

## HiDPI Fix — Virtual Desktop + DPI Scaling

Two levers for 4K readability:
1. **LogPixels** — tells Windows "more pixels per inch" → larger fonts
2. **Virtual Desktop** — confines the app in a fixed-size window (e.g. 1080p)

```bash
# 1. Clean any previous DPI attempt
wine reg delete "HKCU\Control Panel\Desktop" /v LogPixels /f

# 2. Set virtual desktop to 1080p
wine reg add "HKCU\Software\Wine\Explorer\Desktops" /v "Default" /t REG_SZ /d "1920x1080" /f
wine reg add "HKCU\Software\Wine\Explorer" /v "Desktop" /t REG_SZ /d "Default" /f

# 3. Apply 192 DPI (= 200% scaling, good for 4K)
wine reg add "HKCU\Control Panel\Desktop" /v LogPixels /t REG_DWORD /d 192 /f

# 4. Restart Wine server to apply
wineserver -k
```

> Use `wine32` instead of `wine` for the 32-bit prefix.

## Reset / Undo

```bash
# Remove virtual desktop
wine reg delete "HKCU\Software\Wine\Explorer" /v Desktop /f
wine reg delete "HKCU\Software\Wine\Explorer\Desktops" /f

# Remove DPI override (let system decide)
wine reg delete "HKCU\Control Panel\Desktop" /v LogPixels /f

wineserver -k
```
