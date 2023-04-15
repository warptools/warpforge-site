---
title: "Catalog Schema"
layout: base.njk
eleventyNavigation:
    key: "Catalog Schema"
    parent: Catalogs
    order: 30
---

Catalog Schema
==============

(Note: the actual load-bearing schema is on [github](https://github.com/warptools/warpforge/blob/master/wfapi/wfapi.ipldsch)
and may have drifted somewhat from what we have in our documentation. Conceptually, they haven't diverged.)

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
## When projected into a filesystem, this data appears
## in the `{moduleName}/_module.json` file.
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
## and therefore CatalogModuleCapsule provides enough
## versioning hints for this area too.
##
## When projected into a filesystem: this data appears
## in the `{moduleName}/_releases/{releaseName}.json` files.
type CatalogRelease struct {
	releaseName ReleaseName
	items {ItemLabel:WareID}
	metadata {String:String}
}
```

### "Metadata"

What's in the metadata maps?

It's a mixed bag — it's open ended, and meant for extensions.  The metadata maps are meant to freely contain data that we didn't anticipate nor standardize when developing Warpforge.

Mostly we expect metadata to be fairly advisory and fairly freetext (e.g. "author").

Sometimes metadata can be used to contain a load-bearing extension.

Metadata can sometimes be a CID that points to another document!  (And we'll use this ourselves, for some features that Warpforge *does* understand well.)

When we do have well-known extensions, they generally appear just by conventions of well-known names of keys for the metadata map.  See the "extensions" section coming up next.

### Extensions

We don't have a ton of well-know extensions at the moment, but here's at least one
(and a pretty important one -- it's how we store the data for rebuilding and for the "recursive explain" feature!)
which shows how it can look:

1. There's an additional schema, which is wedged onto and tries to pattern-match on the metadata map (whether the module-wide one or the per-release one);
2. and if it needs another whole document's worth of data, that document should be referred to by CID (and will go in another directory of known name, with filename that is the CID; see [Catalogs on the Filesystem](./catalogs-on-the-filesystem.md)).

```bash
# This type can be smashed onto the ReleaseData.metadata map,
# and if it matches, feature detection ensues.
type MetadataRegardingReplays struct {
	replay &Plot
}
```
