# C in the kernel

The kernel is written in C11 with GNU extensions (`-std=gnu11`), without the C
standard library. These are the patterns you will meet in almost every file.
Each one points to a real place in this tree so you can read it in context.

Style rules: `Documentation/process/coding-style.rst`. Read it once now and
again before your first patch.

## 1. Ops tables: function pointers as interfaces

C has no classes. The kernel gets polymorphism from structs of function
pointers. A driver fills one in, and the core calls through it.

```c
/* drivers/net/dummy.c */
static const struct net_device_ops dummy_netdev_ops = {
	.ndo_init		= dummy_dev_init,
	.ndo_start_xmit		= dummy_xmit,
	.ndo_get_stats64	= dummy_get_stats64,
	/* ... */
};
```

When the stack transmits on any device it calls `dev->netdev_ops->ndo_start_xmit()`,
without knowing which driver is behind it. The same pattern appears as
`file_operations` (files), `proto_ops` (sockets) and `ethtool_ops`.

Also note the **designated initializers** (`.field = value`): fields left out
are zero.

**Practice:** find `struct proto_ops inet_dgram_ops` with `git grep` and list
which functions a UDP socket uses for `sendmsg` and `recvmsg`.

## 2. `container_of()`: from a member back to its struct

```c
/* include/linux/container_of.h */
#define container_of(ptr, type, member) ({ ...
	(type *)((void *)(ptr) - offsetof(type, member)); })
```

Given a pointer to a field inside a struct, subtract the field's offset to get
the struct itself. Generic code passes around pointers to embedded members
(`struct list_head`, `struct kref`, `struct work_struct`), and the owner uses
`container_of()` to get back to its own type.

**Practice:** draw the memory layout of a struct with an embedded
`struct list_head` and compute `container_of()` by hand.

## 3. Intrusive linked lists

```c
/* include/linux/list.h */
#define list_for_each_entry(pos, head, member)				\
	for (pos = list_first_entry(head, typeof(*pos), member);	\
	     !list_entry_is_head(pos, head, member);			\
	     pos = list_next_entry(pos, member))
```

The list node lives **inside** your struct, so one object can be on several
lists and no extra allocation is needed. `list_for_each_entry` is
`container_of()` in a loop. The `_safe` variant lets you delete while
iterating.

## 4. Error handling with `goto`

```c
/* drivers/net/dummy.c: dummy_init_one() */
	dev_dummy = alloc_netdev(0, "dummy%d", NET_NAME_ENUM, dummy_setup);
	if (!dev_dummy)
		return -ENOMEM;

	err = register_netdev(dev_dummy);
	if (err < 0)
		goto err;
	return 0;

err:
	free_netdev(dev_dummy);
	return err;
```

Functions return `0` on success and a **negative errno** on failure. Cleanup
labels run in reverse order of setup, so every exit path frees exactly what was
allocated.

Newer code can also use scope-based cleanup from `include/linux/cleanup.h`
(`__free()`, `guard()`), which frees or unlocks automatically when a variable
goes out of scope.

## 5. Error pointers

`include/linux/err.h`: a function that returns a pointer can encode an errno in
it. Check with `IS_ERR(p)`, extract with `PTR_ERR(p)`, create with
`ERR_PTR(-EINVAL)`. Know which convention a function uses (NULL or ERR_PTR)
before checking its result.

## 6. Section and hint annotations

| Annotation | Meaning |
| --- | --- |
| `__init`, `__exit` | Code only needed at load/unload; `__init` memory is freed after boot |
| `__user` | Pointer into user space; never dereference it, use `copy_from_user()` / `copy_to_user()` |
| `__rcu` | Pointer protected by RCU; access with `rcu_dereference()` |
| `__read_mostly` | Put in a cache-friendly section for rarely written data |
| `likely()`, `unlikely()` | Branch prediction hints |
| `READ_ONCE()`, `WRITE_ONCE()` | Stop the compiler from tearing or caching a shared access |

`sparse` (`make C=1`) checks `__user` and `__rcu` misuse:
`Documentation/dev-tools/sparse.rst`.

## 7. Macros and GNU extensions

- Statement expressions `({ ... })` let a macro return a value
  (`container_of` above).
- `typeof()` lets macros work for any type.
- `static inline` functions in headers replace many macros; prefer them when
  writing your own code.
- Bitfields and careful field ordering in `struct sk_buff`
  (`include/linux/skbuff.h`) keep hot fields in the same cache line.

## 8. Memory

- `kmalloc(size, GFP_KERNEL)` may sleep; use `GFP_ATOMIC` in interrupt or
  softirq context and under spinlocks.
- `kzalloc()` zeroes the memory. `kfree()` accepts NULL.
- Reference counting: `refcount_t` and `struct kref`, never a plain `int`.

## Checklist for every file you read

- [ ] Which ops table(s) does it fill in, and who calls them?
- [ ] Where does each object get allocated and freed?
- [ ] Which lock protects each shared field?
- [ ] What context does each function run in (process, softirq, hardirq)?
