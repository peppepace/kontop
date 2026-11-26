# kontop

An interactive, htop-style monitor for your containers, written as a single
bash script. It works with **docker** and **podman**, on Linux and macOS, and
runs on the stock macOS bash (3.2) with no extra packages.

```
CPU [||||||                                                          4.1%]
MEM [||||||||||||||||||||                                           31.6%]
 docker   Containers: 5/9   Sort: NAME ▲   Cores: 8   [active only]   [groups]
 NAME▲               STATE      HEALTH       CPU%    MEM% PORTS           UPTIME
 ▾ shop  (4/4 active)
   shop-api          running    healthy     2.31%   6.20% 8080                2h
   shop-db           running    healthy     0.84%  12.75% 127.0.0.1:5432      2h
   shop-cache        running    -           0.40%   0.31% 6379                2h
   shop-web          running    -           0.02%   1.10% 443,8081→80         2h
 ▾ (no project)  (1/1 active)
   mailhog           running    -           0.00%   0.13% 1025,8025           5d
? Help  q Quit  / Filter  l Log  s Shell  S Start/Stop  r Restart  x Kill
```

## Why another container monitor?

Tools like lazydocker, ctop, oxker and podman-tui are excellent, but each is a
binary to download and install, or needs a language runtime. This one is a
**single bash file with nothing to install**:

- **Runs where you cannot install anything.** Copy one file to a server, a
  jump host or a locked-down machine and run it. All it needs is bash 3.2 and
  the tools already there (`awk`, `sed`, `sort`, `tput`).
- **No daemon socket, no API client.** It only calls the `docker` or `podman`
  command line, so it works with whatever context, remote host and permissions
  your CLI already has.
- **One script for both engines.** The same file drives docker and podman.
- **Readable and hackable.** It is plain shell: you can read it before running
  it and adapt it in minutes.

## Features

- Live CPU and memory meters, plus per-container CPU%, MEM%, memory usage,
  PIDs, network and block I/O.
- Published ports in compact form, health status, uptime and image name.
- Sort by name, CPU or memory; the cursor follows the container when the list
  is re-sorted.
- Filter by container or project name as you type.
- Group containers by compose project and start, stop or restart a whole stack.
- Start, stop, restart, pause, kill and remove containers, each with a
  confirmation prompt.
- Live logs, logs in `less` (scroll and search), `inspect`, and a shell inside
  the container.
- Flicker-free: only the lines that changed are redrawn.

## Requirements

- bash 3.2 or later
- docker or podman
- `tput` (ncurses), `awk`, `sed`, `sort`, `cut`, `mkfifo`
- optional: `less`, for scrollable logs and inspect

All of these are part of a standard Linux or macOS installation.

## Installation

Clone the repository and put the script on your `PATH`:

```sh
git clone https://github.com/peppepace/kontop.git
cd kontop
sudo install -m 755 kontop /usr/local/bin/kontop
```

Or just copy the single `kontop` file wherever you need it and run it.

## Usage

```sh
kontop                            # docker if available, otherwise podman
CONTAINER_ENGINE=podman kontop    # force an engine
```

If you use both engines, a symbolic link whose name starts with `p` prefers
podman: `ln -s kontop ptop`, then run `ptop`.

With rootless podman only the current user's containers are visible; run the
script with `sudo` to see root's containers.

### Environment variables

| Variable | Meaning |
|---|---|
| `CONTAINER_ENGINE` | Engine to use, regardless of the script name (e.g. `CONTAINER_ENGINE=podman kontop`). |
| `KONTOP_SORT` | Initial sort key: `name` (default), `cpu` or `mem`. |

### Keys

| Key | Action |
|---|---|
| `↑` `↓` / `k` `j` | Previous / next row |
| `PgUp` `PgDn` | Page up / down |
| `g` `G` / `Home` `End` | Top / bottom of the list |
| `c` `m` `n` | Sort by CPU / memory / name; press the same key again to reverse |
| `/` | Filter by name or project (`Enter` applies, `ESC` cancels or clears) |
| `a` | Show / hide stopped containers |
| `t` | Group by compose project |
| `l` | Live logs (`q` or `ESC` to go back) |
| `L` | Logs in `less`, to scroll and search (`q` to go back) |
| `s` | Shell in the container (bash if available, otherwise sh) |
| `i` | Inspect |
| `S` | Start / stop |
| `p` | Pause / unpause |
| `r` | Restart |
| `x` | Kill |
| `d` | Remove (forced, after a second confirmation, if the container is active) |
| `?` `h` | Help |
| `q` | Quit |

Every action that changes a container asks for confirmation first.

### Defaults

At startup the list is grouped by compose project, shows only the active
containers and is sorted by name. Press `t` and `a` to change the first two.
While stopped containers are hidden, a project with no active container is not
listed: press `a` to see it.

### Groups

Containers are grouped by the `com.docker.compose.project` label. On a group
row, `S` starts or stops and `r` restarts every container of the project. The
commands are sent straight to the engine, so the `depends_on` order of the
compose file is **not** honoured.

### Columns

`NAME STATE HEALTH CPU% MEM% PORTS UPTIME MEM USAGE PIDS IMAGE NET I/O BLOCK I/O`

The first five are always shown; the others appear as the terminal gets wider.

- **PORTS** lists only the ports published on the host: `8443` (same port in
  the container), `8080→80` (different port), `127.0.0.1:8025` (bound to that
  address only).
- **IMAGE** is the last component of the image name, without registry or digest.
- The CPU meter is scaled on the number of CPUs reported by the engine, which
  on Docker Desktop and podman machine is the VM's, not the host's.

## Notes

- **Logs and escape sequences.** Live logs are filtered before reaching the
  terminal: colours are kept, every other escape sequence or control character
  is removed, so log content cannot alter the terminal. `less` is run with `-R`
  for the same reason.
- **Inspect** shows the container's configuration, including its environment
  variables, which may contain secrets. Nothing is written to disk.
- **Temporary files.** The monitor keeps a small snapshot file in a private
  directory (mode 0700, created with `mktemp -d`) under `$XDG_RUNTIME_DIR` or
  `$TMPDIR`, and removes it on exit.
- **bash 3.2.** The screen is refreshed once per second instead of ten times
  per second, and the filter accepts ASCII characters only.

## Credits

Written by Giuseppe Pace, with the assistance of
[Claude](https://claude.com/claude-code) (Anthropic).

## License

[MIT](LICENSE)
