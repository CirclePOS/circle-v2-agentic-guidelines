# TMUX Commands Cheat Sheet

Default ctrl-b: `Ctrl-b`

## Start and Attach

| Command | What it does |
| --- | --- |
| `tmux` | Start a new unnamed session |
| `tmux new -s work` | Start a new named session |
| `tmux ls` | List sessions |
| `tmux attach -t work` | Attach to a named session |
| `tmux kill-session -t work` | Kill a named session |
| `tmux kill-server` | Stop all tmux sessions |

## Sessions

| Keys | What it does |
| --- | --- |
| `Ctrl-b d` | Detach from current session |
| `Ctrl-b $` | Rename current session |
| `Ctrl-b s` | Show session list and switch |
| `Ctrl-b (` | Previous session |
| `Ctrl-b )` | Next session |

## Windows

| Keys | What it does |
| --- | --- |
| `Ctrl-b c` | Create a new window |
| `Ctrl-b ,` | Rename current window |
| `Ctrl-b w` | Show window list |
| `Ctrl-b n` | Next window |
| `Ctrl-b p` | Previous window |
| `Ctrl-b 0` through `Ctrl-b 9` | Jump to a specific window number |
| `Ctrl-b &` | Kill current window |

## Panes

| Keys | What it does |
| --- | --- |
| `Ctrl-b %` | Split pane vertically |
| `Ctrl-b "` | Split pane horizontally |
| `Ctrl-b o` | Move to next pane |
| `Ctrl-b ;` | Toggle to previous pane |
| `Ctrl-b x` | Kill current pane |
| `Ctrl-b z` | Toggle pane zoom |
| `Ctrl-b q` | Show pane numbers briefly |
| `Ctrl-b {` | Move pane left |
| `Ctrl-b }` | Move pane right |
| `Ctrl-b !` | Break pane into a new window |

## Resize Panes

| Keys | What it does |
| --- | --- |
| `Ctrl-b Ctrl-Left` | Resize pane left |
| `Ctrl-b Ctrl-Right` | Resize pane right |
| `Ctrl-b Ctrl-Up` | Resize pane up |
| `Ctrl-b Ctrl-Down` | Resize pane down |
| `Ctrl-b Alt-1` through `Alt-5` | Select preset layouts |
| `Ctrl-b Space` | Cycle pane layouts |

## Copy Mode and Scrollback

| Keys | What it does |
| --- | --- |
| `Ctrl-b [` | Enter copy mode |
| `q` | Exit copy mode |
| `Space` | Start selection in copy mode |
| `Enter` | Copy selection in copy mode |
| `PgUp` or `Fn-Up` | Page up in copy mode or terminal scrollback |
| `/text` | Search forward in copy mode |
| `?text` | Search backward in copy mode |
| `n` | Next search match |
| `N` | Previous search match |

## Useful Navigation

| Keys | What it does |
| --- | --- |
| `Ctrl-b Arrow Key` | Move between panes |
| `Ctrl-b :` | Open tmux command prompt |
| `Ctrl-b t` | Show a clock |
| `Ctrl-b ?` | Show all key bindings |

## Frequent Command Prompt Actions

Open the tmux command prompt with `Ctrl-b :`, then run:

| Command | What it does |
| --- | --- |
| `setw synchronize-panes on` | Send the same input to all panes in the window |
| `setw synchronize-panes off` | Turn synchronized input off |
| `source-file ~/.tmux.conf` | Reload tmux config |
| `rename-window api` | Rename current window |
| `rename-session ops` | Rename current session |

## Common Workflow

1. `tmux new -s project`
2. `Ctrl-b c` to create windows for app, tests, and logs
3. `Ctrl-b %` or `Ctrl-b "` to split panes as needed
4. `Ctrl-b d` when you want to leave everything running
5. `tmux attach -t project` to resume later

## Notes

- `Ctrl-b` means press `Ctrl-b`, release, then press the next key.
- Some terminals use `Option` instead of `Alt` on macOS.
- If pane resize bindings do not work in your terminal, use `Ctrl-b :` and the `resize-pane` command instead.