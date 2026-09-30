# Steam HiDPI Scaling Guide for Linux (4K Displays)

## Problem

Steam on Linux has no automatic HiDPI scaling. On 4K displays (3840x2160), the interface, fonts, and images appear tiny because Steam runs at 100% scale by default.

This is especially problematic on **Wayland** compositors (Hyprland, Sway, etc.) where Steam runs through XWayland and ignores standard GTK/Qt scaling variables.

## Solution

The fix requires setting **three environment variables** that work together:

| Variable | Purpose |
|----------|---------|
| `STEAM_FORCE_DESKTOPUI_SCALING` | Steam's native UI scaling |
| `GDK_SCALE` | GTK UI element scaling |
| `GDK_DPI_SCALE` | Font DPI scaling |

## Quick Fix (Manual)

### 1. Kill Steam

```bash
killall steam
```

### 2. Launch Steam with Scaling

```bash
STEAM_FORCE_DESKTOPUI_SCALING=1.75 GDK_SCALE=1.75 GDK_DPI_SCALE=1.75 steam
```

### 3. Find Your Perfect Scale

Common values for 4K displays:

| Scale | Use Case |
|-------|----------|
| `1.5` | 27" 4K monitor (dense but readable) |
| `1.75` | 27-32" 4K monitor (comfortable) |
| `2.0` | 32"+ 4K monitor or TV (large) |

Adjust all three values together. Example for 1.5x:

```bash
STEAM_FORCE_DESKTOPUI_SCALING=1.5 GDK_SCALE=1.5 GDK_DPI_SCALE=1.5 steam
```

## Permanent Fix

### Method 1: Environment File (Recommended)

Create a systemd environment file that applies on login:

```bash
mkdir -p ~/.config/environment.d/
cat > ~/.config/environment.d/steam-scaling.conf << 'EOF'
STEAM_FORCE_DESKTOPUI_SCALING=1.75
GDK_SCALE=1.75
GDK_DPI_SCALE=1.75
EOF
```

**Log out and back in** for changes to take effect.

### Method 2: Desktop File Override

Override the system Steam launcher so it works from your application menu:

```bash
mkdir -p ~/.local/share/applications/
cp /usr/share/applications/steam.desktop ~/.local/share/applications/steam.desktop
```

Edit `~/.local/share/applications/steam.desktop` and change the `Exec=` line:

```ini
Exec=env STEAM_FORCE_DESKTOPUI_SCALING=1.75 GDK_SCALE=1.75 GDK_DPI_SCALE=1.75 /usr/bin/steam %U
```

Also update all `Actions` entries (Store, Library, etc.) if you want right-click menu items to launch with scaling too.

Then refresh the desktop database:

```bash
update-desktop-database ~/.local/share/applications/
```

## Why Three Variables?

- **`STEAM_FORCE_DESKTOPUI_SCALING`**: Steam's own scaling flag. Required but not always sufficient on Wayland.
- **`GDK_SCALE`**: Scales GTK UI elements (buttons, menus, windows). Needed because parts of Steam use GTK.
- **`GDK_DPI_SCALE`**: Scales fonts. Prevents tiny text when other elements are scaled.

On **X11**, `STEAM_FORCE_DESKTOPUI_SCALING` alone might be enough. On **Wayland/XWayland**, all three are needed.

## Troubleshooting

### Steam icon disappeared from launcher

Your `.desktop` file is missing fields. Copy the original:

```bash
cp /usr/share/applications/steam.desktop ~/.local/share/applications/steam.desktop
# Then edit only the Exec= line
```

### Changes not applying

Environment files in `~/.config/environment.d/` only load on login. **Log out and back in.**

### Steam still small after setting variables

Check that Steam actually has the variables:

```bash
pgrep -a steam | head -5
cat /proc/$(pgrep -f "steam -srt" | head -1)/environ | tr '\0' '\n' | grep -i scale
```

If missing, Steam was launched without the environment. Make sure you're using the modified `.desktop` file or have logged out/in.

### Using Ansible

See [`steam-highdpi.yml`](steam-highdpi.yml) for an automated playbook.

## Files Created

- `~/.config/environment.d/steam-scaling.conf` — environment variables
- `~/.local/share/applications/steam.desktop` — desktop launcher override

## Desktop Environment Notes

| DE/Compositor | Notes |
|---------------|-------|
| **GNOME (Wayland)** | Usually works with just `STEAM_FORCE_DESKTOPUI_SCALING` |
| **KDE (Wayland)** | May need all three variables |
| **Hyprland** | Requires all three variables + `.desktop` override |
| **Sway** | Same as Hyprland |
| **X11 (any)** | Usually works with `STEAM_FORCE_DESKTOPUI_SCALING` alone |
