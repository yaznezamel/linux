# The New Linus: `drivers/staging/`

Weeks 10-11 (Dec 14-27): first upstream patch. Full workflow:
[`docs/contributing.md`](../../docs/contributing.md)

## What lives here

Drivers that work but are not yet good enough for the main tree. Each has a
`TODO` file listing what must be fixed before it can graduate. Clean-up patches
that netdev would reject (style, naming, dead code) are welcome here, which is
why staging is where most people send their first patch.

Maintainer: Greg Kroah-Hartman. List: `linux-staging@lists.linux.dev`. Tree:
`git://git.kernel.org/pub/scm/linux/kernel/git/gregkh/staging.git`, branch
`staging-next`. (From the `STAGING SUBSYSTEM` entry in `MAINTAINERS`.)

Two drivers here are networking drivers, so your week 8-9 knowledge applies:

| Driver | What it is | Start with |
| --- | --- | --- |
| `rtl8723bs/` | Realtek SDIO Wi-Fi | `TODO`, then `core/rtw_cmd.c` |
| `octeon/` | Cavium Octeon Ethernet | `TODO`, then `ethernet-tx.c` |

## Find a first patch

```sh
cat drivers/staging/rtl8723bs/TODO
scripts/checkpatch.pl -f --terse drivers/staging/rtl8723bs/core/*.c | cut -d: -f3- | sort | uniq -c | sort -rn | head
```

Good first patches, in order of preference:

1. Something from the driver's `TODO` that you understand end to end.
2. Removing code that is provably unused (`git grep` the symbol first).
3. One class of `checkpatch.pl` issue in one file, for example unnecessary
   parentheses or a `CamelCase` variable rename.

Before you start, check nobody has sent the same fix:
<https://lore.kernel.org/linux-staging/> (search the file name).

## The rules that get first patches rejected

- One logical change per patch. Ten files with the same fix can be one patch;
  two different fixes in one file are two patches.
- Build the driver after the change. For `rtl8723bs` (module only):

  ```sh
  scripts/config -e STAGING -e WLAN -e MMC -e CFG80211 -m RTL8723BS
  make olddefconfig && make -j"$(nproc)" M=drivers/staging/rtl8723bs
  ```

  `octeon` only builds on x86 with `CONFIG_COMPILE_TEST=y`; prefer
  `rtl8723bs` for a first patch.
- Subject prefix matches history: `git log --oneline -- drivers/staging/rtl8723bs | head`.
- Base the patch on `staging-next`, not on this fork's branch, and never
  include `NEW_LINUS.md` or `docs/`.
- Run `scripts/checkpatch.pl --strict` on the patch and
  `scripts/get_maintainer.pl` for recipients.
- Plain-text email via `git send-email`; reply inline to review.

## Words to own

- **staging tree**: holding area for drivers below mainline quality.
- **maintainer**: person who reviews and applies patches for a subsystem.
- **subsystem tree**: maintainer's git tree that feeds Linus during the merge window.
- **merge window**: two weeks after a release when new features are merged.
- **linux-next**: daily merge of all subsystem trees, for early conflict testing.
- **Signed-off-by / DCO**: your certification that you may submit the code.
- **v2, v3**: revised versions of a patch after review, with a changelog.

## Answer in your notes

1. Which items in `rtl8723bs/TODO` could you do in a week?
2. What is the most common `checkpatch.pl` complaint in the file you picked?
3. Write your commit message before writing the code. Does it explain *why*?
