# The New Linus: `drivers/char/`

Week 7 (Nov 23-29): files and devices. Lab:
[`labs/07-chardev`](../../labs/07-chardev/NEW_LINUS.md)

## What lives here

Character device drivers: devices you read and write as a stream of bytes
through a file in `/dev`. The driver fills in a `struct file_operations`, and
the VFS from week 7 routes `read()`/`write()` on the device file to it.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `mem.c` | `read_null()`, `write_null()`, `null_fops` | `/dev/null` in full: `read` returns 0 (end of file), `write` pretends everything was written. |
| `mem.c` | `read_iter_zero()` | `/dev/zero`: fill the user buffer with zeros. |
| `mem.c` | `devlist[]` | The table that creates `/dev/null`, `/dev/zero`, `/dev/random`, ... with their minor numbers. |
| `misc.c` | `misc_register()` | The easy way to get a device node: one shared major number (10), a dynamic minor, the node appears automatically. Your lab uses this. |
| `include/linux/miscdevice.h` | `struct miscdevice` | `name`, `minor`, `fops`: all you need to fill in. |

## Watch it in the VM

```sh
ls -l /dev/null /dev/zero                  # "c" = character device, major,minor
cat /proc/devices | head                   # major numbers by driver
cat /proc/misc                             # misc devices and their minors
head -c 16 /dev/zero | od -x
```

## Words to own

- **character device**: byte-stream device accessed through `file_operations`.
- **major/minor number**: driver id / instance id of a device node.
- **device node**: the file in `/dev` that names a major/minor pair.
- **misc device**: helper class for simple char devices (major 10).
- **`copy_to_user()` / `copy_from_user()`**: the only safe way to touch user
  memory; they handle faults on bad pointers.
- **ioctl**: device-specific control call outside read/write.

## Answer in your notes

1. Why does `write_null()` return the count instead of 0?
2. What happens if a driver dereferences a `__user` pointer directly instead of
   using `copy_from_user()`?
3. What minor number does your lab device get, and where did it come from?
