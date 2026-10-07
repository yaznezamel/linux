# Contributing upstream

The kernel does not take GitHub pull requests. Patches go by email to the
maintainers and mailing list for the code you change, and are reviewed in
public on <https://lore.kernel.org>.

Required reading, in this order:

1. `Documentation/process/submitting-patches.rst`
2. `Documentation/process/coding-style.rst`
3. `Documentation/process/email-clients.rst`
4. `Documentation/process/maintainer-netdev.rst` (before any networking patch)

## Branch hygiene for this fork

This fork carries a `docs/` folder that must never reach upstream.

```sh
git fetch upstream
git switch -c fix/short-name upstream/master    # or net/main, net-next/main
# ... edit, build, test ...
git commit -s                                   # -s adds Signed-off-by
git format-patch -1 --base=auto -o outgoing/
```

## Before sending

```sh
scripts/checkpatch.pl --strict outgoing/*.patch
scripts/get_maintainer.pl outgoing/*.patch      # who to send it to
```

- [ ] It builds with your change and with `W=1` for the file you touched.
- [ ] You booted it and tested the behavior you changed.
- [ ] The commit message says **why**, not only what, and has a subsystem
      prefix matching `git log --oneline -- <file>` (for example `net: dummy:`).
- [ ] `Signed-off-by:` with your real name.
- [ ] A bug fix has a `Fixes: <12-char sha> ("subject")` tag.
- [ ] Send a test copy to yourself first: `git send-email --to=you@... outgoing/*`.

## Where to make a first contribution

Ordered from easiest to most valuable:

| Target | Why it's a good start | Watch out for |
| --- | --- | --- |
| **`Documentation/`** fixes | Low risk; you'll find errors while reading | Fix real mistakes, not wording preferences |
| **`drivers/staging/`** | Exists so new contributors can clean up drivers; checkpatch fixes are welcome here. Two are network drivers: `rtl8723bs` (Wi-Fi) and `octeon` (Ethernet), see their `TODO` files | One logical change per patch; build the driver (`make M=drivers/staging/rtl8723bs`) |
| **`tools/testing/selftests/net/`** | Tests are always welcome and teach you the subsystem | Run them before and after your change |
| **Bugs found by syzbot** | Real fixes with reproducers: <https://syzkaller.appspot.com/upstream> | Harder; pick one in code you've studied |
| **netdev core / drivers** | The end goal | See the rules below |

## Netdev rules that catch newcomers

From `Documentation/process/maintainer-netdev.rst`:

- Name the tree in the subject: `[PATCH net]` for fixes, `[PATCH net-next]`
  for everything else.
- `net-next` is **closed during the merge window** (two weeks after each
  release). Check before sending new features.
- **Pure clean-up patches are discouraged** in networking: checkpatch fixes,
  variable reordering, or `devm_` conversions on their own are likely to be
  rejected. Do style clean-ups in staging instead.
- Local variables are ordered longest line to shortest ("reverse xmas tree").
- Don't repost a revised series within 24 hours.
- Fixes need a `Fixes:` tag.

## After sending

- Reply inline (bottom-posting, plain text) to every review comment.
- For a new version, use `git format-patch -v2` and add a changelog below the
  `---` line saying what changed since v1.
- If there's no reply after two weeks, send a polite ping as a reply to your
  patch.
- When it's merged, record the commit and the lore.kernel.org link in
  [brag-sheet.md](brag-sheet.md) and in Notion.
