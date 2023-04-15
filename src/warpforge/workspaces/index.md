---
title: "Workspaces"
layout: base.njk
eleventyNavigation:
    key: Workspaces
    parent: Warpforge
    order: 40
---

Warpforge Workspaces
====================

Workspaces are the directories on your host that you operate Warpforge in.
Various files in a workspace are used configure Warpforge's behaviors,
and some also store state, caches, and so on.

You generally need at least one workspace to use most features of Warpforge.
Commands like `warpforge quickstart` will create a workspace automatically.

Workspaces are detectable by looking for a `.warpforge/` directory.

There are two flavors of workspaces:
one is project oriented (e.g. you'll typically have one per source code repository),
and the other is a "root" workspace (which you can use to configure isolated name resolution scopes).

---

[[TOC]]

---



Purposes of Workspaces
----------------------

Workspaces are, well, a space for working in.

They're a directory that contains one or more modules (which specify some work to be done),
and also a workspace control directory which contains some information about...

- what sorts of names are in scope in this workspace (e.g., it contains Catalogs),
- configuring where some host integration details like filesystem snapshot caches are stored (in root workspaces),
- and various other pieces of useful data storage, such as memoization of previously evaluated computations.

A workspace also contains user content.
Modules within the workspace can refer to this content by using features like "ingest" inputs.



Workspaces on the Filesystem
----------------------------

### Detecting Workspaces on the Filesystem

A workspace is any directory that contains a directory called folder `.warpforge`.
For example, if you have the path `/hey/hi/hello/.warpforge`, then we say the `hello` directory is a workspace.

There's also one special workspace, called the "home workspace",
which is in `~/.warphome`.

### Special Files in a Workspace

Generally, workspaces are expected to be mixed in with your project source code,
and version controlled.

There are a few files that will be recognized by `warpforge` when they're in a workspace:

- `module.wf` — marks the root of a "module", which, generally speaking, means something's going to be computable here.
- `plot.wf` — contains JSON specifying a "plot" (the "L2" graph format, can have multiple steps of computations, can refer to catalogs, etc).  Will be ignored unless a `module.wf` file has already been found.
- Data under `.warpforge/*` are for files that the `warpforge` tool is going to read and write.  Things stored here can include [Catalog](/glossary.md#catalog] data, warehouses and caches, output collection, memoization data, etc.

Beyond those, the rest of the files and directories in a workspace are yours — they're just content in the workspace.
This content won't be examined by Warpforge unless it's pulled into the system by ingest commands or other similarly explicit interactions.



Relationships of Workspaces
---------------------------

### Workspaces vs Modules

A module needs to be within a workspace.
The workspace holds [Catalog](/glossary.md#catalog] data
(as well as configuring where a bunch of caching paths, etc, are),
so a module can't really be evaluated without an enclosing workspace.

Many modules can be within the same workspace.

Modules can be at any position inside a workspace.
They can be in the workspace root directory (and typically are), or can be in any subdirectories, recursively.


### Workspaces vs Version Control

As a general rule, every version control repo (e.g.: every git repository)... should have a workspace in it.

And, as is normal for a workspace, that workspace should have a catalog directory, and that should contain a vendored copy of relevant parts of the catalog.

Typically, it makes sense for the workspace to be seen in every VCS root dir.
(In other words: `.warpforge/` and `.git/` dirs are often siblings!)

The reason that we say every version control repo should have a workspace is so that when someone clones your repo, they should always have enough information to reproducibly evaluate all of the modules within the repo.
When the repo contains both the module (which says what is to be computed, but is using human-readable names like catalog references),
and also has the the catalog info vendored for all things the module refers to because of the workspace files, then that goal is achieved.

`warpforge` commands will generally try to give you nagging warning messages if it sees a version control root that does not contain a workspace.


### Workspaces vs other Workspaces

You can have lots of workspaces.

As you've read above, it's typical to use one workspace for every "project" (whatever that may mean to you), or more concretely, for every version control repo.

You can also have more than one _root_ workspace.
When using root workspaces, any workspace nested inside them becomes partially controlled by that root workspace.
This means that things like what [catalogs](/glossary.md#catalog) are used,
what host system paths are

:::info
Possibly future features may include other relationships for nested workspaces,
such as making "candidate" release names that can be bubbled up and down in concentrically limited scopes.

If you'd be interested in a feature like this, please join one of the [community](/community.md) channels to discuss your thoughts!
:::


### Do I have to have the same workspace nesting as other contributors to the same projects?

Nope.

There is *no need* to have the same workspace parentage setup as *anyone else in the world*, even if you're collaborating with them, either directly or temporarily.
The only things you need to stay in sync on are the vendored parts of catalogs -- which are local to *that* workspace, and not its parents, by design.



Root Workspaces
---------------

A root workspace is one which:

- contains multiple [catalogs](/glossary.md#catalog), and the definitions for how to resolve name lookups within them.
- contains local warehouses (or configures where to find them).
- contains local unpack caches (or configures where to find them).
- contains local memoization data caches (or -- you get the drill! -- configures where to find them).


### Root Workspaces cannot contain Modules directly

Root workspaces also reject having any modules directly within them.
Warpforge will require you to make at least one child workspace, and put modules within that.
This is because for any work you ever want to publish or share, you need some workspace that you can... publish, and share!
A root workspace is not generally suitable for that purpose, because it contains bits of configuration that are about how to interact with the host system,
or even key material (e.g. if you interact with remote warehouses that require authentication, etc),
and also the root workspace directories may sometimes contain the local warehouse (which is big!)...
and so, we force you to make at least one child workspace, because that can then be relied upon to be clean and presentable.

This typically isn't noticed by a new user, due to the home workspace being the root workspace,
and being implicitly created as necessary.
However, you may notice this when getting into more advanced usage and creating other root workspaces.


### Declaring a Root Workspace

A root workspace is declared by presence of a `.warpforge/root` file.

This workspace will now also contain directories like:

- `.warpforge/catalogs/`
- `.warpforge/cache/`
- `.warpforge/warehouse/`
- `.warpforge/memos/`
- and others.

:::info
Future features may include adding support for a `.warpforge.local/` directory split,
which could further separate things like catalog setup (which an organization could potentially want to version control and share!)
from things that are always host-specific (such as cache paths).

If you'd be interested in a feature like this, please join one of the [community](/community.md) channels to discuss your thoughts!
:::


### When to use multiple Root Workspaces

Having more than one root workspace is primarily useful for creating a dedicated, isolated environment with only its own catalogs and its own scope for name resolution.

Creating a new root workspace for logical isolation can be overkill, because `warpforge` is already fairly diligent about cleanly vendoring data, and only the relevant subsets of it.  However, it may still be useful "just to make sure" (e.g., preventing slip-ups where one accidentally refers to a private work project from a personal project, or vice versa).

Creating a new root workspace also makes it possible to configure different local warehouses and caching directories, which can be useful if any of the data being handled is extremely private, or, for simple operational/sysadmin/quota reasons.


### The Home Workspace is a Root Workspace

The home workspace (`~/.warphome`) is always a root workspace.
Any workspace which doesn't have some other root workspace in a parent directory of itself will default to treating the home workspace as its root workspace.
(As a result, users typically can start working by just declaring one workspace,
and the home workspace handles being the root workspace implicitly.)

The home workspace also contains some other user configuration and customization data, but is otherwise a regular root workspace.


### Multiple Catalogs

Root workspaces contain multiple catalogs.

(This is a big contrast to regular workspaces, which contain exactly one catalog dir,
which contains vendored copies of the information relevant to that workspace's modules.)

Multiple catalogs are used by placing them the `.warpforge/catalogs/*` directory.
Each directory within this one is expected to contain another [catalog filesystem](/catalogs/catalogs-on-the-filesystem.md),
and the names of those directories become the local alias for that catalog.

Multiple catalogs can be used to, for example, have a public catalog as well as to a personal catalog as well as a catalog that's used for some company workplace content... and be able to refer to data in any of them.

The existence of multiple catalogs does leave some behavior undefined — e.g., what if there are name collisions between the lineages in the different catalogs?  As a general rule, we make those events require human intervention.  For example, the lineage name collision problem — though we hope the community avoids this with cultural norms — is ultimately addressed by requiring a child workspace, which removes the ambiguity by being required to vendor one of the two catalog entries, and forcing a human to make the choice at the time the child workspace is first updated to include that reference.


### Can I nest a Root Workspace in another Root Workspace?

Yep.  It's actually not very special.  (Which is a good thing.)

The workspace search stops at the first root it finds.
The one that's nearest to your cwd has effect, and the other one(s) don't apply.

In an example: you can have `/foo/.warpforge/root` and `/foo/bar/baz/.warpforge/root`. If you're in `/foo/bar/baz/natch/`, your root workspace is going to be `/foo/bar/baz/`, and that's the end of it.
