---
title: "Wares, Inputs and Outputs"
layout: base.njk
eleventyNavigation: 
    key: Wares, Inputs and Outputs
    parent: Data, Execution and Formulas
    order: 20
---

## Schema

### Wares

```bash
# WareID is a simple tuple of what kind of packing is used, and a hash.
# WareIDs are content-addressable.
#
# We use opaque strings for the packtype and the hash for simplicity.
type WareID struct {
	packtype Packtype   # typically "tar" or "get" or etc.
	hash string   # usually an actual hash, but handed to the io plugin verbatim.
} representation stringjoin {
	joiner ":"
}

# Packtype is a string that should identify a format for fileset packing.
# Typical examples are "tar", "git", etc.
# An enum could be used here; however, we use an opaque string here
# rather than enum because fileset packing is regarded as a plugin-style system.
type Packtype string
```

### Filters

Filters are used more in higher levels, such as when describing Inputs and Outputs.

Filters are also used in CLI commands and APIs for basic direct I/O (e.g. in the `warpforge ware` command family).

```bash
# TODO placeholder type, we may want something more structured here.
# TODO if it does get more structured, some if its uses might need union wrapper.
# TODO or this might be a good place to use multi-phase pattern recognition.
type FilterMap {String:String} representation stringpairs {
	entrySep ","
	pairSep "="
}
```

### Mounts

Mounts are (again, sorry) something used at higher levels.

```bash
type Mount struct {
	mode MountMode
	hostPath String
}

type MountMode enum {
	| Readonly "ro"
	| Readwrite "rw"
	| Overlay "overlay" # review: naming.
}
```

### Literals

Occasionally, we refer to "literal" values.  Those are... pretty simple.

```bash
type Literal string
```

Literals will be used to describe the value of environment variables, for example.

### Inputs and Outputs

We'll talk more about how WareIDs and Mounts are used as the inputs and outputs of computations in the pages for [Formulas and RunRecords](/api-specs/data-execution-formulas/formulas-and-runrecords) and [Modules, Plots, and Replays](/api-specs/data-execution-formulas/modules-plots-and-replays).

### Warehousing

TODO all the types about warehouse addressing and mirror lists and etc.  (Or maybe this belongs in another page about ‣ ?)

```bash
type WarehouseAddr string
```

## FAQ

### What s a "Ware" vs a "Fileset"?

A "ware" is a packed "fileset".

The abstract concept of a folder and files (and possibly their permissions, etc) is a "fileset".

A tar or a zip containing that content is a "ware".

### What do "Wares" have to do with "packages"?

In most computer discourse, the term "package" usually refers to a vague concept of either "software you can install on a system" or "a library release" or something.  Sometimes it comes with even more connotations, like "packages have version" or "packages have dependencies".

*We don't use the word package much when describing warpforge*, by intention, because those connotations can be complex and become opinionated.

So: probably, a "ware" can contain some kind of "package" (because packages usually are a glorified bunch of files).  But, again, depends on what you mean by "package"... so it's not a very clear statement.