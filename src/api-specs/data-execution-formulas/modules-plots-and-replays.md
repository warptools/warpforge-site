---
title: "Modules, Plots, and Replays"
layout: base.njk
eleventyNavigation:
	parent: Data, Execution and Formulas
	order: 40
---

Modules, Plots, and Replays
===========================

tl;dr
-----

- A **module** is the unit of creation.
	- A module has a **module name**, which you'll also see in catalogs (a lineage name and a module name are the same thing!).
	- A module generally has a **plot**.
- A **plot** is a declarative description of a computation (or, a graph of computations) to do.
	- A plot is very comparable to a formula, but permits a lot more:
		- It can contain multiple steps, which each will be resolved into a formula, and pipe data between them.
		- A plot can make references to catalogs for input data.
		- A plot can use ingests to get input data from the local environment.
	- Plots also have a "replayable" form, which is mostly the same but only allows a subset of features.  These can be attached to catalogs as instructions for how to reproduce the content that the catalog refers to!

Schema
-----

### Module

The Module API object is fairly skeletal right now.  Mostly this is a reserved future space for expansion.

```bash
type Module struct {
	name ModuleName
	# semantically: `plot optional Plot`... but practically: it's in a sibling file.
	# future: other optional fields used by "override modules".
	# future: maybe something about recommended update patterns for any catalog inputs in the plot?
}

# ModuleEnvelope is a type used at the root of documents for version agility.
type ModuleEnvelope union {
	| Module "module.v1"
} representation keyed

# ModuleName strings tend to look a bit like URLs.
# For example: "foo.org/teamname/projectname".
#
# Characters like "/" and "." are allowed, but ":" and whitespace is forbidden
# and the use of other punctuation characters is Not Recommended.
type ModuleName string
```

(See also [workspace FAQ regarding override modules](https://www.notion.so/Workspaces-7d27dd769ba74440989b101cc14157c6).)

### Plot

Plots describe a graph of computations.  (You might've also called them pipelines, perhaps?  But they're not required to be very linear!)

Plots have inputs and outputs, conceptually similar to how formulas have those concepts, but they're a little different: plots just map names to things (instead of mapping them directly to sandbox port concepts like mount paths or variable names that will be seen inside the sandbox).

Plots have actions (sort of), but they're called "steps".  Each step will be templated into a Formula.  (Or, contains a sub-Plot — plots are recursive.)

```bash
type Plot struct { # has previously had many names: protoformula, module, etc.
	input {LocalLabel:PlotInput}
	steps {StepName:Step}
	outputs {LocalLabel:PlotOutput}
}

# PlotCapsule is a type used at the root of documents for version agility.
type PlotCapsule union {
	| Plot "plot.v1"
} representation keyed

# StepName is for assigning string names to Steps in a Plot.
# StepNames will be part of wiring things together using Pipes.
#
# Must not contain ':' charaters or unprintables or whitespace.
type StepName string

# LocalLabel is for referencing data within a Plot.
# Input data gets assigned a LocalLabel;
# Pipes pull info from a LocalLabel (possibly together with a StepName to scope it);
# when a Step is evaluated (e.g. turned into a Formula, executed, and produces results),
# the results will become identifiable by a LocalLabel (scoped by the StepName).
#
# (LocalLabel and OutputName are essentially the same thing: an OutputName
# gets casted to being considered a LocalLabel when a Formula's results are hoisted
# into the Plot.)
#
# Must not contain ':' charaters or unprintables or whitespace.
type LocalLabel string

type PlotInput union {
	PlotInputSimple string
	PlotInputComplex map
} representation kinded

# PlotInputSimple is extremely comparable to FormulaInputSimple --
# and it's a superset of it: all the same things are acceptable here.
# PlotInputSimple adds more features:
# some are for getting data from the wider universe (mediated by Catalogs);
# some are for getting data ingested from a host environemnt (unhermetic!);
# and some are simply for wiring all the steps in a Plot together
# into a computable graph!
type PlotInputSimple union {
	| WareID "ware:" # same as in FormulaInputSimple.
	| Mount "mount:" # same as in FormulaInputSimple.
	| Literal "literal:" # same as in FormulaInputSimple.
	| Ingest "ingest:" # allows demanding ingest of data from the environment!
	| CatalogRef "catalog:" # allows lookup of a WareID via the catalog!
	| CandidateRef "candidate:" # Like catalog, but dangling a bit.
	| Pipe "pipe:" # allows wiring outputs from one formula into inputs of another!
} representation stringprefix

# PlotInputComplex allows decorating a PlotInputSimple with filters.
type PlotInputComplex struct {
	basis PlotInputSimple
	filters FilterMap
}

type PlotOutput union {
	PlotOutputSimple string
	PlotOutputComplex map
} representation kinded

type PlotOutputSimple union {
	| Pipe "pipe:"
} representation stringprefix

type PlotOutputComplex struct {
	basis PlotOutputSimple
	filters FilterMap
}

type Pipe struct {
	stepName StepName # if set, should be a sibling; if empty, means it's a reference to the parent's input map.
	label LocalLabel
} representation stringjoin (":")

# We'll define the rest of those new members of InputSketch in the next codeblock.

type Step union {
	| Plot "subplot"
	| Protoformula "formula"
} representation keyed
```

### Protoformulas

One of the two kinds of Step that a Plot can contain are Protoformulas.

As the name suggests, a Protoformula is just information with some placeholders in it which can easily be stamped out into a real Formula.

```bash
type Protoformula struct {
	inputs {SandboxPort:PlotInput} # same as Formula -- but value is PlotInput.
	action Action # literally verbatim passed through to the Formula.
	outputs {LocalLabel:GatherDirective} # same as Formula -- but key is LocalLabel.
}
```

When a Protoformula is turned into a formula, the inputs have to be mapped from `PlotInput` to `FormulaInput`.  That means:

- Any ingests have to be ingested, and turned into WareIDs (by packing new data!).
- Any CatalogRef have to be resolved, and turned in to WareIDs (by lookup in a Catalog).
- Any Pipe have to be resolved, and turned into WareIDs (by lookup locally in the Plot).

After the Protoformula has been turned into a Formula, and then the Formula has been executed, the results are handled very simply:

- The `RunRecord.results` map is cast to have keys that are type `LocalLabel`... and that's it.

### New kinds of Input

Plots get to use all the same inputs that Formulas did (`WareID`, `Mount`, `Literal`), but also several more.

We already defined `Pipe` above because it was needed to define the graph relationships within Plot.

Here are the rest:

```bash
type Ingest TODO

type CatalogRef struct {
	moduleName ModuleName
	releaseName String
	itemName String
} representation stringjoin (":")

type CandidateRef TODO
```

On the Filesystem
-----------------

There are two files for describing modules and their plot:

- `module.wf`
- `plot.wf`

Both of them contain... what the filename says.

The reason for these being separated is because they serve different roles.
The plot document is about some work to be done.
The module document is about how that work and its results can be refered to by other modules.

Typically, every `module.wf` file has a `plot.wf` file next to it.
However, some features in the future roadmap may allow for more diversity of this relationship.

When using commands like `warpforge run ./...`, it searches for any appearances of `module.wf` files, recursively down from the current directory.
