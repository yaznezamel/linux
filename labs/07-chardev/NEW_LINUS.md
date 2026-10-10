# Lab 07: chardev

Week 7 (Nov 23-29). Read first: [`fs/NEW_LINUS.md`](../../fs/NEW_LINUS.md),
[`drivers/char/NEW_LINUS.md`](../../drivers/char/NEW_LINUS.md)

## Goal

A device file of your own, `/dev/newlinus`, that stores what you write and
returns it when you read.

## You write

- `newlinus.c`, `Makefile` (module)
- `stdio_vs_sys.c` (user space, C review)

## Steps

1. Register a misc device named `newlinus` with a dynamic minor and your own
   `struct file_operations` (`.owner`, `.read`, `.write`).
2. `write`: copy up to 256 bytes from user space into a static buffer with
   `copy_from_user()`, remember the length, return the number of bytes taken.
3. `read`: return the stored bytes, honouring `*ppos` so that `cat` stops
   (return 0 at the end). Write the offset handling yourself once, then replace
   it with `simple_read_from_buffer()` and compare.
4. Protect the buffer with a `DEFINE_MUTEX()`; reads and writes can come from
   different processes at the same time.
5. Test:
   ```sh
   echo hello > /dev/newlinus
   cat /dev/newlinus
   strace -e trace=openat,read,write cat /dev/newlinus
   ls -l /dev/newlinus; grep newlinus /proc/misc
   ```
6. Stretch: add `.unlocked_ioctl` with one command that clears the buffer, and a
   tiny user-space program that calls it.

## Done when

- [ ] `echo` then `cat` round-trips, including writes longer than 256 bytes
      (truncated, not crashing).
- [ ] Passing a bad pointer (write a small C program that calls
      `write(fd, (void *)1, 10)`) returns `-EFAULT` instead of crashing.
- [ ] You can trace a `cat /dev/newlinus` from `ksys_read()` to your `.read`.

## Hints

- `include/linux/miscdevice.h`, `include/linux/fs.h`, `include/linux/uaccess.h`
- `drivers/char/mem.c` for how small a driver can be.

## C review: K&R chapters 7 vs 8

`stdio_vs_sys.c`: write 10,000 single bytes to a file twice, once with
`fputc()` and once with `write()`. `strace -c` both. Buffering explains the
difference in syscall count.
