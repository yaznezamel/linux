# Lab 01: hello

Week 1 (Oct 12-18). Read first: [`init/NEW_LINUS.md`](../../init/NEW_LINUS.md),
[`labs/NEW_LINUS.md`](../NEW_LINUS.md) (setup and build loop).

## Goal

Your first loadable module, and proof that you can build, boot, load and unload
against your own kernel.

## You write

- `hello.c`, `Makefile`
- `sizes.c` (user space, C review)

## Steps

1. `hello.c`: an init function that prints one line with `pr_info()` and an
   exit function that prints another. Mark them `__init` and `__exit`.
2. Add `MODULE_LICENSE("GPL")` and `MODULE_DESCRIPTION()`. Then build once
   without the license line: `modpost` refuses with `missing MODULE_LICENSE()`.
   Build once with `MODULE_LICENSE("Proprietary")`, load it, and read the
   taint message in `dmesg` and the value of `/proc/sys/kernel/tainted`.
3. Add two parameters with `module_param()`: an `int count` and a `charp name`.
   Print `name` `count` times on load.
4. Load with `insmod labs/01-hello/hello.ko count=3 name=linus`, read the value
   back from `/sys/module/hello/parameters/count`, unload.
5. `modinfo labs/01-hello/hello.ko`: find your parameters and description.

## Done when

- [ ] Loads and unloads with no warnings except the expected "out-of-tree module taints kernel".
- [ ] Parameters are visible in `/sys/module/hello/parameters/`.
- [ ] You can explain what "taints kernel" means and why it matters for bug reports.

## Hints

- `include/linux/module.h`: `module_init()`, `module_exit()`, `MODULE_*()`
- `include/linux/moduleparam.h`: `module_param()`, `MODULE_PARM_DESC()`
- Permissions on a parameter (`0644`) decide whether it shows up in sysfs.

## C review: K&R chapters 1-2

`sizes.c`: print `sizeof` for `char`, `short`, `int`, `long`, `long long`,
`void *`, `size_t` on x86-64. Then write `bitcount(unsigned x)` (K&R exercise
2-9: `x &= (x - 1)` deletes the rightmost 1 bit) and test it.
Compile with `gcc -Wall -Wextra -O2`.
