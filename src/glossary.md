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

A Catalog is a large, snapshotable, content-addressed datastructure used in Warpforge
to communicate human-readable labels for data and filesystem snapshots.

Catalogs can also store associated metadata within the same labelling system,
and one major use of this is to attach standardized rebuild instructions (called [Replays](#replay))
the data that has been inserted into and labelled in a Catalog.


#### Formula

A Formula is one of the main API objects used to do work with Warpforge.
A Formula contains a set of "inputs", an "action" section that describes some work to do, and some labeled "outputs".

In a Formula, the inputs must all be [WareIDs](#wareid) or other literals.
([Catalog](#catalog) references, or anything at all that refers to other information outside the Formula object, are not allowed!)

Formulas, though used as a part of all processing in Warpforge, are not often seen directly by users.
Typically, a [Plot](#plot) is used to generate a Formula (or series of Formulas, if the plot has multiple "steps"),
while using human-readable names that allow stitching together larger suites of related work.


#### Plot

A Plot is one of the main API objects used to do work with Warpforge.
A Plot contains a set of "inputs", some "steps" which describe work to do, and some labeled "outputs".

In a Plot, the inputs can use human-readable names from [catalog](#catalog) references,
and steps can refer to these inputs by local names,
as well as use other steps outputs as their own inputs.
Each "step" is resolved into a [Formula](#formula) when a Plot is evaluated.

In contrast to a Plot, a [Formula](#formula) is also a major API object in Warpforge
which describes doing some work... but its inputs are restricted to being [WareIDs](#wareid) and other literals.
References to catalog information are not allowed in Formulas,
making them both more concrete but simultaneously much less convenient to use directly in comparson to Plots.


#### WareID

A WareID is an input identifier string that identifies a filesystem snapshot by a hash.

Example WareIDs look like `ware:tar:4z9DCTxoKkStqXQRwtf9nimpfQQ36dbndDsAPCQgECfbXt3edanUrsVKCjE9TkX2v9`
or `ware:git:d68916538966843121b9362a6b925a151d3fbbd3`.

WareIDs are seen in lower-level Warpforge APIs (like [Formulas](#formula)),
and can also be used directly in some commands (like `warpforge ware unpack`),
but are often hidden behind a [Catalog](#catalog) lookup indirection at higher layers (such as in  [Plots](#plot)) that allow human-readable references to be used instead of large hard-to-read hashes.
