# Brag sheet

Facts and numbers for talking about Linux. Prefer numbers you measured yourself,
and say how you got them.

## Measured on this tree (v7.3-rc6, October 2026)

| Fact | Number | How |
| --- | --- | --- |
| Files tracked in git | 96,055 | `git ls-files \| wc -l` |
| C source and header files | 64,553 | `git ls-files '*.c' '*.h' \| wc -l` |
| Lines of C (`.c` + `.h`) | ~37.7 million | `git ls-files '*.c' '*.h' \| xargs cat \| wc -l` |
| Lines in the networking core (`net/`) | ~1.3 million | same, limited to `net` |
| Lines in `net/` + `drivers/net/` | ~6.6 million | same, both directories |
| Directories under `arch/` | 21 (20 CPU architectures + User-Mode Linux) | `ls -d arch/*/` |
| Distinct maintainers listed in `MAINTAINERS` | 2,058 | `grep '^M:' MAINTAINERS \| sort -u \| wc -l` |
| Rust source files | 496 | `git ls-files '*.rs' \| wc -l` |

## Measure next (needs full history: `git fetch upstream --tags --unshallow`)

```sh
git shortlog -sn v7.2..v7.3 | wc -l                  # developers in one release
git rev-list --count v7.2..v7.3                      # commits in one release
git rev-list --count --no-merges v7.2..v7.3 -- net/  # networking share
git log --since="1 year ago" --format='%ae' | sed 's/.*@//' | sort | uniq -c | sort -rn | head
                                                     # which companies' domains appear most
```

## Talking points

- **Started as a hobby.** Linus Torvalds announced it on Usenet in August 1991
  as "just a hobby, won't be big and professional".
- **Git exists because of Linux.** Torvalds wrote git in 2005 to manage kernel
  development.
- **It runs the world's computing.** Every machine on the TOP500 supercomputer
  list has run Linux since November 2017, Android is built on the Linux
  kernel, and most cloud servers run it.
- **Predictable releases.** A new major release every two to three months
  (`Documentation/process/2.Process.rst`): a two-week merge window, then
  weekly `-rc` releases until it's stable.
- **Review is public.** Every patch and every review comment is archived on
  <https://lore.kernel.org>; anyone can read why a line of code exists.
- **"We don't break user space."** Kernel changes must not break existing
  programs; that rule is why decades-old binaries still run.
- **A second language.** Rust has been supported alongside C since 6.1.

## My own record

| Date | What | Link |
| --- | --- | --- |
| | First kernel built and booted | |
| | First module loaded | |
| | First patch sent | |
| | First patch merged | |
