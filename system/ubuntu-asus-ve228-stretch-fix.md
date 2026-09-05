# Resolving Display Stretching on ASUS VE228 (Wayland Viewport Snap)

This article documents the engineering journey, testing iterations, and final software-only resolution for preventing vertical image stretching on the external **ASUS VE228** monitor under GNOME Wayland.

---

## 1. The Concrete Situation

I was trying to establish a custom **1920x880** viewport on my external monitor (`HDMI-1`) to serve as an interactive, letterboxed screen, leaving exactly 200 pixels of pure black at the bottom. The goal was to prevent vertical stretching while keeping the workspace fully interactive and compatible with modern Wayland scaling.

However, the moment I forced the custom resolution, the monitor did this:

```
Expected:                              Actual (Stretched):
┌───────────────────────────────┐      ┌───────────────────────────────┐
│        Active 1920x880        │      │                               │
│           Workspace           │      │                               │
│                               │  →   │        Active 1920x880        │
├───────────────────────────────┤      │      Workspace stretched      │
│  200px Ignored (Black Void)   │      │        to fill 1080p          │
└───────────────────────────────┘      └───────────────────────────────┘
```

The ASUS VE228 has a native physical panel of **1366x768** but accepts standard **1920x1080** signals and downscales them. When fed a non-standard aspect ratio like 1920x880, its internal hardware scaler immediately stretches the pixels vertically to fill the panel. To make things worse, the monitor's physical OSD menu lacks a "1:1 Pixel Mapping" or "Just Scan" setting for non-standard timings.

---

## 2. The Exploration & Failures (Paths A–G)

Before arriving at the elegant native solution, I explored and tested multiple layers of the Linux display stack:

### Path A: DDC/CI Display Control
I attempted to bypass the OSD by using DDC/CI commands to programmatically tell the monitor scaler not to stretch:
```bash
sudo ddcutil capabilities | grep "VCP 0x86"
```
*   *Why it failed:* The ASUS VE228's firmware does not support VCP feature `0x86` (Display Scaling).

### Path B: DRM Connector Properties
I probed the Direct Rendering Manager (DRM) properties of the Intel graphics driver to enforce margins (underscan):
```bash
modetest -M i915 -c
```
*   *Why it failed:* The driver and connector did not expose the `left margin`, `right margin`, `top margin`, or `bottom margin` properties on this hardware.

### Path C: Custom Modelines
I calculated custom modelines to increase the vertical blanking interval (VBI) mathematically while keeping the active area at 880 lines.
*   *Why it failed:* Vertical blanking intervals are timing signals, not visible pixels. The monitor simply ignored the extra blanking and scaled the active 880 lines to the full height.

### Path D: Custom EDID Injection
I extracted the monitor's base EDID, patched the first Detailed Timing Descriptor (DTD) from `1920x1112` (the monitor's weird default) to `1920x880`, recomputed the checksum, and injected it into the kernel:
*   *Why it failed:* Changing the EDID only changed what the GPU sent. The monitor's scaler still saw 880 lines of active video data and stretched it.

### Path E: X11 Viewport Transformations
I attempted to map the framebuffer coordinates using `xrandr --transform`:
```bash
xrandr --output HDMI-1 --mode 1920x1080 --fb 1920x880 --transform 1,0,0,0,1,0,0,0,1
```
*   *Why it failed:* GNOME Mutter on Wayland strictly blocks client-side hardware transformation calls, throwing X11 protocol errors (`BadRRCrtc` / `BadMatch`).

### Path F: OBS Studio Projector (Solution A)
I configured the output to 1080p, opened OBS Studio, captured an 880p window, positioned it at coordinate `Y=0` (Top), and launched a Fullscreen Projector to the HDMI output.
*   *Why it failed:* This solved the stretching visually, but it destroyed all interactivity. The projection is a read-only video stream—you cannot click or type inside it.

### Path G: CEA-861 EDID Overscan Margins
I attempted to inject a 1080p EDID with asymmetrical overscan border flags written into the CEA-861 Extension Block.
*   *Why it failed:* The Linux Intel DRM kernel drivers (`i915`/`xe`) completely ignore EDID overscan/border parameters under Wayland, delegating scaling margins entirely to userspace.

---

## 3. The Gamescope Detour & Wayland Scaling Trap

To restore interactivity, I pivoted to **Gamescope** (Valve's micro-compositor):
```bash
gamescope -W 1920 -H 1080 -w 1920 -h 880 -f -S fit -- <app>
```
This was a major breakthrough: the app was 100% interactive, and the monitor received a standard 1080p signal (no stretching). However, two roadblocks emerged:

1.  **Gamescope Centering:** Gamescope strictly centers nested windows, leaving 100px of black at the top and 100px at the bottom. It lacks layout offset parameters.
2.  **The Wayland Scale Bug:** Because my primary laptop display (`eDP-1`) ran at `1.333333x` scale, GNOME Wayland automatically divided Gamescope's requested `1920x880` size by `1.33`, making the window render at a shrunken `1440x660` in the center of the screen.

I compensated for this by multiplying the boundary sizes by the scale factor at launch (`-W 2560 -H 1173`), but the complexity of maintaining background compositor processes and scripting keyboard sync layouts made it too heavy.

---

## 4. The Elegant Software Resolution: Tiling Shell

The ultimate solution was entirely software-based and natively integrated into GNOME Wayland: **Tiling Shell**.

Instead of trying to force the GPU or the monitor firmware to hide 200 pixels, we configure GNOME's compositor to handle window placements geometrically.

```
+--------------------------------------------+
|                                            |
|          Active 1920x880 Zone              | (Tiling Shell Snap Area)
|       (100% Interactive, Sharp)            |
|                                            |
+--------------------------------------------+
|       200px Solid Black Wallpaper          | (Masks the unused screen bottom)
+--------------------------------------------+
```

### Setup Steps

1.  **Enable XWayland Native Scaling:**
    Tell GNOME to allow native scaling for X11/XWayland windows. This fixes the mixed-DPI logical scaling issue system-wide:
    ```bash
    gsettings set org.gnome.mutter experimental-features '["scale-monitor-framebuffer", "xwayland-native-scaling"]'
    ```
2.  **Install the Extension:**
    Open the **Extension Manager** GUI, search for **"Tiling Shell"** (by *Javier Monton*), and click **Install**.
3.  **Draw the Custom Snap Layout:**
    *   Click the Tiling Shell icon in your top panel and select **Layout Editor**.
    *   Select your external ASUS monitor (`HDMI-1`).
    *   Create a custom layout. Draw a horizontal split line separating the screen into a **1920x880** top region and a **200px** bottom region. Save it.
4.  **Apply Solid Black Wallpaper:**
    Set the desktop background of your external monitor to a solid black color.
5.  **Use the Snapping Gesture:**
    Drag your window (Edge, Cursor, VLC) to the top edge of the ASUS monitor. Release it inside the 880p snap preview. 

The window instantly snaps to `1920x880` at the top, leaving the bottom 200 pixels perfectly black (masked by the wallpaper). The desktop remains fully interactive, performance is native, and keyboard layout AZERTY configurations are untouched.
