# systemd-functions

Short, memorable shell helpers for everyday **systemd** work — wrappers around
`systemctl`, `journalctl`, and `systemd-analyze` that cut the typing, add `sudo`
automatically when it's needed, and print results in a colorized, paged,
[k9s](https://k9scli.io/)-style layout.

It's a single file you `source` from your shell. No daemon, no dependencies to
install, no config.

```console
$ sclu                    # what services are running, colored by state
$ sclt                    # every timer: next run, last run, what it triggers
$ sctd certbot            # a timer's schedule + the unit it fires
$ scgr ssh                # find any unit by name or description
$ scboot                  # what made the last boot slow
$ sc restart nginx        # auto-sudo; no more "Failed to restart: access denied"
$ schelp                  # a colorized cheat-sheet of everything (alias: sch)
```

## Why use it

`systemctl` is powerful but wordy, and you end up typing the same long
incantations all day. These helpers fix the small papercuts:

- **Less typing.** `sc`, `jc`, `sclu`, `sclt`, … instead of
  `systemctl …`, `journalctl …`, `systemctl list-units …`.
- **Auto-sudo, but only when needed.** `sc restart nginx` elevates itself for
  mutating verbs (`start`/`stop`/`restart`/`enable`/…); read-only verbs
  (`status`/`cat`/`list-*`) stay unprivileged. No more re-running with `sudo`.
- **Readable output.** Long lists are re-tabulated, aligned, clipped to your
  terminal width (no ugly wrapping), and paged — with k9s-style color:
  green = up/healthy, dim = off/idle, **red** = failed, lavender = neutral
  states and time durations.
- **Color that behaves.** Colors appear only on a real terminal. Pipe or
  redirect any command and the output comes out clean — safe for `grep`,
  `awk`, and `>` redirects.
- **Discoverable.** Forget a command? Run `schelp` (or `sch`) for a colorized
  list of every helper plus a `journalctl` flag reference. The top of the
  script is also a thorough cheat-sheet.
- **Mnemonic names.** Each name reads as *SystemCtl + verb*, so the letters tell
  you what it does: `sclt` = **S**ystem**C**tl **L**ist **T**imers,
  `scwh` = **S**ystem**C**tl **WH**ere, etc.

## Install

Clone (or just download the one file) and source it from your shell rc:

```sh
git clone https://github.com/jacobm3/systemd-functions.git ~/.systemd-functions

# then add to ~/.bashrc:
source ~/.systemd-functions/systemd-functions.sh
```

Or drop the single file wherever you like:

```sh
curl -fsSL https://raw.githubusercontent.com/jacobm3/systemd-functions/main/systemd-functions.sh \
  -o ~/.systemd-functions.sh
echo 'source ~/.systemd-functions.sh' >> ~/.bashrc
```

Open a new shell (or `source` it again) and run `schelp` to confirm it loaded.

## Commands

| Command  | Mnemonic              | What it does |
|----------|-----------------------|--------------|
| `sc`     | SystemCtl             | wrapper: `sc status\|start\|stop\|restart\|enable\|cat\|edit <unit>` (auto-sudo for mutating verbs) |
| `sclu`   | SC List-Units         | services in a state (default `running`), colored by state |
| `sclt`   | SC List-Timers        | timers: next/last run + what they activate (`sclt --all` for inactive too) |
| `sctd`   | SC Timer-Detail       | a timer's schedule + the unit it triggers |
| `scwh`   | SC WHere              | path of a unit file + any override drop-ins |
| `scgr`   | SC GRep               | find units by name/description, loaded and on-disk |
| `scboot` | SC BOOT               | boot performance — `blame` (default), `chain`, or `time` |
| `scu`    | SC User               | operate on your `--user` units (never needs sudo) |
| `scen`   | SC ENable (`--now`)   | enable **+** start in one shot |
| `scdis`  | SC DISable (`--now`)  | disable **+** stop in one shot |
| `jc`     | JournalCtl            | logs — `jc -fu <unit>`, `jc -xe`, `jc -b -1`, `jc -p err`, … |
| `schelp` | SC HELP               | colorized list of all of the above (alias: `sch`) |

Run `schelp` for the full reference, including a `journalctl` flag table.

## Requirements

- **bash** (uses bash-isms; not POSIX `sh`)
- **systemd** (`systemctl`, `journalctl`, `systemd-analyze`)
- Standard GNU userland — `sed`, `awk`, `column`, `cut`, `tput`
- `less` for paging (falls back to plain output if absent)
- A truecolor-capable terminal for the lavender accent (most modern terminals)

Tab-completion for `sc` and `jc` is wired up automatically when bash-completion's
`systemctl`/`journalctl` completions are present.

## A note on naming collisions

If you already define aliases like `alias sc='systemctl '`, bash would expand
the alias when these functions are *defined*, turning `sc()` into a function
that calls itself forever (a fork bomb / stack overflow that closes your
terminal). The script defends against this by running `unalias` on its own names
before defining them — so it's safe to source even if you have conflicting
aliases. If you want to keep a conflicting alias under a different name, rename
it before sourcing.

## License

MIT — see [LICENSE](LICENSE).
