# Lab 05: mm

Week 5 (Nov 9-15). Read first: [`mm/NEW_LINUS.md`](../../mm/NEW_LINUS.md)

## Goal

Watch demand paging happen, and allocate memory the two kernel ways: from a
slab cache and straight from the buddy allocator.

## You write

- `faults.c` (user space)
- `mm.c`, `Makefile` (module)

## Steps

1. `faults.c`: `mmap()` 100 MiB anonymous memory. Print the minor fault count
   from `getrusage(RUSAGE_SELF)` (`ru_minflt`) before touching it, after
   writing one byte per 4 KiB page, and after writing every byte again.
2. Run it again with `MAP_POPULATE` added to the `mmap()` flags. Where did the
   faults go?
3. `mm.c` module, part 1: create a cache with `kmem_cache_create()` named
   `newlinus_obj` for a 200-byte struct, allocate 1000 objects on load, free
   them and destroy the cache on unload. Find your cache in `/proc/slabinfo`
   while loaded: how many objects fit per slab?
4. Part 2: `alloc_pages(GFP_KERNEL, 4)` on load, `__free_pages()` on unload.
   Compare `/proc/buddyinfo` before and after loading.

## Done when

- [ ] You can explain each fault count `faults.c` prints.
- [ ] `/proc/slabinfo` shows `newlinus_obj` with 1000 active objects while
      loaded and nothing after unload.
- [ ] You can say how many bytes order 4 is, and which `buddyinfo` column moved.

## Hints

- `include/linux/slab.h`: `kmem_cache_create()`, `kmem_cache_alloc()`,
  `kmem_cache_free()`, `kmem_cache_destroy()`
- `include/linux/gfp.h`: `alloc_pages()`, `__free_pages()`
- `sys/resource.h` in user space for `getrusage()`

## C review: K&R 8.7, a storage allocator

Type in K&R's `malloc()`/`free()` and step through it in `gdb` once. It is a
free-list allocator, a user-space cousin of what SLUB does per cache.
