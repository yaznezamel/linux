# 01 · `drivers/net/dummy.c` walkthrough

The first networking code to read: a complete network driver in 200 lines.
Every packet sent to a dummy device is counted and thrown away. Line numbers
are for v7.3-rc6.

## Read it in this order (bottom-up, the order the kernel runs it)

| Step | Lines | Code | Questions to answer |
| --- | --- | --- | --- |
| 1 | 196-200 | `module_init`, `module_exit`, `MODULE_*` | What runs at `insmod`, what at `rmmod`? |
| 2 | 172-194 | `dummy_init_module()`, `dummy_cleanup_module()` | Why register `rtnl_link_ops` before creating devices? What undoes what? |
| 3 | 152-170 | `dummy_init_one()` | What does `alloc_netdev()` allocate? Why `goto err` and `free_netdev()`? |
| 4 | 103-128 | `dummy_setup()` | What does `ether_setup()` fill in? What do `IFF_NOARP` and `IFF_NO_QUEUE` change? |
| 5 | 89-97 | `dummy_netdev_ops` | Which `ndo_*` callbacks exist? Who calls each one? |
| 6 | 63-70 | `dummy_xmit()` | The whole transmit path of this "hardware". Why must it free the skb? What does `NETDEV_TX_OK` promise? |
| 7 | 57-61, 72-78 | `dummy_get_stats64()`, `dummy_dev_init()` | Where do the numbers in `ip -s link` come from? |
| 8 | 130-146 | `dummy_validate()`, `dummy_link_ops` | How does `ip link add x type dummy` find this driver? |

Keep these open next to it:

- `include/linux/netdevice.h`: `struct net_device_ops` (the comment above it
  documents every `ndo_*` callback) and `struct net_device`
- `include/linux/skbuff.h`: `struct sk_buff`, at least `len`, `data`,
  `protocol`, `dev`
- `Documentation/networking/netdevices.rst`

## See it run

Enable the driver and rebuild:

```sh
cd ~/linux
scripts/config -e DUMMY && make olddefconfig
vng --build
vng
```

Inside the VM (`dummy0` exists because `numdummies` defaults to 1):

```sh
ip link set dummy0 up
ip addr add 10.0.0.1/24 dev dummy0
ping -c 3 -W 1 10.0.0.2          # no replies: dummy_xmit() drops everything
ip -s link show dummy0           # TX: 3 packets, 294 bytes
```

98 bytes per packet: 14 Ethernet header + 20 IP header + 8 ICMP header + 56
bytes of ping payload.

Now see who calls `dummy_xmit()`:

```sh
cd /sys/kernel/tracing
echo dummy_xmit > set_ftrace_filter
echo 1 > options/func_stack_trace
echo function > current_tracer
ping -c 1 -W 1 10.0.0.2 >/dev/null
echo 0 > tracing_on
grep -v '^#' trace
```

Output on this tree:

```text
ping-116  [002] b....  16.541586: dummy_xmit <-dev_hard_start_xmit
 => dummy_xmit
 => dev_hard_start_xmit
 => __dev_queue_xmit
 => ip_finish_output2
 => ip_local_out
 => ip_push_pending_frames
 => raw_sendmsg
 => __sys_sendto
 => __x64_sys_sendto
 => do_syscall_64
 => entry_SYSCALL_64_after_hwframe
```

That is the transmit path from [packet-path.md](packet-path.md), measured.
Notes:

- `ping` run as root uses a **raw** socket, so you see `raw_sendmsg()`
  instead of `udp_sendmsg()`.
- Some functions in packet-path.md (`ip_output()`, `neigh_output()`,
  `xmit_one()`) are missing because the compiler inlined them; they have no
  frame of their own.
- Changing `current_tracer` clears the buffer, so stop with `tracing_on`
  before reading `trace`.

## Exercises

1. Change `dummy_xmit()` to `pr_info()` the packet length and
   `ntohs(skb->protocol)`. Rebuild, ping, read `dmesg`.
2. Remove `dev_kfree_skb(skb)`, ping 1000 times, and watch memory in
   `/proc/meminfo` or `slabtop`. Then put it back. Why is this a leak?
3. Boot with `dummy.numdummies=3` on the kernel command line
   (`vng --append dummy.numdummies=3`) and check `ip link`.
4. Answer every question in the table above in a study note
   ([template](../templates/study-notes.md)).
