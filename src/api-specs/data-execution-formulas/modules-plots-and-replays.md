---
title: "Modules, Plots and Replays"
layout: base.njk
eleventyNavigation: 
    key: Modules, Plots and Replays
    parent: Data, Execution and Formulas
    order: 40
---

## The Layers

1. Content — filesystems, and their hashes.
2. Execution — hashes go in, something is exec’d, hashes come out.
3. Plans — multiple steps of execution are described in advance.
4. Intentions — a templating system can crank out plans.

## L0: Content

- nuff said.  WareIDs.  That's it.
- A WareID is a content-addressable identifier of a Ware.  A Ware is just a fancy name for (packed) filesystem snapshot.
- A Ware has no implied or presumed structure.  It’s just files.

## L1: Hashed / Crystallized / Executable

- Ready to invoke — all inputs are content-addressed wareIDs.
- No catalog references, nothing.
- Internal graph structure no longer allowed either.  No pipes.  Just one action.
- Is the basis of memoization.  You can hash this whole document.
    - Should have no extraneous non-load-bearing contents, in order for this to work ideally.  No semantic tagging should float down this far.
    - One exception to this: if mounts have been used here, memoization should probably not be applied.
- Mostly not seen by a user, in practice, it turns out.
    - If you engage some real verbose debugging flags, sure.

## L2: Flattened / Resolved

- The replayable.
- Still has catalog references.
- Still can have internal graph structure, multiple actions, and pipes.
- Necessary to be able to attach this form to the catalog, and use it for "`wf catalog explain --recursive foo.org/bar`".
- **This is also (approximately?!) what should be committed in a git repo to mark a project root.**
    - Which provokes interesting questions: that means it should get *exactly one*, very magical, ingest: for the git repo itself, because it must be guaranteed to have that context.
        - and otherwise we have a cycle.  With hashes.  Not possible.
        - does mean if you yank this file out of context, it ceases to actually serve its purpose.
        - Means what belongs in a catalog commitment and what belongs in a git repo project root are almost identical, but have this one critical distinction.
- Generally, if you see one of these, you should also expect to have a catalog in hand at the same time, and both that catalog root reference and this non-crystallized formula should be covered by the same version control repo (or other merkle tree, in the abstract).
- Unclear if this should allow any semantic tagging on structures like input declarations.
    - Seems a bit weird to get non-load-bearing data in catalog commitments.
        - We could always strip any such things out during the commitment process.
    - But probably ultimately harmless.   (...?)
    - May be useful for allowing intentional layer to crank out some stuff, and then letting humans edit this layer, without it creating awkward authorship clarity issues.
        - Unclear if this is actually important.

## L3: Intentional

- Can use ingests.  Can use candidates.  Anything goes.  No one expects this to re-resolve identically unless many conditions are met.
    - (n.b. we still do want reproducible resolve to be possible, it's just... not the dominant UX factor at this level.)
- Ideally these should tend to be pretty standardized if possible, because that's gonna make ecosystem steering easier... but also, there's really no need to be strict about this.
    - We could have several approaches coexist peaceable.  As long as they're capable of communicating results via Catalogs, everything's peachy.
- Lots of questions about how powerful these should be.
    - If this layer has too much power, it will become likely for the ecosystem to drift towards computing L3 being expensive when working with large suites of projects... and we don't really want that.
- Should this have some kind of support for input bundles?  What would that look like?
    - Design research task:
        
        [decide if there's a way to work with input bundles](https://www.notion.so/decide-if-there-s-a-way-to-work-with-input-bundles-253132cbe97e41af912b32f18c1d49f2)
        
    - Lots of questions here, currently unclear.
    - We didn't prove a need this in the last generation, but arguably also this was about where were started falling down in usability, too, so.
    - Revisit: probably not, no.  Still haven't conclusively proved we need this, so, we probably don't.
- One likely outcome: Starlark scripts.
    - More about this in other pages.
        
        [Larks](https://www.notion.so/Larks-059bee14f279479a9f032c4035e69c8f)
        
    - We'll probably both built this into the `warpforge` tool so that there's *something* we can make clear recommendations around using, but we also won't make it required.  Subcommands for getting on and off at L2 should still exist and be perfectly sufficient to get stuff done if you want to bring your own L3.

## Tool editing

### for dependency update propagation

... This is mostly averted.

The candidate system addresses it.

And even if we didn't have the candidate system, it's still point mutations.  Not super systemically concerning.

If we support passing down bundles of inputs, then this might get a tad more interesting.

### for proglang depman integrations

This is the scary one, because it's a very very present practical concern, and it kinda looks like if handled incorrectly, it's going to produce an edit loop that results in blurry authorship lines within a single document.