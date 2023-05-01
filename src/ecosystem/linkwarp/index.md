---
title: "Linkwarp"
layout: base.njk
eleventyNavigation:
    key: "Linkwarp"
    parent: "Ecosystem"
    order: 40
---

Linkwarp
========

`linkwarp` is a tool for building symlink farms that make many executables available easily in one `$PATH` entry.

:::todo
Linkwarp needs considerably more documentation!
:::



Why not...?
-----------

### ... just use more $PATH entries?

Possible.  But two problems with this:

- it doesn't scale super well (it creates a linearly growing search problem)
- and it's order-dependent and requires string concatenations (which is a pain to manage).

Gathering a bunch of symlinks in one directory is simpler,
faster to serve lookups into,
and doesn't have any order-sensitivity which makes it much easier to compose.

### ... use union filesystems?

Most union filesystems don't scale the way you'd want them to for this to work.



---



Older Documentation
-------------------

Some older documentation that lead to Linkwarp can still be found in the Warpforge Notion
(although we're moving away from using Notion going forward).
Here are the most relevant pages:

- [Wrangling Executable Collections](https://warpforge.notion.site/Wrangling-Executable-Collections-e9e1844bcfc44d528eed09107d2ebadc)

