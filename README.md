# Workspace Forge

Per-workspace **wallpapers and desktop icon sets** for the Cinnamon desktop
(Linux Mint / LMDE, X11). Each virtual desktop gets its own background and its
own desktop icons, and the change fires on *any* workspace switch: keyboard
shortcuts, Expo, panel applet scrolling, window drags, third-party tools.

Cinnamon does not support this natively — upstream feature requests
([#3346](https://github.com/linuxmint/cinnamon/issues/3346),
[#10919](https://github.com/linuxmint/cinnamon/issues/10919)) are closed as
won't-fix — so this is a hardened, CLI-driven implementation of the
symlink-swap technique pioneered by **dj1s** in the Linux Mint forum:
<https://forums.linuxmint.com/viewtopic.php?t=360221>

## Features

- **Catches every switch method** via an `xprop -root -spy` watcher daemon —
  you don't have to change how you use workspaces at all
- Per-workspace wallpaper *and* picture-options (zoom / centered / etc.)
- Per-workspace desktop icon folders (each workspace is a real directory)
- Named workspaces, flashed as a notification on switch
- Central config in `~/.config/workspace-forge/` — no hidden files on your
  desktop folders
- Every config mutation is backed up; one-command `undo`
- Safe one-time bootstrap that relocates (never deletes) your Desktop
- Single-instance-guarded daemon with an autostart entry

## Requirements

- Cinnamon on X11 (default on Linux Mint — not Wayland)
- `x11-utils` (xprop) and `libnotify-bin` (notify-send):

```bash
sudo apt install x11-utils libnotify-bin
```

## Install

```bash
git clone https://github.com/meomizzle/cinnamon-workspace-forge.git
cd cinnamon-workspace-forge
mkdir -p ~/.local/bin
chmod +x bin/*
ln -sf "$(pwd)/bin/ws-forge"  ~/.local/bin/ws-forge
ln -sf "$(pwd)/bin/ws-apply"  ~/.local/bin/ws-apply
ln -sf "$(pwd)/bin/ws-watchd" ~/.local/bin/ws-watchd
```

## Quick start

```bash
# 1. One-time bootstrap: moves your Desktop contents into ~/desk-forge/original
#    and symlinks ~/Desktop to it. Your desktop looks identical afterwards.
ws-forge init

# 2. Register workspaces (1-based, matching Cinnamon's workspace numbering).
#    Simple naming convention shown here: wkspNN workspaces, wkspNN-wpNN wallpapers.
ws-forge create 1 --name wksp01 --wallpaper ~/Pictures/wksp01-wp01.png --seed
ws-forge create 2 --name wksp02 --wallpaper ~/Pictures/wksp02-wp02.png
ws-forge create 3 --name wksp03 --wallpaper ~/Pictures/wksp03-wp03.png --options zoom

# 3. Run the watcher: now and at every login
ws-forge install-autostart
nohup ws-watchd >/dev/null 2>&1 &

# 4. Switch workspaces the way you always do. Enjoy.
```

`--seed` copies your existing desktop icons into that workspace's folder as a
starting point; trim them down per workspace afterwards.

## Commands

| Command | Purpose |
| --- | --- |
| `ws-forge init [-y]` | One-time bootstrap of the Desktop symlink |
| `ws-forge create N --name NAME [--wallpaper PATH] [--options OPT] [--seed]` | Register a workspace |
| `ws-forge amend N [--name ...] [--wallpaper ...] [--options ...]` | Change settings |
| `ws-forge delete N [--purge]` | Remove from config (and folder, with `--purge`) |
| `ws-forge list` | Table of configured workspaces |
| `ws-forge apply N` | Apply workspace N immediately |
| `ws-forge undo` | Restore previous config |
| `ws-forge status` | Current workspace, symlink target, watcher state, journal |
| `ws-forge install-autostart` | Start the watcher at every login |

## How it works

- `ws-watchd` watches the X11 root property `_NET_CURRENT_DESKTOP`
- On change, `ws-apply` sets the wallpaper via `gsettings`
  (`org.cinnamon.desktop.background`) and re-points the `~/Desktop` **symlink**
  at that workspace's folder under `~/desk-forge/ws-N/`
- Nemo is nudged to redraw the desktop by toggling and restoring
  `org.nemo.desktop desktop-layout`

## Known limitations (honest ones)

- **Expo/grid shows the old wallpaper mid-animation**; the swap lands when the
  switch completes. Unfixable without Cinnamon code changes — see the
  upstream issues above.
- System icons (Computer, Trash, mounted volumes) are drawn from Nemo
  settings, not the Desktop folder, so they look the same on every workspace.
- Apps that save to "Desktop" write into whichever workspace folder is active.
  Files do not follow you between workspaces.
- X11 only. Cinnamon on Wayland is not supported.
- Unmapped workspace numbers (e.g. if you add a 6th workspace without
  registering it) are skipped gracefully.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Icons didn't refresh after a switch | `Ctrl+Alt+Esc` (restart Cinnamon) |
| Watcher not running | `nohup ws-watchd >/dev/null 2>&1 &` or re-login |
| `xprop: not found` | `sudo apt install x11-utils` |
| Wallpaper didn't change | File moved/renamed? Check `ws-forge list` |

## Uninstall / rollback

```bash
kill "$(cat ~/.config/workspace-forge/watchd.pid)" 2>/dev/null
rm ~/.config/autostart/ws-watchd.desktop
rm ~/Desktop                  # remove the symlink
mv ~/desk-forge/original ~/Desktop   # your real Desktop returns
```

## Credits

- **dj1s** — the original technique and scripts
  ([forum thread](https://forums.linuxmint.com/viewtopic.php?t=360221))
- **smurphos** — the `xprop -root -spy` watcher concept in the same thread
- Built as a hardening of the above; original scripts provided the proof of
  concept.

## License

MIT — see [LICENSE](LICENSE).
