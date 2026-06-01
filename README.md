# asrt

A fast, Omarchy-style rofi menu for Arch Linux that does one thing: uninstall a
package cleanly and safely.

Pick an explicitly-installed package from a fuzzy menu, see the exact set of
packages that would be removed (the "blast radius"), confirm, and remove it with
`pacman -Rns`. No cascade, no force, no surprises. The whole UI is rofi in dmenu
mode — there's no custom GUI code.

## How it works

1. Lists packages you explicitly installed that nothing else depends on
   (`pacman -Qetq`) in a fuzzy-searchable rofi menu.
2. Computes the removal set with `pacman -Rns --print` (read-only, no root —
   `--print` changes nothing).
3. If the package can't be removed cleanly (something depends on it), it shows
   you pacman's error and exits without touching anything.
4. Otherwise it shows a confirmation screen listing every package that will be
   removed. Cancel is the default; only Cancel/Remove are selectable.
5. On Remove, it runs `sudo pacman -Rns <pkg>` in a terminal so you get a sudo
   prompt, see pacman's output, and answer pacman's own `[Y/n]`. The terminal
   stays open until you press a key so you can read the result.

AUR packages need no special handling — `-Rns` removes them identically.

## Dependencies

- **rofi** (`rofi` / `rofi-wayland`) — the menu UI
- **pacman** — the package manager (you have it; this is Arch)
- **bash**
- **a terminal** — kitty, foot, alacritty, or wezterm (auto-detected; see Config)
- **polkit** — *only* if you switch to the commented-out `pkexec` removal path

## Install

Drop the script in your `~/.local/bin` and make it executable:

```sh
install -Dm755 asrt ~/.local/bin/asrt
```

### Hyprland keybind

Add this to your Hyprland config (adjust the combo to taste):

```
bind = SUPER, BACKSPACE, exec, asrt
```

**PATH caveat:** Hyprland's `exec` environment does not always inherit
`~/.local/bin`, so a bare `asrt` can silently do nothing with no error to explain
why. If the keybind seems dead, use the absolute path instead:

```
bind = SUPER, BACKSPACE, exec, /home/YOUR_USER/.local/bin/asrt
```

(or confirm `~/.local/bin` is on the PATH Hyprland was actually started with).

## Usage

Trigger the keybind (or run `asrt`). Type to filter, Enter to pick. Review the
removal list, choose **Remove** or **Cancel**. Escape anywhere exits cleanly
without changing anything.

Test it safely first — `DRY_RUN` prints the removal command instead of running it:

```sh
DRY_RUN=1 asrt
```

## Config

All of it lives in a block at the top of the script:

| Variable               | Default                 | Purpose                                                                 |
|------------------------|-------------------------|-------------------------------------------------------------------------|
| `LAUNCHER`             | `(rofi -dmenu -i)`      | The menu launcher. Swap for `(fuzzel --dmenu)` / walker if you like.    |
| `LIST_CMD`             | `(pacman -Qetq)`        | Packages offered for removal. `pacman -Qeq` also lists depended-upon ones. |
| `TERMINAL_CANDIDATES`  | `($TERMINAL kitty foot alacritty wezterm)` | Terminals tried in order for the removal; first found wins. |
| `DRY_RUN` (env)        | unset                   | When set, print the removal command instead of running it.             |

The script invokes each terminal correctly for its CLI (kitty/foot take the
command directly, alacritty/xterm use `-e`, wezterm uses `start --`). There's
also a commented-out `pkexec` path if you'd rather have a graphical password
prompt and no terminal window.

## Scope

v1 removes exactly one package at a time, using `pacman -Rns` only. Explicitly
out of scope (with comments in the script marking where they'd slot in):
multi-select/batch removal, installing or updating, orphan cleanup, scanning for
leftover config in your home directory, and theming beyond your existing rofi
theme.

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
