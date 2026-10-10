# Lab 02: list

Week 2 (Oct 19-25). Read first:
[`include/linux/NEW_LINUS.md`](../../include/linux/NEW_LINUS.md),
[`lib/NEW_LINUS.md`](../../lib/NEW_LINUS.md)

## Goal

Use the kernel's intrusive linked list, and prove you understand
`container_of()` by rebuilding it in user space.

## You write

- `ulist.c` (user space, do this first)
- `list.c`, `Makefile` (module)

## Steps

1. `ulist.c`: define your own `struct list_head`, `list_add_tail()`, `list_del()`
   and a `container_of()` macro using `offsetof()` from `<stddef.h>`. Store
   five `struct item { int id; char name[16]; struct list_head node; }` in a
   list, walk it, print it, free it. Run it under `valgrind` (`sudo apt install valgrind`) and get zero leaks.
2. `list.c` module: parameter `n` (default 10). On load, `kzalloc()` `n` items,
   give each an id and name, `list_add_tail()` them onto a static `LIST_HEAD`.
3. Walk with `list_for_each_entry()` and print each item.
4. Delete every item with an odd id using `list_for_each_entry_safe()` and
   `list_del()` + `kfree()`. Print the list again.
5. On unload, free everything left.

## Done when

- [ ] `ulist.c` is valgrind-clean.
- [ ] Module loads with `n=1000`, unloads, and `dmesg` shows no warnings.
- [ ] You can draw the memory layout of one `struct item` with real offsets.

## Hints

- `include/linux/list.h`, `include/linux/slab.h`
- What happens if you use `list_for_each_entry()` (not `_safe`) while deleting?
  Try it in `ulist.c`, not in the kernel.

## C review: K&R chapters 5-6

Pointers, arrays, pointer arithmetic, structs. Do exercises 5-3 (`strcat` with
pointers) and 6-4 (sort words by frequency) or equivalents.
