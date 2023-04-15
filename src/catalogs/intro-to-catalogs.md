---
title: "Intro to Catalogs"
layout: base.njk
eleventyNavigation:
    key: "Intro to Catalogs"
    parent: "Catalogs"
    order: 10
---

Intro to Catalogs
=================

[[TOC]]

---



What are Catalogs?
------------------

Catalogs are a data structure which attaches human-readable names and references to the content-addressable snapshots of data handled by Warpforge.

Roughly speaking, Catalogs provide to Warpforge what tags and branches provide to Git.

More concretely, Catalogs contain:

1. **Mappings of human-readable names to WareIDs.**
2. Mappings of either the human-readable names or the WareIDs to Warehouse Addresses — URLs of places on the network that should be able to provide the content matching those WareIDs.
3. Metadata attached to the human-readable names — open ended, general purpose.
    1. May content simple informational content, like an "author" name.
    2. May contain semi-structured metadata that helps other tooling produce "automatic updates".
    3. May contain special metadata that references rebuild instructions (e.g. the Warpforge Plot document which should completely reproduce things) — this is called a "replay".
        - (Fun fact: Because Plot documents also include the catalog reference names for all the Plot's inputs... once can then look up those catalog entries, and find their replay plots, and... so on: this is recursively walkable, and can produce total explanations for where some any data or applications came from!)

Catalogs are stored within [workspaces](/warpforge/workspaces/).
All name resolution that Warpforge performs uses the catalogs in the nearest enclosing workspace as the data source for the name resolution.



Schema
------

See the [Catalog API Docs](./catalog-schema.md).



Filesystem
----------

See [Catalogs on the Filesystem](./catalogs-on-the-filesystem.md).



Real world example!
-------------------

One large real-world catalog we use heavily is the Warpsys catalog.

You can see this in two forms:

- As raw API objects, in JSON files, in git: see [https://github.com/warpsys/catalog](https://github.com/warpsys/catalog)!
- As navigable, crosslinked, pretty HTML: check out [https://catalog.warpsys.org/](https://catalog.warpsys.org/), which is a rendered version of that data!



Tools for Working with Catalogs
-------------------------------

Many CLI commands for working with catalogs can be found in subcommands of `warpforge catalog`.
Creating releases and updating catalogs and syncing catalogs can all be found in subcommands.

The `warpforge catalog bundle` command is an important one:
this maintains a workspace's catalog, ensuring that it vendors a copy of all data that's used by modules in that workspace.

Another interesting tool is `warpforge catalog generate-html`,
which produces a static website showcasing all of a Catalog's content in a navigable way.
(The website at [https://catalog.warpsys.org/](https://catalog.warpsys.org/) is the product of this tool!)



Local-First Operation
---------------------

Catalogs are meant to be local-first data structures.
Name lookups use already-local catalogs;
publishing releases also goes first to local catalogs, and to become globally visible, needs to later be pushed out.

Catalogs are greatly comparable to Git.
As with git clone before checking, catalogs should be fetched before use.
As with git commiting and subsequent pushing (or other forms of publishing),
releasing into a catalog also happens locally first, and then needs to be published more globally with further actions.

Local-first operation means operations are fast (because no network latency or remote services are involved),
reliable (you don't need internet in order to get work done),
and also gives a lot of wiggle room: you can review and test any changes to the local catalog before publishing further into the world.
