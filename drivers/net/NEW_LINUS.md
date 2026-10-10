# The New Linus: `drivers/net/`

Week 8 (Nov 30-Dec 6): networking I, devices and buffers. Lab:
[`labs/08-mynic`](../../labs/08-mynic/NEW_LINUS.md). Same week:
[`net/core/`](../../net/core/NEW_LINUS.md).

## What lives here

Network device drivers. Real NICs live in `ethernet/<vendor>/`; the files at the
top level are virtual devices, which are the right place to learn because there
is no hardware to get in the way. Every driver does the same three things:
allocate and register a `struct net_device`, fill in `struct net_device_ops`,
and move `struct sk_buff`s in and out.

Detailed walkthrough with a traced experiment:
[`docs/networking/01-dummy-driver.md`](../../docs/networking/01-dummy-driver.md).

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `dummy.c` | whole file (200 lines) | A complete driver. `dummy_xmit()` counts the packet and frees it. |
| `loopback.c` | `loopback_xmit()` | Transmit turned into receive: the skb goes straight back up via `__netif_rx()`. This is `lo`. |
| `veth.c` | `veth_xmit()` | A pair of devices wired together: transmit on one end is receive on the other. Every container's network interface is one end of a veth pair. |
| `netdevsim/netdev.c` | top of file | A simulated device used by upstream selftests to exercise driver APIs without hardware. |
| `tun.c` | top-of-file comment | TUN/TAP: a device whose "wire" is a file descriptor in user space (VPNs, VMs). Skim. |

## Watch it in the VM

```sh
ip -d link show lo                          # -d shows the device type
ip link add v0 type veth peer name v1
ip link set v0 up; ip link set v1 up
ip -s link show v0                          # counters come from ndo_get_stats64
ethtool -i v0                               # driver: veth
```

`ethtool` is not installed on Mint by default: `sudo apt install ethtool` on
the host, and `vng` sees it.

## Words to own

- **net_device**: kernel object for a network interface.
- **ndo_start_xmit**: the driver callback that receives packets to send.
- **NETDEV_TX_OK / NETDEV_TX_BUSY**: "taken" vs "queue full, retry later".
- **veth pair**: two virtual devices connected back to back.
- **loopback**: device whose transmit is its own receive.
- **TUN/TAP**: virtual device backed by a user-space file descriptor (L3/L2).
- **offload**: work the NIC does instead of the CPU (checksums, segmentation).

## Answer in your notes

1. Why does `loopback_xmit()` call `skb_orphan()` before handing the skb back?
2. In `veth_xmit()`, find where the peer device is looked up. What happens if the
   peer is down?
3. Which `features` does `dummy_setup()` set, and what does each one promise
   the stack?
