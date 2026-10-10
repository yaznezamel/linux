# The New Linus: `include/linux/`

Week 2 (Oct 19-25): C refresher and kernel data structures. Lab:
[`labs/02-list`](../../labs/02-list/NEW_LINUS.md). After week 2 this file is
your index of headers for the rest of the plan.

## What lives here

The kernel's internal API. There is no libc in the kernel, so these headers
replace `<stdlib.h>`, `<string.h>` and friends, and they define the core
structs every subsystem shares. A header is the contract; the `.c` file behind
it is the implementation.

## Week 2: read these first

| Header | Read | What to get out of it |
| --- | --- | --- |
| `container_of.h` | whole file (40 lines) | Pointer arithmetic with `offsetof()` to get from a member back to its struct. |
| `list.h` | `struct list_head`, `list_add()`, `list_del()`, `list_for_each_entry()`, `list_for_each_entry_safe()` | The intrusive doubly linked list used all over the kernel. |
| `types.h` | `u8`/`u32`/`u64`, `bool`, `struct list_head` definition | Fixed-size integer types and why the kernel avoids plain `int` for on-wire or on-disk data. |
| `err.h` | `ERR_PTR()`, `IS_ERR()`, `PTR_ERR()` | Returning an error code inside a pointer. |
| `minmax.h` | `min()`, `max()`, `clamp()` | Type-checked macros; read how they reject mixed signed/unsigned comparisons. |
| `slab.h` | `kmalloc()`, `kzalloc()`, `kfree()` | The kernel's `malloc()`. Note the GFP flags argument. |
| `cleanup.h` | `__free()`, `guard()` | Scope-based cleanup, the kernel's answer to C's lack of destructors. |
| `rbtree.h`, `hashtable.h`, `kfifo.h` | top-of-file comments | Other ready-made data structures; know they exist. |

## Index for later weeks

| Week | Header | Struct or API |
| --- | --- | --- |
| 1 | `init.h`, `module.h`, `printk.h` | `__init`, `module_init()`, `pr_info()` |
| 3 | `sched.h` | `struct task_struct` |
| 3 | `sched/signal.h` | `for_each_process()` |
| 3, 7 | `uaccess.h` | `copy_to_user()`, `copy_from_user()` |
| 5 | `mm_types.h` | `struct mm_struct`, `struct vm_area_struct`, `struct page` |
| 5 | `gfp.h` | `GFP_KERNEL`, `GFP_ATOMIC` |
| 6 | `spinlock.h`, `mutex.h`, `atomic.h`, `refcount.h`, `kref.h` | locking and reference counting |
| 6 | `rcupdate.h` | `rcu_read_lock()`, `rcu_dereference()`, `rcu_assign_pointer()` |
| 6 | `interrupt.h`, `workqueue.h`, `percpu.h` | IRQs, softirqs, deferred work, per-CPU data |
| 7 | `fs.h` | `struct inode`, `struct file`, `struct file_operations` |
| 8 | `skbuff.h` | `struct sk_buff` |
| 8 | `netdevice.h` | `struct net_device`, `struct net_device_ops` |
| 10 | `bpf.h` | BPF programs and maps inside the kernel |

## Words to own

- **intrusive list**: the list node lives inside the object, so one object can
  sit on several lists without extra allocations.
- **container_of()**: recovers the outer struct from a pointer to a member.
- **ERR_PTR**: an errno stored in the top of the pointer range.
- **GFP flags**: tell the allocator what it may do (sleep, do I/O, use reserves).
- **opaque type**: a struct whose fields callers are not supposed to touch.

## Answer in your notes

1. Draw a `struct list_head` embedded in your own struct and compute
   `container_of()` by hand with real offsets (`offsetof()` in a user-space
   program).
2. Why does `list_for_each_entry_safe()` need a second cursor?
3. Why does `min()` refuse to compare an `int` with a `size_t`?
