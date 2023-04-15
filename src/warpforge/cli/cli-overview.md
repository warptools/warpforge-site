---
title: "CLI Overview"
layout: base.njk
eleventyNavigation: 
    key: CLI Overview
    parent: Warpforge
    order: 40
---

CLI Overview
============

:::todo
This page is in need of a refresh.

Our future aim for this sector of documentation is to mirror these online docs with what's available in the `warpforge` CLI,
and ensure both of those are sufficiently complete and robust to help you find what you need, either online or offline.
:::


## Meta

For contributors: [https://clig.dev](https://clig.dev/) has some pretty good recommendations about CLI UX goals.

## Command list

This page is just a quick list of every command we've thought we'll need, so we can look for consistent patterns.

Not all of these are currently implemented!  (We wanted to look forward to functionality we anticipate building, in order find naming patterns that will stay firm during growth

**Existing commands**

- run
- check
- catalog →
    - init
    - add
    - release
    - ls
    - show
    - bundle
    - update
    - ingest-git-tags
    - generate-html
- watch
- {status,info}
- quickstart
- ferk

### Execution

Execution commands can do fileset mutations. These mutations may have records, caches, and memoizations stored in a workspace.

- `warpforge quickrun` — launch a container with your whole host mounted ro, and the cwd mounted rw, and the container shell launched in that same cwd.  Details configurable with config files in your home workspace.
- `warpforge quickbox` — similar to `warpforge quickrun` but starts with a minimal root image instead of your whole host.  Details configurable with config files in your home workspace.
- `warpforge run [file | pattern]...`  — looks for either a module or a plain formula and Does The Thing.  Or take an entire pattern (e.g. "`./...`"), in which case we're looking for modules, and are gonna do a bunch of fun stuff.
    - This has many, many options.  In particular, you must say if it's allowed to do non-hermetic things, like `--net=allow` and `--mount=allow` and `--ingest=allow`.
- `warpforge impact` — makes changes on your host!  Runs some module or formula, and then, upon success, will unpack its outputs onto your host!
    - Can be configured with where to put stuff: it's much like the input spec for a formula.  (Additionally, it also allows relative paths!)
    - Has a built-in support for "atomic mode" (unpacking to a tmpdir and updating a symlink).
    - Impact profiles can be shipped alongside a module... But are only recommendations.  To take effect, they must be copied into your root or home workspace.  (This is similar to how gitattributes config works, and for similar reasons.)
    - N.B. not sandboxed even a little tiny bit!  Can totally be used to overwrite content in your workspace.  (Which can be cool — if used carefully.)

### IO

These commands work on wares and warehouses. A default warehouse may be provided by a **workspace**.

- `warpforge ware pack --type=<packtype> <targetpath>`
- `warpforge ware scan --type=<packtype> [--nokeep] <targetpath | URL>`
- `warpforge ware unpack <packtype:hash> <targetpath>`
- `warpforge ware describe <packtype:hash>`
- `warpforge warehouse mirror [--dest=<whaddr>] [--src=<whaddr>][...] <packtype:hash>`
- `warpforge warehouse list` — list all the wareIDs in the warehouse.
- `warpforge warehouse path <packtype:hash>` — say what filesystem path the given WareID is at.  Not often used by humans, but occasionally handy for scripts.  Sometimes an approximate answer; use with care.

### Utility

- `warpforge workspace sitrep` — should report a whole bunch of things: what modules are in this workspace, when they were last built, what the success was, which were most recently built, whether any of those builds are no longer up to date due to candidates updating, etc, etc...
This seems like a more complicated `status` and maybe we just make status very, very flexible.

### commands that work without a workspace

- `warpforge formula check [files...]` — should just check a formula for syntax and logical sanity, but not run it.
- `warpforge formula check --fmt [files...]` — reformat the thing.
- `warpforge formula template [file]`  — spits out a quick dummy formula into a file, ready for you to edit.
- `warpforge find workspaces [pattern...]` — search the filesystem for more workspaces.  Just lists them (meant for use in scripting).
- `warpforge find modules [pattern...]` — search the filesystem for modules.  Just lists them (meant for use in scripting).  Does not include modules that are within a child workspace by default.

### commands that require a local workspace

- `warpforge workspace init` — Makes a .workspace directory.  It won't commit with git without any *files* in there though so that might be something to fiddle with.
    - `warpforge workspace init --root` — Creates a root workspace.
- `warpforge workspace release` — needs definition (may turn out to be a multi-step command sequence).
- `warpforge workspace clean memos` — delete memoization records.  With options: those over a certain age, any with a certain wareID in the inputs, etc. Both local and root by default.
- `warpforge workspace tidy` — (or `sync`?) ensure the workspace catalog has all the data vendored that it needs to have in order for the modules in the workspace to evaluate.  (and optionally, is trimmed to have nothing more.)
    - Modules in the workspace need to be discoverable. Default to recursive traversal to find modules in the workspace. Cannot call tidy for a single module.
    - Need to distinguish between catalog items that are part of the workspace and those that are not.
    - Historical information is expected to be tracked by VCS. The tidy command will only track dependencies that are currently in use.
- `warpforge workspace watch` — looks for a module, and expects it to have exactly one ingest of a kind that is known to be pollable (namely: git).  Proceeds to act like a CI service.
    - With more args: can also be instructed to push the results of the its main focus module into other modules that consume that module name, and test the candidate outputs in those too.
    - Recurse upwards until it finds the root workspace (or a local workspace?)
    - Having exactly one pollable ingest is an arbitrary limitation for now. Potentially specifying which ingest to poll or tracking multiple ingests seems like reasonable feature expansions.
- `warpforge spark` — emits tiny status emoji relating to the cwd by attempting to find a socket to a `warpforge watch` instance.  With some configuration: can talk about `warpforge sitrep` data in general.  Colorful!  Joyful!  Delights!

### **commands that can operate without a local workspace, but will interact with the root workspace**

- `warpforge catalog add <name> <URL>` — add a new catalog... to your *root* workspace.  (You'll still use `warpforge catalog sync` to get any relevant segments of copied into your project workspaces.)
- `warpforge catalog lineage create` — creates a lineage, may involve key generation or fancy stuff.
- `warpforge catalog check` — check syntax
- `warpforge catalog check --fmt` — reformat the thing.
- `warpforge catalog search [namefrag]` — search all catalogs in the root workspace for any lineages with a name that contains the fragment.
- `warpforge catalog search --fulltext [text]` — honestly just use grep though?
- `warpforge catalog follow <name> <URL>` — alternative name for "add" to consider
- 
- `warpforge module check` — check a module/plot for syntax and logical sanity, but do not evaluate it.
- `warpforge module check --fmt` — reformat the thing.
- `warpforge module graph` — spit out a dot file of relationships between steps in the module's plot.  Optionally: do it recursively, looking up the replay plots for anything this module imports that has that information available in any of our available catalogs.  Optionally: do it recursively, but only for modules that are in this same workspace.
- `warpforge module run` — an alias for `warpforge run`.
- `warpforge module release` — run, and, drag us through a release tagging process afterwards.  Modifies the proximate workspace's catalog (publishing it up to other catalogs is a separate business).
