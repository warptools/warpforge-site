---
title: "Ecosystem: Warpsys"
layout: base.njk
eleventyNavigation:
    key: "Warpsys"
    parent: "Ecosystem"
    order: 10
---

Warpsys
=======

Warpsys is the name of a [catalog](/glossary.md#catalog) full of software
that's built in, and ready to use in, Warpforge.

The Warpsys catalog and its contents are considered part of the _ecosystem_ of Warpforge,
and not an integral part of it, because
you can use the software in the Warpsys catalog without using Warpforge,
and you can use Warpforge without using any references to the Warpsys catalog.
Many of the same people are involved in contributing to and maintaining both projects, though.



Seeing Warpsys
--------------

The Warpsys catalog is visible in a couple different ways:

- the rendered HTML catalog website: **https://catalog.warpsys.org/**
	- This is the easiest to navigate -- click through module names to find release info, download links for content, and links to metadata.
	- In addition to content built with Warpforge, you can find *how it was built* by looking at the [replays](/glossary.md#replay).
	  For example, this is a replay showing how the warpsys Bash package is built: [catalog.warpsys.org/warpsys.org/bash/_replays/zM5K3Vgei4...Zv5TQ](http://catalog.warpsys.org/warpsys.org/bash/_replays/zM5K3Vgei44et6RzTA785sEZGwuFV75vCazjhR11RH5veFdMTx7F5cg2c4NA5HXPK8Zv5TQ.html)
- the same Catalog data, but raw, in JSON files, in Git:
	- [**github.com/warptools/catalog**](https://github.com/warptools/warpsys-catalog)
- and the source code that produced these build instructions originally:
	- [**github.com/warptools/warpsys**](https://github.com/warptools/warpsys)



Contributing to Warpsys
-----------------------

We're happy to have more people working on contributing packages to the Warpsys collection!

Right now the way to do that is simply joining the [community](/community.md) chat,
and making pull requests to [warpsys source repo](https://github.com/warptools/warpsys).



Conventions within Warpsys
--------------------------

### Warpsys module, release, and item naming

The Warpsys catalog attempts to be a good example of the [recommended Catalog naming conventions](/catalogs/conventional-naming.md).

### Warpsys uses Zapps

The Warpsys catalog packages most software as [Zapps](/ecosystem/zapps).

Some software is also packaged as statically linked, where that seemed appropriate.

The item label or metadata in the catalog should indicate which packages are which.


