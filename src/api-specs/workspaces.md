---
title: "Workspaces"
layout: base.njk
eleventyNavigation: 
    key: Workspaces
    parent: API Specs
    order: 20
---

(WIP.  See [decide how workspaces and catalogs interact](https://warpforge.notion.site/decide-how-workspaces-and-catalogs-interact-beb3a48a26c7428ea623cbb079f08af3) for early notes, which still need to be extracted to this doc.)

## What are Workspaces?

Workspaces are the main organizational concept in Warpforge and how it exists in your development environment (i.e. your computer).

In brief: you're looking at a workspace whenever you see a folder that contains a folder called `.warpforge`.

## Workspaces and your Development Environment

### How are Workspaces seen on the filesystem?

A workspace is declared wherever there's a folder on your computer that contains a folder called `.warpforge`. For example, if you have the path `/hey/hi/hello/.warpforge`, then the `hello` folder is a workspace.

There are a few files that will be recognized by `warpforge` when they're in a workspace:

- `module.wf` — marks the root of a "module", which, generally speaking, means something's going to be computable here.
- `plot.wf` — contains JSON specifying a "plot" (the "L2" graph format, can have multiple steps of computations, can refer to catalogs, etc).  Will be ignored unless a `module.wf` file has already been found.
- `lark.wf` — contains skylark script (an "L3" system) which generates the `plot.wf` file.  Will be ignored unless a `module.wf` file has already been found.
- Data under `.warpforge/*` are for files that the `warpforge` tool is going to read and write. This includes but may not be limited to catalogs, warehouses, memos, etc.

Beyond those, the rest of the files and folders are yours — they're just content in the workspace.  This content won't be examined by warpforge unless it's pulled into the system by ingest commands or similarly explicit interactions.

## How do Workspaces relate to each other?

Workspaces in your development environment can be nested.

When this occurs, the workspace "above" (e.g. has a shorter path) the other is called its parent.  Any workspaces "below" (e.g. have a longer path) another are called its children.

`warpforge` commands will generally work on the nearest workspace to them.

In other words, if your current directory is `/hey/hi/hello/yo/you/`, and there's a workspace indicated by `/hey/hi/hello/.warpforge` and also one indicated by `/hey/hi/.warpforge`, commands you issue right now will relate to and affect the `hello` workspace.

There's a couple operations that do reach across workspaces.  These mostly relate to catalogs.

For example, there are commands which will add content to the current workspace's catalog.  In order to make that easy, those commands can look at catalogs in the parent workspaces, and pull content down through them into the current workspace.  In addition to reading data from the parent (or grandparent, etc) workspace, these commands will also update any workspaces up the parentage line (if it fetched data from a grandparent or above).

There is also something called a "root workspace".  A root workspace is one above which no further parents will be detected.

Everyone has at least one root workspace, their home workspace — which implicitly exists at `$HOME/.warpforge/`. 

See [What is the Root Workspace?](https://warpforge.notion.site/Workspaces-7d27dd769ba74440989b101cc14157c6) for more details on the concept of root workspaces.

## How do Workspaces relate to Modules?

It's fine to have more than one module in a workspace.

Modules can be at any position inside a workspace.  They can be in the workspace root directory (and often are), or in any subdirectories, recursively.

(We care about at exactly one catalog being defined for a module, such that both are in the same VCS root.  It's fine if that's a one-to-many.)

## How do Workspaces relate to Version Control Systems?

As a general rule, every VCS repo (e.g.: every git repository)... should have a workspace in it.

And, as is normal for a workspace, that workspace should have a catalog directory, and that should contain a vendored copy of relevant parts of the catalog.

Typically, it makes sense for the workspace root dir to be the same as the VCS root dir.

So in effect, we should see a `.warpforge` dir in every repo on github. And a `.warpforge/catalog` dir inside, which then contains info on every lineage of data that's used any of the modules in the VCS repo.

The reason that we say every VCS repo should have a workspace is so that when someone clones your VCS repo, they should always have enough information to reproducibly execute any modules within the VCS repo.

When the VCS repo contains both the module (which says what is to be computed, by name), and the catalog info for all things the module refers to (which is the info for resolving names into `WareID`s), then that goal is achieved.

`warpforge` commands will generally try to give you nagging warning messages if it sees a VCS root that does not contain a workspace.

## What is a Root Workspace?

A root workspace is simply one which, because it declares it self as a root workspace, has no parents.

When `warpforge` commands are discovering workspaces, they simply stop looking for more parents when encountering a workspace that's marked as a root workspace.

Root workspaces also reject having any modules directly within them.  They'll require you to make at least one child workspace, and put modules within that.  This is because for any work you ever want to publish or share, you need some workspace that you can... publish, and share!  A root workspace is not generally suitable for that purpose, because it can contain personal configuration, or even key material (e.g. if you interact with remote warehouses that require authentication, etc), and will also sometimes contain the local warehouse (which is big!)... and so, we force you to make at least one child workspace, because that can then be relied upon to be clean and presentable.

### What's atypical about a Root Workspace?

Aside from the key fact about having no parents and allowing no modules...

- Root workspaces are where memoizations records pool.
- Root workspaces are where local warehouses are (or are configured from).
- Root workspaces are where local unpack caches are (or are configured from).
- Root workspaces can have more than one catalog!

A root workspace has a `.warpforge/catalogs/` directory — *note the 's'*.  This is in contrast to the usual `.warpforge/catalog` dir.

The `.warpforge/catalogs/*` directories allow putting several catalog roots next to each other.

This can be used to, for example, put a public catalog next to a personal catalog next to a workplace catalog, and be able to refer to data in any of them.

The existence of multiple catalogs does leave some behavior undefined — e.g., what if there are name collisions between the lineages in the different catalogs?  As a general rule, we make those events require human intervention.  For example, the lineage name collision problem — though we hope the community avoids this with cultural norms — is ultimately addressed by requiring a child workspace, which removes the ambiguity by being required to vendor one of the two catalog entries, and forcing a human to make the choice at the time the child workspace is first updated to include that reference.

### What is the Home Workspace?

The home workspace is the one workspace that everyone in every environment has.  It implicitly exists at `$HOME/.warpforge/` (or the semantic equivalent for your platform, if it's not unixy). 

(Even if the `$HOME/.warpforge/` folder does not exist, most `warpforge` commands will act as if it does and it has default content, and will create it and write new files there if appropriate.)

The home workspace is automatically your root workspace, if you don't have any other root workspace.

Like other root workspaces, the home workspace also has the special rule about actively disallowing you to have any modules within it.  You must make at least one other workspace.

The home workspace is also sometimes used to store some user configuration.

Of course, such "user configuration" may never be something that affects how catalog resolution works, how plot evaluation works, how formulas and the hermetic computations work, etc.  Only decorative configuration is considered.

### Can I have more than one Root Workspace?

*Yes*.  You can declare any workspace to be a root workspace.

This can be useful if you want to create a dedicated, isolated environment with only its own catalogs.

Creating a new root workspace for logical isolation can be overkill, because `warpforge` is already fairly diligent about cleanly vendoring data, and only the relevant subsets of it.  However, it may still be useful "just to make sure" (e.g., preventing slip-ups where one accidentally refers to a private work project from a personal project, or vice versa).

Creating a new root workspace also makes it possible to configure different local warehouses and caching directories, which can be useful if any of the data being handled is extremely private, or, for simple operational/sysadmin/quota reasons.

### Can I nest a Root Workspace in another Root Workspace?

Yep.  It’s actually not very special.  (Which is a good thing.  It’s not “odd”.)

The workspace search stops at the first root it finds.  The one that’s nearest to your cwd has effect, and the other one doesn’t.

In an example: you can have `/foo/.warpforge/root` and `/foo/bar/baz/.warpforge/root`. If you're in `/foo/bar/baz/natch/`, your root workspace is going to be `/foo/bar/baz/`, and that's the end of it.

## FAQ

### Holy smokes, how many workspaces will I have?

Not *that* many.  At least not in depth.  We don't expect the recursion feature to be too heavily used, especially not for individuals working on small projects.

Most developers starting to use the system will have two depths:

1. their home workspace, which is a root workspace.
2. the workspace in each of their git repos.

... and that's it.

If you work on a *lot* of projects, you might have a lot of workspaces.  You'll have at least as many workspaces as you have VCS repos.

But the workspace parentage doesn't have to get *deeper* just because you have more of them.  Don't start doing grandparent workspaces unless you decide it actually helps you.

If you're working in a team in a work environment, or, organizing something really large which has many components managed by different authors (e.g. a distro of some kind), you'll probably add another workspace in the middle.  That workspace may or may not want to flag itself as a root, depending on how you decide to work.

Mostly, how many workspaces to use, and how much to nest them, should be a personal choice.

At most, it becomes a team choice, if you work with a team that decides they want to share a parent workspace configuration.

It's *never* a global choice.  (This is important!)  **There is *no need* to have the same workspace parentage setup as *anyone else in the world*, even if you're collaborating with them on a workspace and module.**  The only things you need to stay in sync on are the vendored parts of catalogs — which is local to *that* workspace, and not its parents, by design.

### Okay — *how* should I use these?

It depends.

First of all, we ALWAYS recommend having your workspace in version control.

(Although maybe not your home workspace.  That's up to you. 
Maybe version it with the rest of your "dotfiles", if you're the kind of person who does that.)

Beyond that, it depends on:

- How much work you're doing,
- How many people you're doing it with,
- and How do you want to coordinate and organize it?

### How should I use these as a solo developer on a small project?

You’ll already have a home workspace, naturally.  You’ll probably fetch some publicly curated catalogs and store them there, so you can build on what other people have already packaged.  `warpforge catalog update` will do that for you.

Make another workspace in the VCS root for your project.  `git init` and `mkdir .warpforge` in the same directory, typically.

Make a module file in you project, too, probably in that same dir.  `warpforge quickstart` will make a nice starter file, but you’ll want to customize it from ther.

Your module will probably want to have one input which says roughly: `"src": "ingest:git:.:HEAD"` (assuming that you’re using git). This incantation will cause Warpforge to take your latest git commits and stick those into the filesystem in the sandbox.

Consider writing multiple steps in your module if it seems appropriate, e.g. a step for building and a step for testing.

When you're done writing your module, run `warpforge workspace sync` to make sure any catalog references you used are vendored into the workspace's local catalog, so it's ready to use freestandingly.

Commit both the module file and the workspace directory to your version control system.

Build and be fruitful!  

`warpforge run` should now build your project!

You can readily use `warpforge watch` on this project now, too.  It should give you the joyous experience of [localhost](http://localhost) CI! 🤓

And when you want to make "releases" of things, so other people can import your project's outputs via catalog references, use `warpforge module release`!

### How should I use these on a bigger project?

Much the same as for a small project — but feel free to use multiple modules in the same workspace.

(In the future, we hope to also support scoped git ingests, which could make some common patterns of use easier to define, and potentially also give better memoization, but this is not yet available.)

### How should I use these when my work spans multiple VCS repos?

You'll probably want to consider making a parent workspace for them, if you'll be putting any effort into making sure these projects are relatively closely coordinated.  You don't have to, though.

TODO expand on how candidate publish scope works, how workspace-boxed downstream testing can help, etc.

### How should I deal with other projects that have a workspace in their VCS that has contents I want to override?

That's relatively well-supported, we'd like to think!

You can write a module file in your workspace that refers to another module file and overrides certain parts of it (specifically, the inputs).

That module will still otherwise act (in terms of ingests, etc) as if it was in the place of the overridden module... And yet in your workspace (i.e. workspace is determined by where your override module file is).

If you need more invasive changes than that.... That's called "forking", I'm afraid; or "patch management".  There's only so much we can do. 🤷