---
title: "Catalogs"
layout: base.njk
eleventyNavigation: 
    key: Catalogs
    parent: API Specs
    order: 30
---

# Catalogs

# What are Catalogs?

Catalogs contain:

1. **Mappings of human-readable names to WareIDs.**
2. Mappings of either the human-readable names or the WareIDs to Warehouse Addresses — URLs of places on the network that should be able to provide the content matching those WareIDs.
3. Metadata attached to the human-readable names — open ended, general purpose.
    1. May content simple informational content, like an “author” name.
    2. May contain semi-structured metadata that helps other tooling produce “automatic updates”.
    3. May contain special metadata that references rebuild instructions (e.g. the Warpforge Plot document which should completely reproduce things) — this is called a “replay”.
        - (Fun fact: Because Plot documents also include the catalog reference names for all the Plot’s inputs… once can then look up those catalog entries, and find their replay plots, and... so on: this is recursively walkable, and can produce total explanations for where some any data or applications came from!)

Catalogs have a schema (like all parts of Warpforge APIs!), but *also* come with a standard definition for how to project them onto a filesystem, because this is often operationally useful.  See [Catalogs on the Filesystem](https://www.notion.so/Catalogs-on-the-Filesystem-8bf611efa0db4261aef3f66cbf29fcf5) for more details on the filesystem projection part; keep reading below for the schema and other general info about Catalogs.

## Schema

(Note: the actual load-bearing schema is in [https://github.com/warptools/warpforge/blob/master/wfapi/wfapi.ipldsch](https://github.com/warptools/warpforge/blob/master/wfapi/wfapi.ipldsch) and may have drifted somewhat from what we have in Notion.  Conceptually, they haven’t diverged.)

```bash
## Catalog is a large data structure that maps human readable names to WareIDs
## (as well as a variety of metadata).
##
## The catalog tree itself uses a large sharded merkle-tree structure,
## which allows scalable, paralellizable verification,
## as well as efficient verification of sub-trees.
##
## Within that large tree structure, a simple structure of
## "moduleName:releaseName:itemName" can be seen as pathing over the catalog,
## and is sufficient to lookup a Ware ID.
##
## Catalogs are defined foremost by this schema,
## and their IPLD hashed document structure.
## However, they also have a standardized projection to a filesystem,
## which is often operationally handy.
type Catalog {ModuleName:CatalogModule}
  representation advanced ProllyTree

# Remember, `ModuleName` has already been declared earlier.
# It's just a string -- something that looks vaguely like a URL.

type ReleaseName string # with some limits: roughly [a-zA-Z0-9-], just to keep simple.
type ItemLabel string   # with some limits: roughly [a-zA-Z0-9-], just to keep simple.
  # ^ This is the the same range as OutputName, but we define different types for them.

## CatalogRef is a tuple that allows lookup of a WareID in a Catalog.
##
## A typical value might look something like "foobar.org/frob:v1.2.3:linux-amd64-zapp".
## CatalogRef values are often seen in serialized documents with a "catalog:" prefix,
## in the same way that WareIDs are often seen with a "ware:" prefix;
## they're usually used with a wrapper type with that prefix for clarity purposes.
type CatalogRef struct {
	moduleName ModuleName
	releaseName ReleaseName
	itemLabel ItemLabel
} representation stringjoin {
	join ":"
}

## CatalogModuleCapsule is a small wrapper type used for versioning
## the CatalogModule type when serialized.
type CatalogModuleCapsule union {
	| CatalogModule "catalogmodule.v1"
} representation keyed

## CatalogModule is the first level of data in a Catalog,
## after having navigated the large sharded module name map.
##
## A CatalogModule contains a map of named releases,
## and also some free-form metadata.
##
## When projected into a filesystem: this data appears in the `{moduleName}/_module.json` file.
##
## Note that the releases map is order-preserving.
## Newer content is generally placed at the "top" of the map.
## Some tooling may choose to rely on this property.
##
## Note that the values of the releases map are a CID --
## these link to another, separate document.
## This is so subtrees can be trimmed down and only partially transmitted,
## and to ensure the serial size of CatalogModule itself does not grow
## unduly quickly as releases are appended to the structure.
type CatalogModule struct {
	name ModuleName
	releases {ReleaseName:&CatalogRelease}
	metadata {String:String}
}

## CatalogRelease is part of a Catalog's tree structure, 
## and is one of the discreet documents within a Catalog.
## The CIDs in the releases map in the CatalogModule type point to this.
##
## The releaseName value here reiterates the same name
## indicated by the CatalogModule pointing at this.
##
## The "items" map associates item labels
## (which are freetext, but by convention contain phrases like
## "linux-amd64-zapp" or "darwin-amd64-static" or "src", etc)
## with WareIDs (which are usually IDs of filesystem snapshots).
##
## Freeform metadata may appear in each release document.
## Some of this metadata is "well-known".
## For example, the key "replay", if present, may be expected
## to contain a CID to a document that's a Warpforge plot that can reproduce
## the contents of this release.
##
## Note the a lack of "capsule" type for versioning this structure;
## this because we assume that if this part of the protocol evolves,
## then it will do so in tandem with the CatalogModule type,
## and therefore CatalogModuleCapsule provides enough versioning hints for this area too.
##
## When projected into a filesystem: this data appears in the `{moduleName}/_releases/{releaseName}.json` files.
type CatalogRelease struct {
	releaseName ReleaseName
	items {ItemLabel:WareID}
	metadata {String:String}
}
```

### “Metadata”

What's in the metadata maps?

It's a mixed bag — it's open ended, and meant for extensions.  The metadata maps are meant to freely contain data that we didn’t anticipate nor standardize when developing Warpforge.

Mostly we expect metadata to be fairly advisory and fairly freetext (e.g. "author").

Sometimes metadata can be used to contain a load-bearing extension.

Metadata can sometimes be a CID that points to another document!  (And we’ll use this ourselves, for some features that Warpforge *does* understand well.)

When we do have well-known extensions, they generally appear just by conventions of well-known names of keys for the metadata map.  See the “extensions” section coming up next.

### Extensions

We don’t have a ton of well-know extensions at the moment, but here’s at least one (and a pretty important one — it’s how we store the data for rebuilding and for the “recursive explain” feature!) which shows how it can look:

1. There’s an additional schema, which is wedged onto and tries to pattern-match on the metadata map (whether the module-wide one or the per-release one);
2. and if it needs another whole document’s worth of data, that document should be referred to by CID (and will go in another directory of known name, with filename that is the CID; see [Catalogs on the Filesystem](https://www.notion.so/Catalogs-on-the-Filesystem-8bf611efa0db4261aef3f66cbf29fcf5)).

```bash
# This type can be smashed onto the ReleaseData.metadata map,
# and if it matches, feature detection ensues.
type MetadataRegardingReplays struct {
	replay &Plot
}
```

## Filesystem

See [Catalogs on the Filesystem](https://www.notion.so/Catalogs-on-the-Filesystem-8bf611efa0db4261aef3f66cbf29fcf5).

## Real world example!

See [https://github.com/warpsys/catalog](https://github.com/warpsys/catalog) for a catalog in the wild (as its filesystem form, in a git repo, which happens to be a dang handy way to handle it).  And check out [https://catalog.warpsys.org/](https://catalog.warpsys.org/) as a rendered version of that data!

# Tools for Working with Catalogs

Many CLI commands for working with catalogs can be found in subcommands of `warpforge catalog`.

One particularly interesting tool is `warpforge catalog generate-html`, which produces a static website showcasing all of a Catalog’s content in a navigable way.

You can see an example of such a website at [https://catalog.warpsys.org/](https://catalog.warpsys.org/) — this is where a series of packages for the Warpsys project are published!

The raw data for this website matches the [https://github.com/warpsys/catalog](https://github.com/warpsys/catalog) repo.

# Where do Catalogs live?

Every Workspace has a Catalog.

Root Workspaces have several catalogs, which are named.

The `warpforge catalog bundle` command maintains a workspace's catalog.

Other commands are usually used to manage the named catalogs in a root workspace (because these named catalogs are generally something being shared and coordinated with larger groups of people).

TODO extract summary from [decide how workspaces and catalogs interact](https://www.notion.so/decide-how-workspaces-and-catalogs-interact-beb3a48a26c7428ea623cbb079f08af3).

FUTURE: should changelogs be attachable?  As... wares?  (Probably, but also we can probably make this an extension thing using metadata, and specify it later.  Much later.)  (If we make any search tools, they should know how to probe this.)  (May imply that a catalog should have a config detail for "suggested warehouseaddr for misc like changelog blobs"?)

# Releases

## Publishing New Releases

Publishing new releases is a multi-phase process.  It starts local, and then you expand it in scope with subsequent actions.

### Local Release Flow

When building with warpforge, you can make a release record with what you just built.

- The output names in the module's plot become item labels in the release info.
- The module name is the lineage name.
- The release name is for you to specify at the time — either in a CLI arg, or you'll be prompted for it, or the special name "candidate" can be used as a default (but will only be allowed to move a limited distance through the rest of the release publication scopes; this is meant for local-only testing).

The catalog appended will be **the catalog in the immediate workspace**.  (Not the root workspace, or the home workspace, or anywhere farther out.)

This means it's... Well, basically useless.  For now.

The only thing you can do with that is refer to it from other modules in the same workspace.

(So, okay, if you have a large workspace with lots of modules, maybe that's already a big deal.  But to continue the discussion, let's suppose you've got a very simple workspace: one git repo, one warpforge workspace right at the root of it, and one warpforge module all by itself in that workspace.)

The next things you can do are:

- Publish globally, somehow!  (Git push and have a bot online somewhere pick it up?  Or use other push based tools?)
- Publish to the parent workspace, increasing the scope of visibility.
- Publish directly into one of the catalogs in your root workspace!

## What's in a Release?

### The content!

At minimum, a release has its identifiers (the module name, the release name, and the set of item labels within it) and the WareIDs that are the release's content.

Each WareID identifies a packed filesystem full of data.

Beyond that: There's also a couple of attachments that are considered pretty standard.  See the following sections.

### Mirrors

- "Mirrors" is a list of information about where to fetch Wares from.
    - This data can be indexed by WareID, or by ModuleName, etc.
    - Without this, to actually fetch that data if you didn’t already have it, you’d just have to... run around the whole internet, asking every machine “hey do you have $this WareID?”.  While you could do that — and it would even be “secure”, since WareIDs are cryptographic hashes — it would be beastly inefficient :)
- Where it lives:
    - In Catalogs on the filesystem, this appears in a `_mirrors.json` file next to the `_module.json` file.  (Both are in a directory that's the module name.)
- Where it hashes:
    - If mirror info is hashed, it's in a parallel tree to the release data itself.
        - Release data does not point to a hash of the mirror info.  It's not relevant enough.
        - The root hash of a catalog usually includes the root hash of the mirror tree.
            - This isn't standardized yet.  (Could; Haven't been forced to yet.)
            - e.g. if the catalog is being shipped around as a git repo: yeah, the git repo root hash happens to cover the mirror info.

### Replay plots

- A "replay" is a term describing a plot, with restrictions: all the inputs must be fixed.
    - A release can link to a replay, thus stating "here's how you should be able to reproduce this release's contents yourself".
- Where it lives:
    - In Catalogs on the filesystem, these appear in a content-addressed directory, `_replays/*` next to the `_module.json` file.  (They're heaped together, not projected as indexed under the release name that points to them.)
- Where it hashes:
    - Because the release metadata refers to the replay, the hash of the release covers the replay instructions.

# Example Filesystem

[Catalogs on the Filesystem](https://www.notion.so/Catalogs-on-the-Filesystem-8bf611efa0db4261aef3f66cbf29fcf5)