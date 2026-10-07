# Setup: build, boot, navigate

The goal is a fast loop: change a line, rebuild, boot, test, in a few minutes.

## 0. Where to work

The kernel builds on Linux. Work inside a Linux system, on a Linux filesystem:

| Your machine | Use |
| --- | --- |
| Linux | Work directly |
| Windows | WSL2 with Ubuntu; clone into `~/` inside WSL, **not** under `/mnt/c` |
| macOS | A Linux VM (OrbStack, UTM or Lima) with Ubuntu; clone inside the VM's own disk |

Why not a macOS or Windows folder: those filesystems ignore letter case by
default, and the kernel has 13 pairs of files whose names differ only by case,
for example `include/uapi/linux/netfilter/xt_DSCP.h` and `xt_dscp.h`. On such a
disk one silently overwrites the other and `git status` never comes clean.

## 1. Get the code

```sh
git clone --depth=1 --branch claude/sharp-wright-pe6fkj \
    https://github.com/yaznezamel/linux.git ~/linux
cd ~/linux
git switch -c learning          # optional: a shorter local branch name
```

`--depth=1` downloads only the latest snapshot. Fetch the full history later,
when you need `git log`, `git blame` or `Fixes:` tags:
`git fetch --unshallow`.

## 2. Packages (Debian/Ubuntu)

```sh
sudo apt install build-essential flex bison bc libelf-dev libssl-dev \
    dwarves cpio kmod udev iproute2 qemu-system-x86 \
    git git-email clangd universal-ctags cscope pipx
pipx install virtme-ng          # provides the `vng` command
pipx ensurepath                 # then open a new shell
```

`kmod`, `udev` and `iproute2` are already there on a normal Ubuntu install, but
minimal WSL or VM images may lack them; `vng` fails without them (`depmod: not
found`, or exit code 255 with no output).

On an Apple Silicon Mac the Linux VM is arm64: install `qemu-system-arm`
instead of `qemu-system-x86`. `vng` builds and boots an arm64 kernel the same
way.

Minimum tool versions are listed in `Documentation/process/changes.rst`.

## 3. Remotes

```sh
git remote add upstream https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
git remote add net-next https://git.kernel.org/pub/scm/linux/kernel/git/netdev/net-next.git
git fetch upstream               # several GB; not needed until your first patch
```

`origin` is the GitHub fork. Patches for networking are based on `net` (fixes)
or `net-next` (new work), see [contributing.md](contributing.md).

## 4. Build a small kernel and boot it

virtme-ng builds a minimal config and boots the kernel in QEMU, sharing your
host filesystem read-only, so there is no disk image to manage.

```sh
cd ~/linux
vng --build --configitem 'CONFIG_LOCALVERSION="-learning"'
vng                              # boot it; you get a shell inside the VM
uname -r                         # inside the VM: 7.3.0-rc6-learning-virtme
exit                             # powers the VM off
vng -- uname -r                  # or: boot, run one command, power off
```

`vng` appends `-virtme` to the version string. Rebuild with the same
`vng --build ...` command; plain `make` uses a different version string and
recompiles more than it needs to.

Timings checked on this tree (v7.3-rc6, 4 CPU cores, 15 GB RAM, no KVM):

| Step | Time |
| --- | --- |
| First `vng --build` | ~12 min |
| Rebuild after changing one `.c` file | ~1.5 min |
| Rebuild with nothing changed | ~5 s |
| Boot to shell | ~6 s |

If `/dev/kvm` doesn't exist (no hardware virtualization, common in VMs), add
`--disable-kvm` to `vng`. It runs slower but works.

To keep files the VM writes (ftrace output, test logs), share a directory:
`vng --rwdir ~/linux-out`.

Without virtme-ng:

```sh
make defconfig
scripts/config --set-str LOCALVERSION "-learning"
make -j"$(nproc)"
```

## 5. Config options worth turning on while learning

```sh
scripts/config -e DEBUG_INFO_DWARF5 -e KASAN -e DEBUG_ATOMIC_SLEEP \
    -e PROVE_LOCKING -e DYNAMIC_DEBUG -e FUNCTION_TRACER \
    -e FUNCTION_GRAPH_TRACER -e NETDEVSIM -m DUMMY -m VETH
make olddefconfig
```

KASAN and lockdep (`PROVE_LOCKING`) catch memory and locking bugs in your
modules; they slow the kernel down, which is fine in QEMU.

## 6. Build an out-of-tree module against this tree

```make
# Makefile next to hello.c
obj-m += hello.o
```

```sh
make -C /path/to/linux M=$PWD modules
```

Full guide: `Documentation/kbuild/modules.rst`.

## 7. Navigate the code

```sh
make compile_commands.json       # after a build; clangd then resolves every symbol
make cscope tags                 # alternative for vim/emacs
git grep -n "ndo_start_xmit"     # fast, respects .gitignore
```

Online cross-reference: <https://elixir.bootlin.com/linux/latest/source>.

## 8. Debugging tools

| Tool | Use it for | Doc |
| --- | --- | --- |
| `pr_info()` / `pr_debug()` + dynamic debug | Quick prints you can switch on at runtime | `Documentation/admin-guide/dynamic-debug-howto.rst` |
| ftrace `function_graph` | See the call tree of a code path | `Documentation/trace/ftrace.rst` |
| `scripts/decode_stacktrace.sh`, `scripts/faddr2line` | Turn an oops into file:line | `Documentation/admin-guide/bug-hunting.rst` |
| KASAN | Use-after-free, out-of-bounds | `Documentation/dev-tools/kasan.rst` |
| `bpftrace` | Ad-hoc probes without rebuilding | - |
