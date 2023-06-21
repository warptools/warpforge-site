---
title: "Formulas and RunRecords"
layout: base.njk
eleventyNavigation:
    key: "L1: Formulas and RunRecords"
    parent: api
    order: 30
---


Layer 1 Warpforge APIs: Formulas and RunRecords
===============================================

Formulas are the descriptions of computations that we want to happen.

They talk about *inputs*, *actions*, and *outputs*.

Formulas are the key "[Layer 1](./layers.md)" concept.

## Schema

(And remember, any of the types already defined in [Wares, Inputs and Outputs](./wares-inputs-outputs.md) will continue to be referred to here — this layer is building upon that one.)

## Formulas

The formula type itself is short and sweet:

```bash
# Formula describes a single computation.
# What exactly the computation is is defined by the action
# (of which there are several kinds, but typically, it's commands
# which will be evaluated in some kind of hermetic container),
# and the environment the action wil run it is described by the inputs map.
# What data we collect after the action is completed is defined by the outputs.
type Formula struct {
	inputs {SandboxPort:FormulaInput}
	action Action
	outputs {OutputName:GatherDirective}
}
```

All the fun stuff comes inside the details of the inputs and outputs, and the action.

## Inputs and Outputs

Broadly speaking, identifying where an input should land and where an output will be sourced from is done with straightforward-looking strings.  That said, there's a few types that help tease apart and label the semantics of those strings, too:

```bash
# SandboxPort defines someplace within the sandbox we'll run the action in
# where data can be either put in or pulled out.
type SandboxPort union {
	| SandboxPath "/"
	| VariableName "$" # REVIEW: maybe call this SandboxVar for some consistency?
} representation stringprefix

# SandboxPath is one of the members of the SandboxPort sum type.
# It's a unix-like path, e.g. something like "foo/bar/baz".
#
# (Despite not beginning with a slash, it's interpreted as an absolute path,
# because the leading slash was stripped by SandboxPort union parse step.)
type SandboxPath string

# VariableName is used to describe a variable (as contrasted with a path).
# When the action is a one of the unixy-container actions,
# these will correspond to environment variables.
#
# Note that though VariableName is syntactically accepted in output declarations,
# it's not guaranteed to be supported by all actions.
#
# Mapping a VariableName to any kind of input other than a literal is strange
# and currently undefined.
type VariableName string
```

Input data can be declared in a couple of forms:

```bash
type FormulaInput union {
	FormulaInputSimple string
	FormulaInputComplex map
} representation kinded

# FIXME revisit name, this happens to also be the RunRecord.results value type!
#  ... except for the mount part, which would make no sense there.  Meh.
type FormulaInputSimple union {
	| WareID  "ware:"     # this is most of the time!
	| Mount   "mount:"    # not hermetic!  we'll warn about the use of these.
	| Literal "literal:"  # a fun escape valve, isn't it.
} representation stringprefix

type FormulaInputComplex struct {
	basis FormulaInputSimple
	filters FilterMap
}
```

TODO: is some weird composition of "literal:" and a FilterMap going to be sufficient to describe how to make an empty directory?  If not, we may need to add another thing to the `FormulaInputSimple` union for that.

TODO: I'm not thrilled with the way "literal" is working out there.  Goal: I feel like API users will be happier with us if we can make the literal field freestanding from the POV of a primitive JSON parser.

TODO: can we take "literal:" out of the string prefix and move it into the `FormulaInputComplex` somehow?  

TODO: (consider together with the above): mind that filters on a literal actually do make sense... iff it's going to go into a file.

TODO: trying to move literal to a field in `FormulaInputComplex` also causes us to need to say the other input field *shouldn't* be set then, which is... needing another union.

Something is funky here and I don't see the solution yet.

Outputs at the moment can be WareID or Literal.  Maybe this is really fine.

REVIEW: are we going to be inconveniencing ourselves if FormulaInputComplex isn't stringable?  See [notes about rich inputs and caching](https://warpforge.notion.site/decide-how-rich-to-make-inputs-359c79743ccc4c3e80d55da25eb693b1) .  (Probably not.  If a subsystem needs that, both entries in the struct are already stringable — this does not seem like a hard problem to kludge.)

Output declarations reuse the same `SandboxPort` concept that was used in defining Inputs, but of course are carrying data out of that, rather than in, so we need a structure that says several other things in addition to just where to pick up the data: for example, what packtype to use, and any filters that should happen along the way out:

```bash
# OutputName is a plain freetext string which a Formula (or Plot) author uses
# to identify the output data they want to collect.
# It's used when writing the Formula's outputs description,
# and will be seen again returned in the RunRecord
# which is produced by a tool when it evaluates the Formula.
type OutputName string

type GatherDirective struct {
	from SandboxPort
	packtype optional Packtype # should be absent iff SandboxPort is a VariableName.
	filters optional FilterMap # must be absent if SandboxPort is a VariableName.
}
```

TODO: review an alternate draft of gathering, which used more stringprefix unions in the common cases and thus felt more similar to the inputs: e.g. `"pack:tar:/thepath"`.

