---
title: "Warpforge API Layers"
layout: base.njk
eleventyNavigation:
    key: "Layers Overview"
    parent: api
    order: 10
---

Warpforge API Layers: an Overview
=================================

:::todo
Content in this page is from older documentation, and needs rewriting.

It's offered here because something is better than nothing,
but we're aware it's not easy to read.
Additionally, some content is described as speculative,
even though it has now been implemented since this document was last updated.
:::

## In superbrief:

1. Content — filesystems, and their hashes.
2. Execution — hashes go in, something is exec'd, hashes come out.
3. Plans — multiple steps of execution are described in advance.
4. Intentions — a templating system can crank out plans.

## L0: Content

- nuff said.  [WareIDs](/glossary.md#wareid).  That's it.
- A WareID is a content-addressable identifier of a Ware.  A Ware is just a fancy name for (packed) filesystem snapshot.
- A Ware has no implied or presumed structure.  It's just files.

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
- Concretely: we support this via "[Buildplugs](./buildplugs.md)".
    - This is an open-ended extension point -- any process that can produce JSON can be used here.
    - There is also a starlark buildplug, which we provide as a batteries-included option.
