---
title: "Larks"
layout: base.njk
eleventyNavigation: 
    key: Larks
    parent: Data, Execution and Formulas
    order: 50
---


(This part is *extremely speculative*.  It's beginning to use Starlark as an example of what we might want to see in an "L3+" system.  The goal in this doc isn't necessarily working stuff or committing to details, it's just to dream about what good composition would look like.)

((UPDATE: Work started on https://github.com/ipld/go-datalark — this should provide a very useful binding library that, combined with our existing schemas, should make massive amounts of things “just work”.)

### Considerations

- We probably ought to be able to see how a programmatic ‣ library could be formed here.
- Remember to try to solve for [understand user story for generating inputs based on proglang](https://www.notion.so/understand-user-story-for-generating-inputs-based-on-proglang-dcde115cc8a141d3a255ddc3aa345d61) with this — ingesting someone else's things probably needs a tool that's "templated" somehow, which means we should be able to see it up here, probably.

## Larksnips

(A working title for "a snippet of starlark which patches a warpforge plot document".)

### Traits of a good Larksnip

A good larksnip should be applicable to a plot as a patch, and it should be commutative with other well-behaved larksnips.

In essense, that is always reached if it's a patch that A) doesn't look at what it's about to patch (!) and thereby doesn't change behavior based on what's come before and B) doesn't collide with anything.

(That Part B criteria would be really tricky... if we didn't have content-addressing providing a natural conflict avoidance mechanism.  But we do have it!  Sweet.)

### What about things that don't fit that model?

How should one deal with noncommutable things?

For example, `$PATH`.  Appending path — like any string append — is not commutative.  It's order sensitive.

Okay, then there are two options:

- Come up with some logical manipulation of the data that's commutable.  For example, sorting the segments as strings, then re-concatenating them.
    - (... but that's also a pretty great example of when this angle of approach can fail to be a good idea.  That's not a semantically great idea for `$PATH`!)
- *Don't try to solve the problem in larks at all*.  Maybe the ecosystem should have a tool for solving this problem.
    - This is why we build the symlink farming tools that assemble a `/bin` folder by looking at `/app/*/bin/*`.  This solves the problem nicely and is not order-sensitive.
        - (This remains a TODO at present!  But seems likely to be one of the best options going forward, and very easy to do.)
- Or... roll with it.
    - We do sorta inevitably append the `action.script`, and assume that the symlink farmer is gonna come first.
    - Sometimes maybe you do want a larksnip that's not commutative, and actually looks at and wangles some of the existing plot content.  If so... just try to document it well.

## Using Libraries

Larksnips wouldn't be very useful if you couldn't refer to libraries of them that get common work done.

### The "standard library"

There's almost nothing in the "standard library".

Just the `Plot` and `Formula` and suchlike built-in types.

And the `PlotPatch` function, which takes a bunch of functions that 

### Using other libraries

🚧 Very unclear how this should work.

Almost certainly, the libraries are allowed to exist in parent workspaces.

(There wouldn't be much practical value in boilerplate reduction provided if this wasn't allowed!)

(If working in a team, you'll want to track the libraries in the parent workspace that the team shares for all the teams projects.)

## Inputs

🚧 Needs definition, badly.

The catalog is probably an input.

Are parent catalogs searchable?  Yeah, a handle to that seems reasonable.  Maybe.

I don't think ingesting during the lark evaluation is allowed.  No.

Are lark libraries an input?  No idea.

// In general this should probably feel a fair bit like "go generate".  It can be looser with its inputs than our systems otherwise can be.  That's acceptable purely because we're committing the outputs.

## Drafts

See [Review Starlark for an L3+ language](https://www.notion.so/Review-Starlark-for-an-L3-language-58d5f76e959b4775b24cdc11a6a7c790).

```python
import typicalbase from zomg/foo
import addStrace from yonder/larksnips

PlotPatch( # takes varargs, and applies them sequentially.
	typicalbase, # a document.  If you don't start the stack with a doc, it's zero.
	addStrace, # a `func(plot):plot` that adds an strace ware to the apps folder.
)

```

- I think that's not how imports work.  I think there's a Load function or something.
- I wonder if we'll evolve some "pragma" comments or "frontmatter" at the top that say whether this script wants to look at the workspace filesystem or not.
    - We want these to be relatively safe to evaluate — they shouldn't be able to exfiltrate your private keys through a heredoc into a script into the containers they template out.  So the default vision the lark script gets should be not much and very allowlisted.
    - Yet I think a common behavior might be "look at the go.mod file and generate a formula with inputs based on it".
    - Also: possible we might want to put stdlib versioning info in such a header, because it would be easier to programmatically mutate it that way.  (Though this would also have many restrictions, such as not allowing mix-n-match loading of versions.  Maybe allow both and just have this tweak the default?  Seems reasonable compromise.)  Defer this until proven necessary.
- I think we... may want to have output done via sideeffecting methods.
    - We'll still just buffer all those outputs until the end.
    - But this gives a clear answer to "well what if i want to stamp out *lots* of plots in a loop".
- Another option is to make some function names well-known, and say "these are what we will call, and this is what they should return".
    - This approach opens up a lot of room for future extensibility.  Imagine if we want to add "phases" for example.
        - On the other hand: I don't currently anticipate wanting this.
- I dearly hope we end up with something that looks a lot more functional than bazel files do.
    - Outside of functions that do final output emission, *nothing* else should be sideeffecty.

```python
# stdlib: 1.11
# filesystem: allowed
somefunc = load("libzow:15") # these would be looked up in catalogs, I guess?

emit.module(
	name = "example.org/modname",
	plot = PlotPatch(....)
)
```

- Implementing might require some binding libraries.
    - [https://github.com/starlight-go/starlight](https://github.com/starlight-go/starlight) might help but unsure if it's up to date.
    - @warpfork  wrote something about this before, can find and port.
    - Or maybe we just... build a binding to IPLD, which should be short and sweet.
        - Would also let us do some nice custom sauce for accessing either type-level or repr-level of things.  Stretch goal / nice-to-have.
    - There's also a `lib/json` in the go starlark now!  That might be nice.  (Ooh, by b5, too.  We can ask him for gritty opinions.)
    - UPDATE: WE STARTED A NEW ONE:
        
        [https://github.com/ipld/go-datalark](https://github.com/ipld/go-datalark)
        
- Implementation examples:
    - [https://github.com/google/starlark-go/blob/master/starlark/example_test.go](https://github.com/google/starlark-go/blob/master/starlark/example_test.go)

## TODO

Is any of this, frankly, a good idea?  *Unclear.*  The one critical scorecard row is "are we every going to feel a desire to template lark files", and the answer to that **must** be "no".

Should a lark be stuck emitting just one plot and module?  *Probably not*.

^ See [Path Golgi](https://www.notion.so/Path-Golgi-32f6cea9bb8c47b5ba4610d2e5fa4bd6) — larks should *definitely* be able to emit multiple modules.

Does the above imply that splitting plots and modules might've been a mistake?  *Yes quite possibly.*

If a plot is generated, will there be a reliable way to remember (in the same workspaces as the generation was originally performed in, at least) what lark did so?  *I certainly hope so*; we will want this info for any refreshing purposes as well as for GC if a lark eval *doesn't* generate a module this time.