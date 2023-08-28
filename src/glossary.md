---
title: "Glossary"
layout: base.njk
eleventyNavigation:
    key: Glossary
    order: 99
---

Glossary
========

{# (This is a nunjucks comment!) #}
{#
Styleguide:

- Each definition is an h4.
	- Alphasort the headings!
	- If something has aliases, give each one an h4, and give the aliases one sentence and a link to the main entry.
- The first sentence should always be "A {{Foo}} is a {{...}}"
	- No sentence fragments!
- In this page: Keep things as terse as possible.
	- Introduce only.  Crosslink heavily.
	- For anything that's not a key concept, two sentences should be plenty.
	- Even for a key concept: more than three paragraphs would be too many.
	- As much as possible: Make another page and link to it!
#}


#### Catalog

A Catalog is the serializable API used by Warpforge
to associate human-readable names to filesystem snapshot hashes.

Catalogs are themselves snapshotable, content-addressed data structures -- just like the filesystem snapshots they label.

Catalogs have an extensible schema and are also meant to store arbitrary associated metadata along with the primary naming information.
Major uses of this include both simple metadata -- like "author" --
and also is the basis of more complex and powerful features, like Warpforge's storage of standardized rebuild instructions (called [Replays](#replay)).

[See the main articles for Catalogs](/catalogs/)!



#### Formula

A Formula is one of the main API objects in Warpforge, and it describes some commands to be invoked within an environment.

A Formula contains:
1.  a set of "inputs" (filesystems, and where to mount them),
2. an "action" section that describes some work to do,
3. and filesystem paths to collect, save a snapshot of, and report as "outputs" after the work is done.

Formulas look similar to [Plots](#plot), because both describe actions to be taken, but there are some major differences:

- In a formula, all inputs must all be [WareIDs](#wareid) or other literals.  [Catalog](#catalog) references, or anything at all that refers to other information outside the Formula object, are not allowed!  But such references *are* allowed in [Plots](#plot).
- In a formula, there's one environment setup and one action described.  In practical terms, a formula describes one container execution.  By contrast, in a [Plot](#plot), there can be multiple "steps", and each "step" is resolved into a Formula.  (Plot steps can refer to each other and use each others outputs as inputs; a formula can't do this because that reference wouldn't be sufficiently literal.)

One of the virtues of a formula is that it can be hashed and content-addressed --
and this is a perfect key to use for [memoization](https://en.wikipedia.org/wiki/Memoization),
since it's a total description of a static environment.
([Plots](#plot) are not similarly suitable, because they allow non-static references!)

Formulas, though used as a part of all processing in Warpforge, are not often seen directly by users.
Typically, a [Plot](#plot) is used to generate a Formula (or series of Formulas, if the plot has multiple "steps"),
while using human-readable names that allow stitching together larger suites of related work.

[See the main articles about Formulas in the Warpforge API docs](/warpforge/api/formulas-and-runrecords/)!



#### Ingest

An ingest directive is one of the kinds of input strategy that can be used in a [Plot](#plot).
It directs Warpforge to gather a snapshot of some host filesystem state, then use that snapshot as the input.

Example ingest directives look like `ingest:git:.:HEAD` or `ingest:tar:./path`.

Ingests are a non-static kind of input, and so cannot be used in a [Formula](#formula) (although it can be used in a [Plot](#plot)).
Ingests are also not valid in a [Replay](#replay), despite being valid in a [Plot](#plot) -- because ingests are all about bringing unmanaged data into the grasp of Warpforge, they don't make make much sense to attempt to replay.

The main practical uses of ingest directives is easy integration with development environments.
Ingests are not useful during release engineering.



#### Plot

A Plot is one of the main API objects used to do work with Warpforge, and describes a series of steps to be evaluated, each of which does some isolated piece of work, and may be described as using other work as inputs.

Plots are very similar to [Formulas](#formula), except:

- each Plot contains several "steps" (each of which becomes one [Formula](#formula) when resolved),
- and a Plot is allowed to use non-static references as inputs -- like refering to output of another step of the plot as input for a subsequent one, or refering to content by a human-readable name (which has to be resolved by a [Catalog](#catalog] lookup), or even using an [ingest directive](#ingest).

Plots are thus very much more "powerful" than [Formulas](#formula), and as a user you're more likely to use the Plot API for most purposes.
However, a Formula is more independent than a Plot, since resolving a Plot may require additional context (like a catalog, or some piece of host filesystem for an ingest, etc).


[See the main articles about Plots in the Warpforge API docs](/warpforge/api/modules-plots-and-replays/)!



#### WareID

A WareID is an input identifier string that identifies a filesystem snapshot by a hash.

Example WareIDs look like `ware:tar:4z9DCTxoKkStqXQRwtf9nimpfQQ36dbndDsAPCQgECfbXt3edanUrsVKCjE9TkX2v9`
or `ware:git:d68916538966843121b9362a6b925a151d3fbbd3`.

WareIDs are seen in lower-level Warpforge APIs (like [Formulas](#formula)),
frequently appear in logs,
and can also be used directly in some commands (like `warpforge ware unpack`).

In higher level APIs (such as in [Plots](#plot)), WareIDs are often not seen directly,
but are instead indirected behind a [Catalog](#catalog) lookup,
which allow human-readable references to be used instead of large hard-to-read hashes.

[See the main articles about Wares and WareIDs in the Warpforge API docs](/warpforge/api/wares-inputs-outputs/)!
