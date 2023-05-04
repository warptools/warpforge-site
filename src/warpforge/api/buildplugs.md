---
title: "Buildplugs"
layout: base.njk
eleventyNavigation:
    key: "L3+: Buildplugs"
    parent: "Warpforge API"
    order: 50
---

Higher level composition of Plots and Modules
=============================================

(aka, "Buildplugs")
===================

Building lots of software, or large numbers of data analysis pipelines, requires higher level compositional tools.
It's important to have a way to say repeated things once, so that we can focus our energy on the important, interesting, unique parts of the domain we're working on.

Warpforge focuses on being drivable by declarative APIs, and while that is good for reliability and having simple, standardized ways to debug and audit things...
it's not so great for high level composition.
Declarative APIs, by their nature, tend to involve saying things very methodically -- and often, this can mean quite _redundantly_, when building large systems.

We solve this problem by adding layers:

- Warpforge [Plots](/glossary.md#plot) (which you've probably already read about by now)
are the declarative, methodical API object -- something which needs no further expansion, and has a sharply limited ability to reference any other data.
- Warpforge "Buildplugs" (which this document is about) are higher level scripting systems that _produce_ plots --
and in buildplugs, we relax the expectations of declarativity, and let them pull in more information sources, so that we can get more done with terser writing.



Warplark Buildplugs
-------------------

The "batteries-included" buildplug system supported by Warpforge is called "Warplark",
and it uses the Starlark language.
(Starlark is a python dialect, so it's pretty easy to pick up for most users --
and, we bundle the complete interpreter, so it's always available and requires no additional setup.)

When using Warplark, users write Starlark scripts that assemble python data objects
that have the same shape as [Plot](/glossary.md#plot) API objects.

Within Warplark, adding inputs is as easy as assigning new keys in a map.
Users can write functions that combine related or commonly repeated constructions,
like adding common sets of related inputs at once,
perhaps while also modifying the `action.script` list, and so on.

:::todo
Much more documentation required here.

- Example lark: https://gist.github.com/ericevenchick/31bf340e0a4bb278c82dbce1dd8350d0
:::



Other Composition Tools
-----------------------

Warpforge supports a simple and open-ended API for composition tools.
The starlark features that Warpforge supports natively are not special,
beyond the fact that they come bundled "batteries-included" in the standard distribution of Warpforge.

You can design your own composition tools!

:::todo
- document the exec API and the stdin/stdout expectations.
:::

### Considerations when designing a Composition Tool

If designing a composition tool, here are some things you may wish to keep in mind:

- The output of your composition tool will still be a plot.  Specifically, the inputs in the resultant document need to be fully named.
	- You can't emit a version range or any kind of wildcard!  Plots are sorta like "lockfiles" in this regard.
- Be careful to be as explicit as possible about what scripts in your tool should have access to.
	- You probably want your composition tool to be as safe as possible for someone to evaluate, even without additional isolation.  So, for example, the tool shouldn't give a script the ability to exfiltrate your private keys through a heredoc into the description of the containers they template out.
	- On the other hand: your tool may want to have access to files in the workspace (for example, perhaps it wants to look at files handled by other language-specific package managers, and use those as inputs).
	- Pick an isolation-vs-utility balance, document what it is, and stick to it!
- You _can_ expose read access to the whole catalog system to your scripts, and implement version "wildcards" and "ranges" in your scripting system (or allow user script to do so)... but consider carefully whether that's a good idea.
	- If scripting logic is used to select versions of inputs, then that makes it quite difficult, perhaps even impossible, to create automatic "version bumping" tools that operate from outside the script!  That might obstruct a lot of cool tooling automation possibilities.
- Your users will probably want some sort of library loading features.
	- Be as careful with this as with other filesystem exposure.
	- It's probably a good idea to document what kind of relationship is recommended for those files versus a project's version control borders.
- Be mindful of what operations in your scripting language (and standard library) are commutative (e.g. the order doesn't matter).
	- More commutative generally means simpler and better for end-users.
	- This is easier for some topics than others!
		- Appending inputs is generally commutative (sorting the inputs map is usually fine).
		- Appending an `action.script` list is _not_ commutative by nature -- so this could produce interesting challenges!
		- Appending certain common env vars like `$PATH` is _not_ commutative by nature -- so this could produce interesting challenges!
			- (You may note this is why we have tools like `linkwarp` in the ecosystem!  It adjusts another part of the playing field so we don't have to fight with `$PATH` composition at all wherever `linkwarp` can be used.)
- You may want to have a composition tool that can take an existing Plot object and apply modifications to it.
	- This poses some up-front design challenges when creating the composition tool... but will make it possible to compose with _other_ composition tools, or tools that work purely on manipulations of serialized Plot objects.



History
-------

Buildplugs have formerly been known as "larks", as a placeholder name, during some parts of the evolution of the Warpforge project.
We moved to the current name because we didn't want to imply that buildplugs _must_ be in starlark
(even though our first implementation was),
and because "larks" didn't auto-explain itself very well in the first place.

<!--
For some much older research notes, see:
- [Review Starlark for an L3+ language](https://warpforge.notion.site/Review-Starlark-for-an-L3-language-58d5f76e959b4775b24cdc11a6a7c790).
- [understand user story for generating inputs based on proglang](https://warpforge.notion.site/understand-user-story-for-generating-inputs-based-on-proglang-dcde115cc8a141d3a255ddc3aa345d61) should be solved by this — ingesting someone else's things probably needs a tool that's "templated" somehow, which means we should be able to see it up here, probably.
-  [decide if there's a way to work with input bundles](https://warpforge.notion.site/decide-if-there-s-a-way-to-work-with-input-bundles-253132cbe97e41af912b32f18c1d49f2)
-->
