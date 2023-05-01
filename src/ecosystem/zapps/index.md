---
title: "Ecosystem: Zapps"
layout: base.njk
eleventyNavigation:
    key: "Zapps"
    parent: "Ecosystem"
    order: 60
---

Zapps
=====

Zapps is short for "Zero Dependency Applications",
and is a style of executable and library linking that is heavily used and encouraged in the ecosystem around Warpforge.

The main features of Zapps are that they run on any kind of linux environment,
without requiring any libraries or dependency management,
and without requiring any installation steps at all beyond existing on the filesystem.
This makes Zapps very easy to use, and we favor them in the Warpforge ecosystem for this reason.

The Zapp project has its own website: https://zapps.app/



Relationship of Zapps to Warpforge
----------------------------------

Zapps were pioneered within Warpforge, because Warpforge was a great environment in which to test such a thing,
and fully ensure its system independence and portability...
as well as because Zapps are great to _use_ within Warpforge, since they require no post-install hooks,
meaning you just mount them and you're good to go.

Depsite that origin, and that mechanical sympathy, Zapps are a totally independent concept.
You can use Zapps without `warpforge`, and you can use `warpforge` without Zapps,
and you can build and test a Zapp without `warpforge` (although it's hard!), and so forth.

The [Warpsys Catalog](/ecosystem/warpsys/) is predominantly full of software that's been packaged in the Zapp style.

The [Linkwarp](/ecosystem/linkwarp/) tool, also a part of the Warpforge ecosystem,
makes it very easy to gather large numbers of Zapps together in one `$PATH`, for ease of use.



---



Older Documentation
-------------------

Some older documentation that evolved into Zapps can still be found in the Warpforge Notion
(although we're moving away from using Notion going forward).
Here are the most relevant pages:

- [Wrangling Dynamic Library Linking](https://warpforge.notion.site/Wrangling-Dynamic-Library-Linking-68d36a19f1614785b0d9ebcda6623889)
- [Goal: Path-agnosticism](https://warpforge.notion.site/Goal-Path-agnosticism-1afbca83896d4ef3bff36c9b1344ee89)
- [Goal: Co-installability](https://warpforge.notion.site/Goal-Co-installability-b13a81f48bd94c56a09153770af6d28b)
- [Convention: Package File Layout](https://warpforge.notion.site/Convention-Package-File-Layout-36551029a2aa47dfb47f187fb89d73ce)
- [Wrangling Shebangs](https://warpforge.notion.site/Wrangling-Shebangs-7629959c30464e4a8b6a8294fe0d95f8) remains a somewhat unsolved problem.
