---
title: "FAQ"
layout: base.njk
eleventyNavigation: 
    key: FAQ
    order: 10
---
## What’s the scope of warpforge?

Warpforge is focused on giving you an *environment* to build software and process data.  That means handling the supply chain — handling the metadata that identifies any compilers, programs, other tools and data sets you need, and provides locations to fetch them from; as well as actually doing the fetching —  and it means setting up a place for those programs to run — in other words, warpforge does containers for you.

Warpforge also gives you a way to say that you want some data *out* at the end of a process.  Warpforge is meant to help you build and create things.

Warpforge does all of this work while heavily emphasizing *content-addressable* primitives.  That means: every time we handle some data, we identify it with a cryptographically secure hash.  (We provide API layers that map human-readable names onto this, to make it usable — but the core is all hashes, all the time.)  Even the execution environment descriptions are content-addressable documents (which means perfect caching — neat!).

Warpforge is primarily focusing on working with *existing* computing technology and existing software and compilers.  In practice: that means we’re using linux containers.  The scope of this may expand in the future, but it’s a very practical place to start.

Warpforge’s scope *does* include building pipelines of computations.  Ultimately, we’d like the tool to be useful for single projects with single git repos, which use either one or many build steps... and scale up to team projects which might be a segmented monorepo *or* a series of separate repos, with multiple separate releasable subunits... and scale up all the way to the size of coordination required to create something like a linux distribution.

Warpforge also includes a few bells and whistles which are just neat and productive and fun.  For example: warpforge can be used as a “localhost CI” very easily.  We include a CLI command that’s meant to go in your bash (or other shell) prompt, which will instantly give you little red or green lights if the project in your current working directory builds cleanly or not.  It’s fun!

Philosophically: warpforge aims to be a tool which allows data to live in the merkle universe, and creates new data natively within the merkle universe.

## Why “another one”?

- Warpforge is for pure functional builds.  (There are a few other things that do this, but it’s a bit rare.)
- Warpforge uses content-addressed storage.  (There are a few other things that do this… but surprisingly few of them *combined* with the other traits we list here.)
- Warpforge is API-driven.  It’s “just JSON” and meant to be easy to use.  (Many other build tools have bespoke DSLs, which can make them opinionated and difficult to compose.)
- Warpforge has a message-passing mechanism — the Catalog API — meant to help you share build instructions with friends!
- Warpsys packaging conventions are aggressively portable, and actively refuse to use any strategies that would result in “distro”-like choices that lead to balkanization.  (E.g., no absolute paths, etc, etc.  Easier said than done!)

## Where’s the source code?

