# The New Linus: `fs/`

Week 7 (Nov 23-29): files and devices. Lab:
[`labs/07-chardev`](../labs/07-chardev/NEW_LINUS.md). Same week:
[`drivers/char/`](../drivers/char/NEW_LINUS.md).

## What lives here

The VFS (virtual file system) and every filesystem. The VFS is an abstraction
layer: `open()`, `read()` and `write()` work the same on ext4, NFS, `/proc`, a
pipe or a device, because each of them fills in the same ops tables. "Everything
is a file" is a VFS design decision, not a slogan.

Four objects to learn: **superblock** (a mounted filesystem), **inode** (a
file's metadata), **dentry** (a name in a directory, cached), **file** (an open
file: position, flags, ops).

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `include/linux/fs.h` | `struct inode`, `struct file`, `struct file_operations` | The core objects and the ops table a driver fills in for week 7's lab. |
| `include/linux/dcache.h` | `struct dentry` | Name-to-inode cache entry. |
| `include/linux/fs/super_types.h` | `struct super_block` | One per mounted filesystem. |
| `open.c` | `SYSCALL_DEFINE4(openat)` → `do_sys_openat2()` | From a path string to a file descriptor. |
| `namei.c` | `do_file_open()`, `path_openat()`, `link_path_walk()` | Path lookup, one component at a time, first lock-free (RCU-walk), falling back to ref-walk. |
| `read_write.c` | `SYSCALL_DEFINE3(read, ...)` → `ksys_read()` → `vfs_read()` | Look up the `struct file` from the fd, then call its `->read` or `->read_iter`. |
| `ramfs/inode.c` | `ramfs_fill_super()`, `ramfs_get_inode()`, `init_ramfs_fs()` | The smallest real filesystem in the tree: a few hundred lines, all in the page cache. Read all of it. |
| `proc/array.c` | `proc_pid_status()` | What prints `/proc/PID/status`. Files in `/proc` are generated on read. |

## Watch it in the VM

```sh
strace -e trace=openat,read,close cat /etc/hostname
ls -l /proc/self/fd                        # fds of the ls process
stat /etc/hostname                         # inode number, links, blocks
mount | head                               # superblocks, one per line
mkdir /tmp/r && mount -t ramfs none /tmp/r && echo hi > /tmp/r/f && cat /tmp/r/f
```

## Words to own

- **VFS**: the common layer every filesystem plugs into.
- **inode**: per-file metadata (size, owner, mode, block map); no name.
- **dentry**: a directory entry linking a name to an inode; cached in the dcache.
- **superblock**: per-mount filesystem state.
- **file descriptor**: an index into the process's table of `struct file *`.
- **file_operations**: ops table behind `read`, `write`, `ioctl`, `mmap` for a file.
- **page cache / writeback**: file data cached in RAM; dirty pages are written
  back later.
- **RCU-walk**: lock-free path lookup that retries with references on conflict.
- **pseudo filesystem**: `/proc`, `/sys`, `debugfs`; no disk behind them.

## Answer in your notes

1. Two processes open the same file. How many inodes, dentries and
   `struct file`s are there?
2. Why does a hard link not need a new inode?
3. Trace `read()` on `/proc/self/status` from `ksys_read()` to
   `proc_pid_status()`: which `file_operations` sits in between?
