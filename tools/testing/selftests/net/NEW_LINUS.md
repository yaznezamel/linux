# The New Linus: `tools/testing/selftests/net/`

Week 10 (Dec 14-20): eBPF, tests, first patch.

## What lives here

The networking selftests: shell scripts and small C programs that build
network namespaces, veth pairs and routes, send traffic, and check the result.
The netdev maintainers run them on every patch series. Reading them is the
fastest way to see how a feature is meant to be used, and adding or fixing a
test is a welcome first contribution.

## Read, in this order

| File | What to get out of it |
| --- | --- |
| `Documentation/dev-tools/kselftest.rst` | How selftests are built, run and reported (TAP output). |
| `lib.sh` | Shared helpers: namespace setup, `check_err`, `log_test`. Most tests source it. |
| `Makefile` | `TEST_PROGS` lists what runs. |
| `rtnetlink.sh` | Many small tests of `ip link`/`ip addr` operations; easy to follow. |
| `fib_tests.sh` | Routing table behaviour; pairs with week 9's `fib_trie.c`. |
| one test touching code you studied | Search: `git grep -l veth tools/testing/selftests/net/*.sh` |

## Run it

```sh
make -C tools/testing/selftests TARGETS=net    # build the C helpers
vng --user root                                 # boot as root, then inside:
cd tools/testing/selftests/net
./rtnetlink.sh                                   # TAP-style PASS/FAIL/SKIP
```

Many tests skip when a config option or a user-space tool is missing. A
skipped test tells you which `CONFIG_` it needs: that is useful reading too.

## Words to own

- **kselftest**: the kernel's in-tree test framework.
- **TAP**: Test Anything Protocol, the output format.
- **netns-based test**: each test builds its own throwaway network in namespaces.
- **regression test**: a test added with a fix so the bug can't come back.

## Answer in your notes

1. Pick one test in `rtnetlink.sh`. What does it set up, what does it check?
2. Run it, then list every test that skipped and why.
3. Is there a test that covers `dummy` or `veth`? What would you add?
