---
title: "Bifrosts"
layout: base.njk
eleventyNavigation: 
    key: Bifrosts 
    order: 70
---
A “bifrost” is what we call some piece of tooling that connects other non-merkle package management ecosystems (e.g. npm, pypi, maven, etc, etc…) to a merkle-native ecosystem like Warpforge.

Generally, a Bifrost looks at whatever data sources exist for a non-merkle package management system, “scrapes” this info together by whatever means necessary, and produces (or updates) Warpforge [Catalogs](https://www.notion.so/Catalogs-d63c788547794e80bf05de9195505aaf).

(While the Warpforge [Catalogs](https://www.notion.so/Catalogs-d63c788547794e80bf05de9195505aaf) format was made for Warpforge first, we have tried to keep it unopinionated — it’s just a merkle datastructure and should be reusable by any project!)

To make this topic managable, we break it down a bit:

“What a Bifrost is Not” covers a little bit of essential disambiguation.

“Bifrost API Definitions” talks about how we might make bifrosts in small pieces that communicate with simple messages.

“Bifrost Success Levels” talks about the goals we have, in increasing order of excellence (but also increasing level of development effort required.

“Known Bifrost Tools” — todo; please contribute info about things you know of!

## What a Bifrost is Not

Quick disambiguation: a Bifrost is about bringing foreign (typically non-merkle) package managers and their content into the fold of a merkle-native system (mostly, namely, Warpforge).

Drilling out of a Warpforge container with a readwrite host mount, and using it for “caching”, is generally considered a bit of a hack.  And a fine hack, if it makes you productive and doesn't cause you harms.  But also: not a bifrost.  A mount hack that allows persistence outside of any warpforge pipeline control and hashing isn't really helping advance the ecosystem in any way.

Bifrosts don't necessarily mirror full content.  They should focus first on metadata.  (Mirroring full content should be pretty trivial after the metadata is all gathered.  And admittedly, for some systems, the Bifrost tooling might end up downloading the complete dataset just to be able to fully index it.)

Bifrosts don't need to re-expose the API of whatever foreign system they gathered data from.  A Bifrost is first and foremost responsible for just gathering and merklizing metadata — because that's the necessary+sufficient prereq for being able to instruct a system to Mount those files somewhere at job container initialization time.

## Bifrost API Definitions

This is an open topic of debate and research 🙂

Current suspicions:

- It’s probably useful to have tools that scrape the non-merkle systems, and produce directories of the package filesystems (or directories of tarballs if things are easier to treat pre-packed; *handwaving*) plus the names of those packages and versions (and optionally some additional files which contain whatever metadata that system supports).
    - This would let filesystem hashing tools be applied over this result, which is a nice separation of concerns.
- It seems necessary to emit some set of mount entries — being pairs of paths (or at least path suffixes!) and catalog refs (or ware literals; whatever) — from a bifrost tool that's anywhere at 2-stars or above.
- …which would also imply we're going to need a function to help ram such entry sets together (okay, no surprise)
- …and would also imply we almost certainly need to either be able to invoke these things during L3+ planning evaluations; or at minimum ask for them to have been invoked, and get the data they produced.

## Bifrost Success Levels

There’s varying levels of hackiness with which one can approach this problem.

For sheer package content snapshotting and labelling itself:

- 0 stars — allowing network in the job container environment, and letting an unknown tool do unknown things of unknown reproducibility, and doing this every time even for the same setup situation (n.b. even if caching locally, this is still zero stars because caching isn’t sharable).
- 1 star — ⭐ — allowing network in the job container environment, and letting an unknown tool do unknown things of unknown reproducibility, but at least **snapshotting the output**, and “releasing” that within a merkle package information system (such as Warpforge Catalogs).  For 1-star success, snapshotting the entire filesystem produced by the foreign tool as one indiscriminate hunk is acceptable, and having a fixed mount path requirement is tolerable.
- 2 stars — ⭐⭐ — As per 1-star, but now the mount path must support being varied freely.  (F.ex: one can snapshot a `node_modules` dir, and mount it somewhere else, and that’s probably a 2star outcome.  (As long as that dir didn’t contain any native code that’s not path-agnostic…))
- 3 stars — ⭐⭐⭐ — The Bifrost tool should be able to identify separate subtrees of filesystem for each package+version that the foreign tool has manifested, and export a snapshot of each individually, and also mechanically transform the foreign package naming to a catalog modulename and version name.  A successful 3-star tool should be able to see multiple bifrost users can see convergence in their capture results, if the foreign system is reasonably stable.
- 4 stars — ⭐⭐⭐⭐ — As per 3-star, but now we consider it a requirement that the result be deterministic.  (For some foreign ecosystems, this is trivial.  In others, post-install hooks may drastically complicate the story.  As a result, this distinction between 3-star and 4-star outcomes is useful.)
- 5 stars — ⭐⭐⭐⭐⭐ — Do all of the above without needing to unpack anything (e.g. operate purely on data transformations, and not needing to execute subprocesses and use the filesystem as a communication standard).  (5-star is more of an operational nice-to-have, rather than a critical feature that shapes outcomes.  (Maybe it belongs in a different part of the chart.))

Notice the success levels depend mostly on the capability of the bifrost tool, but higher levels also tend to start being unreachable unless the sanity of the package ecosystem they’re bridging to is over a certain level.  E.g., if a package ecosystem that a bifrost is bridging to has nondeterministic results… it’s still easy to reach a 1-star or 2-star outcome, and possibly a 3-star outcome, but higher is more or less impossible.

In some cases, bifrost tools can be aimed at an entire foreign package repository, and can scrape the whole thing at once.  In other cases, it may only really be possible to gather packages when holding some project info which references them for consumption.  (This detail isn't entered into affecting the star ratings above, because it only affects how much work it is to reach completeness of ingestion — it doesn't affect quality of outcomes otherwise.)

An additional goalpost could be re-exposing the foreign system's network APIs again too, for systems where that becomes really important to integratability.  A Bifrost project *might* find itself with most of the code necessary to do that anyway.  Still, it's a bonus, not part of the core definition of what's required to be a bifrost.