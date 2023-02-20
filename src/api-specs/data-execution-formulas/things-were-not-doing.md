---
title: "Things we're not doing"
layout: base.njk
eleventyNavigation: 
    key: Things we're not doing
    parent: Data, Execution and Formulas
    order: 60
---

## Things we're not doing

This page is for listing API choices we considered, and *rejected*.

## Excessive use of unions for inputs and outputs

Something we could do, and considered, is huge unions of stringprefix for the inputs and outputs:

```bash
type Ware union {
	| WareTar "tar:"
	| WareGit "git:"
} representation stringprefix

type WareTar struct {
	hash String
}

type WareGit struct {
	hash String
}
```

Most output types would the same structure, but we repeat it as types anyway just in case:

```bash
type Output union {
	| OutputTar "tar:"
	# more fileset pack types go here.
} representation stringprefix

type OutputTar struct {
	hash String
}
```

### Why not:

It's just absurdly verbose and type-spamming and doesn't really help anyone very much.

The example doesn't look too bad, but this list would get out of hand if actually implemented.

### What we're doing instead:

Just strings.  It's fine.

We *could* use an enum for packtype without issue, probably, but it doesn't seem super necessary or useful, either, given that there's kiiiinda a plugin sorta thing going on down there.

## Attempting to Use Types to describe Sandboxing Degree

We could try to use an abundance of schema types to immediately and clearly describe and verify whether features that mean lesser degrees of sandboxing are being used.

Or we could try, anyway.  Here's now that turned out, as a draft:

```bash
# FormulaSketch describes an intent to do some stuff.
# FormulaSketch is the vaguest form of Formula,
# and needs to be processed into a FormulaWired,
# and then finally resolved into a FormulaCrystalized,
# before it can be executed.
#
# Almost any kind of input description is allowed in a FormulaSketch,
# including ones that aren't going to be pure or replayable.
type FormulaSketch struct {
	input {InputKey:InputSketch}
	action Action
	ouptut {OutputName:Output}
}

# FormulaWired is the same as FormulaSketch,
# but ingests are no longer allowed (the ingestion must now have been computed),
# nor are 'candidate' references allowed (catalog selection must be done now).
# Catalog references are still allowed -- so, this is a replayable
# instruction, but only assuming a catalog is associated.
type FormulaWired struct {
	input {InputKey:InputWired}
	action Action
	ouptut {OutputName:Output}
}

# FormulaCrystalized is the same has FormulaWired, but even further pinned down.
# It's fully curried, no non-hashed references left -- ready to run.
# (Correspondingly, it's lost a lot of semantic info, such as catalog references,
# meaning this structure has no external references,
# and is nicely content-addressable itself with minimal noise sources,
# but also not very suited to be explainable
# (so we often keep the other forms too)).
type FormulaCrystalized struct {
	input {InputKey:InputCrystalized}
	action Action
	ouptut {OutputName:Output}
}

type InputSketch union {
	| WareID "ware:"
	| Pipe "pipe:"
	| CatalogRef "catalog:"
	| CandidateRef "candidate:" # Like catalog, but dangling a bit.
	| Ingest "ingest:" # Sideeffect rich.
	| Mount "mount:" # Extremely sideeffect rich.
} representation stringprefix

type InputWired union {
	| WareID "ware:"
	| Pipe "pipe:"
	| CatalogRef "catalog:"
} representation stringprefix

type InputCrystalized union {
	| WareID "ware:"
	# Review: okay, irritatingly, we do still want 'mount' here.  Sometimes.
} representation stringprefix

type Action union {
	| ActionExec "exec" # Container execution.  Simplest.
	| ActionScript "script" # Like exec, but easier to use, more debuggable.
	| ActionNoop "noop" # For the rare case where I/O is all you needed.
} representation keyed

type Output struct {
	packType string
	# filters are not defined in this draft, yet.
}
```

Doing work here with types instead of predicates is *kinda* nice.  

### Why not?

- The sheer proliferation of types is... a bit.
    - and I don't think we were even done yet; it probably keeps multiplying as we deal with the higher layers.
- The bit about mounts drills through it especially badly.
    - Maybe treating mount as a predicate one and keeping the rest is the most practical choice?
    - Wonder if we should just push mounts out into a different thing that's not considered an input at all.  They share the same effect space as input paths, but otherwise, they're... kinda not an input, logically.  Putting them in a separate space would fix the weird types-not-saving-us thing above.
        - Doesn't really win at types-describe-sandbox-level either, unless you write a whole alternate family of formula-like types without the mounts field... and thus then a whole family of plot types without it... etc.  That seems like an unreasonable amount of noise.
        - Also mildly unfortunate in that approach: it would remove the ability to let plots give them a name for easy multiple use.
- There are several other things that are also still necessary to verify with logic:
    - Pipes referring to sensible things.  Or even being used within a bigger plot at all, for that matter.
        - This isn't a count against types-describe-sandboxing, per se, but it does at least count for "there will be verifiers that run on this that are not at schema-time".
- We'd be forcing ourselves to use schema try-stacks kind of a lot.
    - It's viable, the ipld schema system is designed to make this possible!  But, eh, it's probably better left as a last resort and for version evolution, rather than in basic behavior of a single version of an api.

### Another alternative, also not great:

- We could let mounts be part of the union used in plots (so we can name them for ease of multiple use) but then make them a special bit of a formula action instead of an input there.
    - Not sure if this is really improving anything.
    - Kinda want to point at the fact that a mount is both an input and an output, but this is only doing at that one level, so, it only communicates that half the places it should?  Yeah, not helpful.

### What we're doing instead:

Different types for the inputs of a formula vs a plot still seem useful, because those are whole different subsystems.

(We could imagine someone building a swappable component that needs to speak formulas, but doesn't need to speak plots (and we've done that before, and probably will do it again), so it's not a bad idea to have some scope cutoff points there.)

Stuff like "mount" are just going in the unions, though.  If an API takes it, then we're putting it in the union; that's it.  No attempts to do clever logical checks with schemas.

## Gather directives being a whole family of types

Tried this:

```bash
type GatherDirective struct {
	from SandboxPort
	strategy GatherStrategy
}

# TODO please find a less enterprise name for this.
type GatherStrategy union {
	| Packtype string # in the simple case: just say what pack format to use!
	| GatherStrategyComplex map # if you need filters or something, this.
} representation kinded

type GatherStrategyComplex struct {
	packtype Packtype
	filters FilterMap
}
```

### Why not?

It looked weird in a bunch of ways.

TODO: if your port is a `VariableName`, do you... really need a `GatherStrategy`?  Nope — we actually need a zero value for that case, idk how to squeeze that in.  (We also might want something fancier here in the future too, e.g. if we try to do object passing that's gonna need an opt-in mechanism, so that really complicates things.)