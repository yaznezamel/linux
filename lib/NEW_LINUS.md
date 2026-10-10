# The New Linus: `lib/`

Week 2 (Oct 19-25): C refresher and kernel data structures. Lab:
[`labs/02-list`](../labs/02-list/NEW_LINUS.md)

## What lives here

The kernel's own standard library: string functions, sorting, data structures,
`printf` formatting, checksums, compression. Nothing here is linked against
glibc; the kernel carries its own implementation of everything it needs. This is
the best folder in the tree for C practice, because most files are short,
self-contained and have no hardware in them.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `lib/string.c` | `strlen()`, `strcpy()`, `strcmp()` | Plain C you already know, written the kernel way. Compare with your own versions. |
| `lib/list_sort.c` | `list_sort()` | Merge sort on a `struct list_head` list without allocating memory. Hard to read; worth it. |
| `lib/rbtree.c` | `rb_insert_color()`, `rb_next()` | Red-black tree rebalancing. The scheduler keeps runnable tasks in one (week 4). |
| `lib/kfifo.c` | `__kfifo_in()` | Lock-free single-producer/single-consumer ring buffer using power-of-two masking. |
| `lib/vsprintf.c` | `pointer()` and the `fmt[0] == 'I'` branch | How `printk("%pI4", &addr)` prints an IPv4 address. Kernel `printf` has extensions user space does not. |
| `lib/maple_tree.c` | top-of-file comment only | The B-tree that has stored process memory areas (VMAs) since Linux 6.1. You'll meet it in week 5. |

## Watch it in the VM

There is nothing to run for most of `lib/`; the practice is in user space:

```sh
# copy lib/string.c's strlen into a user-space file, add a main(), compile:
gcc -O2 -Wall -o mystrlen mystrlen.c && ./mystrlen
```

## Words to own

- **red-black tree**: self-balancing binary search tree; O(log n) insert and
  delete with at most a few rotations.
- **maple tree**: range-based B-tree used for VMAs, safe for RCU readers.
- **ring buffer (kfifo)**: fixed-size circular queue; head and tail wrap with a
  mask because the size is a power of two.
- **freestanding C**: C without the hosted standard library, which is what the
  kernel is.

## Answer in your notes

1. Why does `kfifo` insist on a power-of-two size?
2. Why can `list_sort()` not call `kmalloc()`?
3. Find two places in `net/` that use `%pI4` or `%pM` in a `pr_*()` call
   (`git grep -n '%pI4' net/ | head`).
