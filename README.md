# Steam HiDPI Settings

Ansible playbook and guide to fix Steam's tiny interface on HiDPI/4K displays on Linux.

## Problem

Steam on Linux has no automatic HiDPI scaling. On 4K displays (3840x2160), the interface, fonts, and images appear tiny because Steam runs at 100% scale by default.

This is especially problematic on **Wayland** compositors (Hyprland, Sway, GNOME Wayland, KDE Wayland, etc.) where Steam runs through XWayland and ignores standard GTK/Qt scaling variables.

## Solution

The fix requires setting **three environment variables** that work together:

| Variable | Purpose |
|----------|---------|
| `STEAM_FORCE_DESKTOPUI_SCALING` | Steam's native UI scaling |
| `GDK_SCALE` | GTK UI element scaling |
| `GDK_DPI_SCALE` | Font DPI scaling |

## Quick Start

### Prerequisites

- Ansible installed on your system
- Steam installed

### Usage

```bash
# Clone or download the playbook
git clone https://github.com/yourusername/steam-highdpi-settings.git
cd steam-highdpi-settings

# Run with default scale (1.75)
ansible-playbook steam-highdpi.yml

# Or specify a custom scale
ansible-playbook steam-highdpi.yml -e "steam_scale=2.0"
```

## What It Does

The playbook:

1. Creates `~/.config/environment.d/steam-scaling.conf` with the scaling variables
2. Copies the system Steam desktop file to `~/.local/share/applications/steam.desktop`
3. Updates the desktop file to include the scaling environment variables
4. Updates the desktop database

## Scale Recommendations

| Scale | Use Case |
|-------|----------|
| `1.5` | 27" 4K monitor (dense but readable) |
| `1.75` | 27-32" 4K monitor (comfortable, default) |
| `2.0` | 32"+ 4K monitor or TV (large) |

## Files

- `steam-highdpi.yml` - Ansible playbook for automated configuration
- `steam-highdpi.md` - Detailed manual guide with troubleshooting
- `README.md` - This file

## Manual Method

If you prefer not to use Ansible, see [`steam-highdpi.md`](steam-highdpi.md) for step-by-step manual instructions.

## Troubleshooting

### Changes not applying

Environment files in `~/.config/environment.d/` only load on login. **Log out and back in** after running the playbook.

### Steam still small

Check that Steam actually has the variables:

```bash
pgrep -a steam | head -5
cat /proc/$(pgrep -f "steam -srt" | head -1)/environ | tr '\0' '\n' | grep -i scale
```

### Test immediately

To test without logging out:

```bash
STEAM_FORCE_DESKTOPUI_SCALING=1.75 GDK_SCALE=1.75 GDK_DPI_SCALE=1.75 steam
```

## Desktop Environment Notes

| DE/Compositor | Notes |
|---------------|-------|
| **GNOME (Wayland)** | Usually works with just `STEAM_FORCE_DESKTOPUI_SCALING` |
| **KDE (Wayland)** | May need all three variables |
| **Hyprland** | Requires all three variables + `.desktop` override |
| **Sway** | Same as Hyprland |
| **X11 (any)** | Usually works with `STEAM_FORCE_DESKTOPUI_SCALING` alone |

## Why Three Variables?

- **`STEAM_FORCE_DESKTOPUI_SCALING`**: Steam's own scaling flag. Required but not always sufficient on Wayland.
- **`GDK_SCALE`**: Scales GTK UI elements (buttons, menus, windows). Needed because parts of Steam use GTK.
- **`GDK_DPI_SCALE`**: Scales fonts. Prevents tiny text when other elements are scaled.

On **X11**, `STEAM_FORCE_DESKTOPUI_SCALING` alone might be enough. On **Wayland/XWayland**, all three are needed.

## License

MIT License - Feel free to use and modify as needed.

## Contributing

Issues and pull requests welcome! If you have improvements for specific desktop environments or distributions, please share.
