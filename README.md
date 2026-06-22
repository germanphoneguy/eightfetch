<div align="center">
  <h1>eightfetch</h1>
  <p><strong>Blazing fast system fetch tool — Rust port of <code>myfetch</code></strong></p>
  <p>
    <a href="#-benchmarks"><strong>benchmarks</strong></a> ·
    <a href="#-install"><strong>install</strong></a> ·
    <a href="#-usage"><strong>usage</strong></a>
  </p>

  <table>
    <tr>
      <td align="left">
        <pre>                    -'                    ╭────────────────────────────────────────╮
                  .o+'                    │ OS: Arch Linux                         │
                 'ooo/                    │ Kernel: Linux 6.18.32-1-lts            │
                '+oooo:                   │ Device: Alienware 16 Aurora AC16250    │
               '+oooooo:                  │ Uptime: 2h 23m                         │
               -+oooooo+:                 │ Packages: 697 (pacman)                 │
             '/:-:++oooo+:                │ Shell: fish                            │
            '/++++/+++++++:               │ DE: Hyprland                           │
           '/++++++++++++++:              │ Terminal: xterm-kitty                  │
          '/+++ooooooooooooo/'            │ Resolution: 2560x1600                  │
         ./ooosssso++osssssso+'           │ CPU: Intel(R) Core(TM) 7 240H          │
        .oossssso-''''/ossssss+'          │ GPU: Intel Graphics                    │
       -osssssso.      :ssssssso.         │ GPU-2: GeForce RTX 5060 Max-Q / Mobile │
      :osssssss/        osssso+++.        │ RAM: 3.6 GiB / 31.0 GiB                │
     /ossssssss/        +ssssooo/-        │ Disk (/): 67GiB / 937GiB (7%)          │
    '/ossssso+/:-        -:/+osssso+-     ╰────────────────────────────────────────╯
   '+sso+:-'                  '.-/+oso:
  '++:.                           '-/+/
  .'                                 '</pre>
      </td>
    </tr>
  </table>
</div>

---

## benchmarks

> tested on Arch, all sysfetches may be slower on other distros

| tool                 | time        | comparision     |
|----------------------|-------------|-----------------|
| **myfetch (C port)** | 725.1 ms    | **213.4x slower**  |
| **fastfetch**        | 27.7 ms     | **8.1x slower** |
| **eightfetch**       | **3.4 ms** | **go brr(sorry my humor is bad)** |

> Since i want to be honest, here is how it performed on Pop!_OS:

| tool                 | time        | comparision     |
|----------------------|-------------|-----------------|
| **fastfetch**        | 50.1 ms     | - |
| **eightfetch**       | **9.4 ms** | **~5.3x faster** |

> atp im just distro flexxing but here is how it performned on NixOS:

| tool                 | time        | comparision     |
|----------------------|-------------|-----------------|
| **myfetch (C port)** | 497.8 ms    | **~3,548x slower** |
| **fastfetch**        | 141.9 ms    | **~1,011x slower** |
| **eightfetch**       | **140.3 µs** | - |


Arch:
```
$ hyperfine myfetch
  Time (mean ± σ):     725.4 ms ± 78.9 ms  [User: 103.9 ms, System: 164.3 ms]

$ hyperfine fastfetch
  Time (mean ± σ):      27.7 ms ±  2.8 ms  [User: 4.3 ms, System: 12.5 ms]

$ hyperfine 8fetch
  Time (mean ± σ):       3.4 ms ±   2.4 ms    [User: 1.8 ms, System: 2.4 ms]
```
NixOS:
```
$ hyperfine myfetch
  Time (mean ± σ):     497.8 ms ± 781.6 ms    [User: 75.5 ms, System: 280.1 ms]

$ hyperfine fastfetch
  Time (mean ± σ):     141.9 ms ±  18.2 ms    [User: 79.0 ms, System: 52.0 ms]

$ hyperfine 8fetch
  Time (mean ± σ):      69.7 µs ± 305.8 µs    [User: 83.5 µs, System: 403.9 µs]

$ hyperfine 8fetch # second test
  Time (mean ± σ):     140.3 µs ± 442.7 µs    [User: 86.9 µs, System: 458.9 µs]
```
> IMPORTANT: these are not copy-pasted from the terminal, i stripped away unnesesary stuff.
> the tool is so fast that, on Arch and Nix, `hyperfine` warns that it cannot calibrate the shell startup time!!

eightfetch is **~213.4x faster** than the original `myfetch(C port)`, and **~8.1x faster** than `fastfetch`; all while using **zero external dependencies** (pure Rust stdlib)!!

---

## install

### Via cargo (from source)

```bash
cargo install eightfetch
```

Then run:

```bash
8fetch
```

### Via AUR

```
yay -S 8fetch
```

### manually (build + copy)

```bash
# clone
git clone https://github.com/germanphoneguy/eightfetch.git
cd eightfetch

# build
cargo build --release

# copy to PATH
cp target/release/8fetch ~/.cargo/bin/8fetch
# or: sudo cp target/release/8fetch /usr/local/bin/8fetch

# run it
8fetch
```

---

## usage

```bash
8fetch

# grey monochrome
8fetch --grey

# custom hex color
8fetch --color:5f5f5f

# machine-readable JSON (zero dep, no extra skips)
8fetch --json
```

---

## how it works

eightfetch avoids subprocess spam(like the deprecated `myfetch(C port)` is doing) by reading directly from sysfs and procfs:

- **OS info** - `/etc/os-release` (one read, 4 fields parsed)
- **Kernel** - `/proc/sys/kernel/osrelease` (file read, no `uname`)
- **GPU** - driver-based sysfs scan via `/sys/bus/pci/drivers/*/` (no `lspci`)
- **CPU** - `/proc/cpuinfo` (already read for virt detection, shared)
- **RAM** - `/proc/meminfo`
- **Disk** - `statvfs()` syscall (no `df` subprocess)
- **Packages** - `readdir("/var/lib/pacman/local/")` (no `pacman -Q`)
- **Shell/Terminal/DE** - Environment variables only
- **Resolution** - `hyprctl monitors` (fastest available for Wayland)

Only **one** subprocess per run: `hyprctl monitors` :)

---

## build requirements

- Rust 1.70+ (edition 2021)
- Linux (uses sysfs, procfs — no Windows/macOS support)

---

## license

MIT
