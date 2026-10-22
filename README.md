# tmh – tmux host layouts

`tmh` opens a predefined combination of hosts in a new tmux session, one pane
per host, arranged in a fixed grid layout. Keyboard input is synchronized
across all panes, so you can type the same commands on several servers at once.

On macOS, `tmh` also resizes the terminal window so the layout fits exactly,
and restores the original size when you leave the session.

```
┌──────────────── stage1 ───────────────┬───────────────── cm2 ────────────────┐
│                                       │                                      │
│                                       │                                      │
├──────────────────────────────── auth1 ───────────────────────────────────────┤
│                                                                              │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Requirements

- bash
- tmux 2.9 or newer (`resize-window` is used)
- awk, ssh
- Optional, for resizing the terminal window: macOS with `osascript`

## Installation

```sh
cp tmh ~/bin/          # or any other directory in your $PATH
chmod +x ~/bin/tmh
mkdir -p ~/.config/tmh
```

## Usage

```sh
tmh <combination>
```

`<combination>` is the name of a file in `~/.config/tmh/`. Set `$TMH_DIR` to use
a different directory. Running `tmh` without arguments (or with `-h`/`--help`)
lists the known combinations.

Each call starts a **new** session named after the combination. If a session
of that name already exists, a suffix is added: `prod`, `prod-2`, `prod-3`, …

When run inside tmux, `tmh` switches the current client to the new session
instead of attaching, and does not touch the terminal size.

## Configuration

Each combination is a plain text file with one line per pane:

```
<host>  <column>,<row>,<width>,<height>
```

Example `~/.config/tmh/prod`:

```
# two panes side by side, one wide pane below
stage1  0,0,80,15
cm2     1,0,80,15
auth1   0,1,161,15
```

Rules:

- `<column>,<row>` are positions in the grid (`0,0` = top left), not character
  coordinates.
- `<width>` and `<height>` are given in characters.
- All panes in a row get the largest height specified in that row.
- If a row is narrower than the widest row, its last pane is widened to fill
  the gap.
- There is a 1-character border between panes, so two 80-column panes side by
  side take 80 + 1 + 80 = 161 columns.
- The host `-` opens a local shell instead of an ssh connection.
- Blank lines and lines starting with `#` are ignored.

The host is passed to `ssh` as is, so anything from your `~/.ssh/config`
(aliases, `user@host`, …) works.

## Behavior

- **Synchronized input.** `synchronize-panes` is switched on, so every
  keystroke goes to all panes. Turn it off with
  `tmux setw synchronize-panes off` (or your own key binding) when you want to
  work in a single pane.
- **Fixed layout.** The window is created with exactly the size the layout
  needs. If the terminal could be resized to match, later manual resizes scale
  the layout as usual (`window-size latest`); otherwise the window keeps its
  fixed size and a warning is printed if parts will be cut off.
- **Terminal resizing (macOS).** `tmh` first tries the `CSI 8 t` escape
  sequence (works in iTerm2, Terminal.app and xterm). If the terminal ignores it
  (e.g. Ghostty), it queries the text area and cell size and resizes the front
  window via System Events. This fallback requires the terminal app to have the
  *Accessibility* permission (System Settings → Privacy & Security →
  Accessibility).
- **Restore.** After you detach or the session ends, the terminal is resized
  back to its original size.

## Author

Written by **Claude** (Anthropic's AI assistant) in collaboration with
Stephan Hradek.
