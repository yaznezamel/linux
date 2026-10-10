# The New Linus: `mm/`

Week 5 (Nov 9-15): memory management. Lab:
[`labs/05-mm`](../labs/05-mm/NEW_LINUS.md)

## What lives here

Physical page allocation, the slab allocators behind `kmalloc()`, virtual
address spaces, page faults, the page cache, reclaim and the OOM killer. Two
views to keep apart: **physical memory** (pages, zones, the buddy allocator)
and **virtual memory** (each process's address space, made of VMAs and backed
by page tables).

Skim `Documentation/mm/index.rst` first; don't study it.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `include/linux/mm_types.h` | `struct page`, `struct vm_area_struct`, `struct mm_struct` | One `struct page` per physical page; one `mm_struct` per process; one VMA per mapping (a line in `/proc/PID/maps`). |
| `page_alloc.c` | `__alloc_pages_noprof()`, `__rmqueue_smallest()` | The buddy allocator: free lists per order (1, 2, 4, ... pages), split a bigger block when the right size is empty. |
| `slub.c` | `kmem_cache_alloc_noprof()`, `__kmalloc_noprof()`, `kfree()` | SLUB carves pages into same-size objects. `kmalloc(64)` comes from the `kmalloc-64` cache. |
| `slab_common.c` | `__kmem_cache_create_args()` | How a named cache (seen in `/proc/slabinfo`) is created. |
| `mmap.c` | `do_mmap()` | `mmap()` only creates a VMA; no memory is allocated yet. |
| `memory.c` | `handle_mm_fault()`, `do_anonymous_page()`, `do_wp_page()` | Memory is allocated on first touch, in the page fault handler. `do_wp_page()` is the copy-on-write you met in week 3. |
| `filemap.c` | top-of-file comment | The page cache: file contents kept in RAM and shared by every reader. |
| `vmscan.c` | top-of-file comment | Reclaim and `kswapd`: getting memory back when it runs low. |
| `oom_kill.c` | `out_of_memory()`, `oom_badness()` | When reclaim fails: how a victim is scored and killed. |

The `_noprof` suffix comes from memory allocation profiling (6.10): the
public names like `kmalloc()` are macros that record the caller and call these.

## Watch it in the VM

```sh
vng -m 1G
cat /proc/buddyinfo                       # free blocks per order, per zone
grep -E 'MemTotal|MemFree|Cached|Slab' /proc/meminfo
sort -k2 -n -r /proc/slabinfo | head      # biggest slab caches (as root)
cat /proc/self/maps                       # VMAs of the cat process
ps -o min_flt,maj_flt,comm -p $$       # page faults of this shell
```

## Words to own

- **page**: the unit of memory management, 4 KiB on x86-64.
- **buddy allocator**: hands out physically contiguous blocks of 2^order pages
  and merges freed neighbours ("buddies").
- **slab / SLUB**: object caches on top of the buddy allocator.
- **VMA**: a contiguous virtual range with one set of permissions and backing.
- **page fault**: CPU exception on a missing or forbidden mapping; minor if the
  page is already in memory, major if it needs I/O.
- **demand paging**: allocate on first access, not at `mmap()` or `malloc()`.
- **page cache**: file data cached in RAM.
- **TLB**: CPU cache of virtual-to-physical translations; a TLB shootdown
  flushes it on other CPUs.
- **OOM killer**: last resort that kills a process to free memory.
- **overcommit**: Linux lets processes reserve more virtual memory than exists.

## Answer in your notes

1. `malloc(1 GiB)` succeeds on a VM with 1 GiB of RAM. Why, and when does it
   actually fail?
2. Which line in `/proc/buddyinfo` changes when you load a module that does
   `alloc_pages(GFP_KERNEL, 4)`?
3. What makes a page fault "major"?