TODO no idea what a good word would be for reading vars out.   (and this still has some matching problems... but these are essentially the same with the above draft, and also are felt by inputs, so i guess that's just something we have now.)

Note that there are many possible combinations of ports and input and output types that are syntactically valid per the schema, but logically nonsense.  For example:

- mapping a `WareID` onto a `VariableName` input makes no sense.
- filters on data gathered from a `VariableName` make little sense.
- a `packtype` isn't actually optional if `from` is a `SandboxPath`.
- later on, at L2, if you `pipe` a `VariableName` into another `VariableName`, it will work out fine, but many other combinations would be nonsensical.
- etc.

Warpforge tools will check for obviously silly things like this before trying to evaluate a formula, and the `warpforge formula check` command can also be used to check for invalid constructions like this, without executing the formula.

## Actions

These are... fairly self explanatory, hopefully:

```bash
type Action union {
	| Action_Echo "echo"
	| Action_Exec "exec"
	| Action_Script "script"
} representation keyed

# Action_Echo is an action which will cause a formula to execute by
# just echoing its own formula.
# It's not useful on its own, except for as a debugging and demo tool.
type Action_Echo struct {
	# Not much to say in this one!
}

# Action_Exec describes launching a container, and running a single process in it.
# (Consider using Action_Script; it's more user-friendly.)
type Action_Exec struct {
	command [String] # fairly literally, what will be handed to exec syscall.
	cwd optional String
	network Bool (implicit: false)
	userinfo optional ActionUserinfo
	# n.b. no 'env' map: we moved that to inputs.
}

# Action_Script describes launching a container, launching a shell processes
# within the container, and then feeding your commands to that shell process.
# This is somewhat more complicated than Action_Exec, but also offers more
# opportunities for debugging, and lets you easily run several commands
# while in the same container.
type Action_Script struct {
	commands [String] # very different than exec's string list, though!  is parsed.
	shell optional [String] # specifies what's going to parse your commands.
	# future: consider an optional enum here for what features to expect from shell.
	cwd optional String
	network Bool (implicit: false)
	userinfo optional ActionUserinfo
	# n.b. no 'env' map: we moved that to inputs.
}

# Action_Noop is an action which does... nothing!
# It's sometimes useful if you have data munging work to do that's so simple
# that you can do it using the FilterMap on a FormulaInputComplex.
# (This is fairly rare to see in practice.)
type Action_Noop struct {
	# Not much to say in this one!
}
```

Some types provide more info to certain kinds of actions:

```bash
# ActionUserinfo can describe optional configuration for unix-like environments.
# Actions that launch containers will optionally contain this information.
type ActionUserinfo struct {
	uid Int (implicit: 0)
	gid Int (implicit: 0)
	username String (implicit: "luser")
	homedir String (implicit: "/home/luser")
}
```

FUTURE: we may want to add option to associate a whole OCI config hunk with these Actions that are wrappers for containers.  This would be treated as even more unhermetic/dangerous as mounts, but probably ought to be supported.  (Audience for this would mostly be people who want to do deployments of things.  Low priority.)

## FormulaContext

```bash
# FormulaAndContext is what we actually use as the document root
# when parsing a formula file.
type FormulaAndContext struct {
	formula Formula
	context optional FormulaContext
}

type FormulaContext struct {
	warehouses {WareID:WarehouseAddr}
}
```

## RunRecords

RunRecords are what come out of evaluating a Formula.

The main field of interest is the `results` field, which is simply a map that contains... whatever you asked for in the `Formula.outputs`, with the computed values the same keys as you used when writing the `GatherDirective`s.

```bash
type RunRecord struct {
    guid String      # purely to force uniqueness.
    time Int         # again, to force uniqueness.
    formulaID String # hash of the Formula that triggered this.
    exitcode Int     # what is says on the tin.  zero is success, per unix.
    results {OutputName:FormulaInputSimple} # map corresponding to output gathers.
}
```

## Examples

### A simple FormulaAndContext

```json
{
	"formula": {
		"inputs": {
			"/mount/path": "ware:tar:qwerasdf",
			"$ENV_VAR": "literal:hello"
		},
		"action": {
			"exec": {
				"command": ["/bin/bash", "-c", "echo hey there"]
			}
		},
		"outputs": {
			"theoutputlabel": {
				"from": "/collect/here",
				"packtype": "tar"
			},
			"another": {
				"from": "$VAR",
			}
		}
	},
	"context": {
		"warehouses": {
			"tar:qwerasdf": "ca+file:///somewhere/"
		}
	}
}
```

### A RunRecord

```bash
{
	"guid": "asefjghr-34jg5nhj-12jfb5jk",
	"time": 1245869935,
	"formulaID": "Qfm2kJElwkJElfkej5gH",
	"exitcode": 0,
	"results": {
		"theoutputlabel": "ware:tar:qwerasdferguih",
		"another": "literal:some content hello yes"
	}
}
```

## Sandbox Purity Caveats

Use of some sandbox-breaking features will imply use of other sandbox-breaking features.

For example, allowing a container to use the network will also cause it to mount several files from your host (in read-only) mode: specifically, your `/etc/resolve.conf` and your certificate store (whatever path that is).
