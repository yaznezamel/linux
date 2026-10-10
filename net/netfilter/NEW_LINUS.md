# The New Linus: `net/netfilter/`

Week 9 (Dec 7-13): networking II, the stack and netfilter. Lab:
[`labs/09-netfilter`](../../labs/09-netfilter/NEW_LINUS.md)

## What lives here

The packet-filtering framework behind iptables, nftables, NAT, connection
tracking and Kubernetes service routing (kube-proxy in iptables or nftables
mode). The IP stack calls `NF_HOOK()` at five fixed points; anything
registered on that hook sees the packet and returns a verdict.

The five IPv4 hooks:

```text
          PRE_ROUTING ──> routing ──> LOCAL_IN ──> local socket
                              │
                              └──> FORWARD ──> POST_ROUTING ──> out
  local socket ──> LOCAL_OUT ──> routing ──> POST_ROUTING ──> out
```

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `include/linux/netfilter.h` | `struct nf_hook_ops` | What you register: function, protocol family, hook number, priority. |
| `include/uapi/linux/netfilter.h` | `NF_DROP`, `NF_ACCEPT`, `NF_STOLEN`, ... | The verdicts a hook can return. |
| `core.c` | `nf_register_net_hook()` | Hooks are kept per network namespace, sorted by priority. |
| `core.c` | `nf_hook_slow()` | Runs every hook on the list until one doesn't say ACCEPT. |
| `nf_conntrack_core.c` | `nf_conntrack_in()` | Connection tracking: every packet is matched to a flow. Required for NAT and for "ESTABLISHED,RELATED" rules. |
| `nf_tables_api.c` | top of file | nftables: rules are compiled into a small bytecode run by `nf_tables_core.c`. Skim. |

## Watch it in the VM

Needs `CONFIG_NETFILTER=y` and `CONFIG_NF_CONNTRACK=y` (week 1 learning config).

```sh
lsmod | grep nf_                            # nothing if built in; see /proc/config.gz
zcat /proc/config.gz | grep -E 'CONFIG_NETFILTER=|CONFIG_NF_CONNTRACK='
dmesg | grep -i icmp_counter                # after loading lab 09
```

## Words to own

- **netfilter hook**: one of five points in the IP path where callbacks run.
- **verdict**: hook result: accept, drop, stolen, queue, repeat.
- **conntrack**: state table of flows (NEW, ESTABLISHED, RELATED).
- **NAT**: rewriting addresses; needs conntrack to rewrite replies too.
- **nftables**: the current rule engine; iptables-nft translates to it.
- **hook priority**: order in which callbacks on the same hook run.

## Answer in your notes

1. Your lab 09 hook drops a packet with `NF_DROP`. Who frees the skb?
2. Why must NAT run after conntrack?
3. Which hook does Kubernetes use to redirect a Service IP to a pod, and why
   that one?