[**https://github.com/warpfork/warpforge**](https://github.com/warpfork/warpforge)

Also you can find our default catalog here:

[https://github.com/warpsys/catalog](https://github.com/warpsys/catalog)

And the source and scripts that produced it here:

[https://github.com/warpsys/warpsys](https://github.com/warpsys/warpsys)

## Is this Open Source?

Yes!  Extremely!  Free forever, and please contribute if you can!

The license is either Apache2 or MIT, at your option.  We’re going for maximum freedom, here.

## What does “hermetic” mean?

Something like “clean room” and “sealed”.

We use to more or less to say “containers” without saying “containers” and getting hung up on the implementation details.  We also usually mean things like “no network access allowed” when we say “hermetic”.

“Hermetic” doesn’t just mean containers.  A VM is also a hermetic environment… as long as you don’t let it fetch unknown stuff; etc.  A Solaris Zone could also be hermetic, and so on.

## Can I add...?

Yes!

### Can I add more ware pack types?

Yep.  We mostly use the one called “tar” right now, because we’ve got it and it’s practical.

But there’s a plugin system here: you can easily develop something new, pick a name for it, and plug that name in wherever you see formulas and wares using the string “tar” right now — there’s some minimal glue code to rig up to demux it, but it’s very doable.

### Can I add more warehouse and transport systems?

Yes please!

We use the scheme field of URLs pretty freely (as you might have already noticed), and so that’s where we specify any other transport plugins.

For example, we already have `https+ca://` as a supported URL scheme for warehouse addresses.

We could also readily have e.g. `ipfs+ca://` or other URL schemes to indicate other plugins.  Contributions welcome!

### Can I add execution models that aren’t linux containers?

*Yes!*

This is a little less paved than some other roads, but we’d love to see it and support it, and we hope to already have set you up for success.

Probably the best way to do this is to introduce a new member to the “action” union — see the API specs in [Formulas and RunRecords](https://www.notion.so/Formulas-and-RunRecords-cb5718bb605e4f799d2bfa119f9c5cf1); that page talks about how to extend the API there.

The “inputs” system already has support for variable names in place of filesystem mounts, which might be another request we’d imagine you might have.

Is this enough to add, say, a WASM environment or something?  We kinda hope so.

Let us know if we can help more!

### How about WASM?

We already said yes, didn't we?

It might look something like this:

```json
{
	"inputs": {
		"thingy": "literal:somevalues"
	},
	"steps": {
		"hello-world": {
			"protoformula": {
				"inputs": {
					"$param": "pipe::thingy"
				},
				"action": {
					"wasm": {
						"interpreter": "wasm2022",
						"contents": "bafybytecodecid"
					}
				},
				"outputs": {
					"out": {
						"from": "$varname",
						"packtype": "literal"
					}
				}
			}
		}
	},
	"outputs": {
		"output": "pipe:hello-wasm:out"
	}
}
```

You may notice that wasn’t wildly different than the way a plot looks for a linux-ish container;

it’s just got `$variables` instead of `/mount/paths`, and the `action` is a `"wasm"` instead of a `"script"`.

## How are we going to build this ecosystem?

**Very carefully.**  There’s a lot to do.

Warpforge itself is staying focused to the sandboxed execution and content-addressable computation thing.  That’s its job.

In practice, there’s a ton of other challenges to making composable computation environments that are nice to work with.  The ball-of-mud image snapshotting approach predominates right now, and that really sucks.  We need to figure out how to get away from it.  (Good tools for working with a [Bill of Materials](https://en.wikipedia.org/wiki/Software_bill_of_materials) don’t help much if no one makes their materials available in a way that can be referred to be the bill!)

See [Ecosystem Conventions](https://www.notion.so/Ecosystem-Conventions-e0edbaf392b4464490bf7ca9c002595b) for a whole forest of information about how we want to proceed.

## How is this different from...?

Hoo boy.  There’s a lot of different fill-in-the-blanks this question could be answered with, and we’re probably not going to be able to answer them all individually.

All that said…

Broadly: warpforge has an emphasis on:

- Content-addressable, and computation-addressable, first.  (”Hash all the things”!)
- A comprehensible [“Bill of Materials”](https://en.wikipedia.org/wiki/Software_bill_of_materials) instead of a ball-of-mud image approach.
- Hermeticism!
- API-driven composability.
- Catalogs of materials that can be snapshotted and synced in a decentralized way.
- Concepts of “workspace” that are friendly to private work, public work, and teamwork — groups of any size, and not necessarily mandating totally public monolithic namespaces.
- Detangling the concept of building a package and running a computation — those are the same tool, working in different modes.
- No strong opinions about what language you use to template computation instructions.
- (… and at the end of the day, some elusive, magical UX.  Something that hasn’t been hit yet, in any other project, evidentally — because nothing’s taken the world by storm yet — and a successful project in this space *should*, because of how useful it would be.)

We don’t believe any other projects have hit all of these at once.

### ... Nix?

❌ ✅ ✅ 😕 😕 😕 ❌ ❌

### ... Guix?

❌ ✅ ✅ 😕 😕 😕 ❌ ❌

### ... Debian or other linux distros?

❌ ❌ ❌ ❌ ❌ ❌ ❌ ✅

(Forgive us for lumping all of these together.  But in general, historically, most linux distros haven’t been focused on build environments, so... yeah, there’s a lot of red x’s here.)

### ... Bazel?

❌ ✅ ❌ ❌ ❌ ✅ ❓ ❌

### ... Runc (or other container systems)?

We use them inside, actually.

Warpforge is pretty much gluing content-addressable data input and output systems onto containers.  (Literally, we template OCI spec files, at the end of the day, and then (usually) invoke `runc`.  It’s a good layering!)

Otherwise:

❌ ❓ ✅ ❓ ❌ ❌ ✅ ✅

### …AppImage?

AppImage is more about packaging than about building, so it’s not really fitting in the checkmarks pattern above.  But it can be compared to some of the [Ecosystem Conventions](https://www.notion.so/Ecosystem-Conventions-e0edbaf392b4464490bf7ca9c002595b), especially [Warpsys Dynamic Library Linking Strategy](https://www.notion.so/Warpsys-Dynamic-Library-Linking-Strategy-65f91fbd20c349eab8352f4c303445dc).

AppImage usually uses SquashFS mounts to bundle all the application’s libraries.  This makes things reasonably path-agnostic, which is good, and the same goal as the Warpsys ecosystem conventions.

The [Warpsys Dynamic Library Linking Strategy](https://www.notion.so/Warpsys-Dynamic-Library-Linking-Strategy-65f91fbd20c349eab8352f4c303445dc) works without needing mounts.  This is nice because it’s less of an ask from the host system; and also because you have the ability to dedup libraries even with simple techniques like symlink farms (whereas AppImage’s squashfs mounts would block such possibilities).

### … some {other thing}?

Alright, look.

Computing has been done before.  Almost every problem has various possible solutions.  Some choices in the details of how to do things matter; some don’t; sometimes its hard to tell which is which until you’ve done it.  We’re here, doing this, because we think some combination of coordinates of the solution space hasn’t been explored yet.

If you think this isn’t worth doing, because some $projectX has already done it, you’re probably the sort of person who says “docker is just lxc”, while entirely missing the point of why docker took the world by storm.  To which we must say: have a nice day :)

## Why are so many of the docs on Notion?

Seealso [What’s this Notion for?](https://www.notion.so/What-s-this-Notion-for-7cc8c063cb294ca99195318e506fe3a4).

- it works;
- it works today/immediately;
- it seems a little more visually pleasing than straight to a readme on github;
- it can be edited rapidly, even on mobile;
- it handles page renames and redirects with zero effort;
- we lean on the collaborative editing features pretty heavily, especially when drafting stuff or taking meeting notes;
- it supports public published as well as unpublished (e.g. draft) content, in the same place.

Still not in love with it:

- one can't really have sane deeplinks (Notion’s links don’t contain parents, and always have long random strings on the end);
- nor link to headings within a page with readable anchors (e.g. can’t `page/#heading` when linking);
- if you fubar a link, it takes you to a login rather than a 404, which is just weird.
- Frankly pretty slow render, even for the readonly version (which you’d really think would cache well, wouldn’t you).
- Closed source, and not particularly free.
- It’s not easy to programmatically get our schemas or examples back out, or do other executable-documentation sorts of things.
- No change history that’s even remotely close to on par with Git.

For now, the pros outweigh the cons.

We’ll probably move to something more self-hosted and git-oriented someday.  (Contributions welcome if you’d like to help with that!)

## Does it support dynamic build graphs?

No, Warpforge doesn’t support dynamic build graphs.  Not per se — when you're invoking `warpforge run`, even with the `-r` flag, warpforge has to be able to plot out all the build graph that needs executing before it begins.

*But:* dynamic evaluation graphs can be done with a higher level templating phase.

(The focus on having outlines of computations before starting to operate is necessary for having a feasible replay system, and the explain and audit features around it, so this is not a design choice made lightly.)

## Why are WareIDs so much more than just a hash?

Recap: WareIDs look like `ware:tar:baMfjE6y3{...a big hash...}aE84fJkma}`.  There’s approximately three pieces to them: the `ware:` prefix; an ID strategy indicator (`tar` in this case), and finally the hash itself.

Why?  Why can’t they be just the big hash part?  Why can’t we skip the colons and the other prefixes?

The primary reason is that a WareID isn't *just* a content ID: a WareID is *also* responsible for saying what the strategy is to turn that content into an actual unpacked filesystem.

(There’s more than one packing and unpacking strategy for mapping filesystems into content-addressed storage.  When we’re handling CIDs: not every CID is pointing at a filesystem.  And we don't particularly want to do fingering detection on the inside of the content the CID points to to figure that out: partly because Principle; partly because ew complex code; and partly because we consider it preferable to have control of the strategy from the outside, rather than end up forced to repackage any content if we want to use a different strategy to unpack it.)

Also: we're fond of having the "ware:" string prefix literal because it gives a visual consistency feeling when compared to the other kinds of values that can go in similar places.