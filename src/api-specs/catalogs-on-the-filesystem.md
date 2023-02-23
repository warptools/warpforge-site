---
title: "Catalogs on the Filesystem"
layout: base.njk
eleventyNavigation: 
    key: Catalogs on the Filesystem
    parent: API Specs
    order: 50
---

The filesystem layout is a ***convention***.

The essential form of a Catalog is just a series of IPLD objects.  We project them into a filesystem layout based on some of their properties, for convenience and ease-of-splunking.

Everything is also linked by hash, and the filesystem paths shown here have nothing to do with that merkle tree.  (That a git merkle tree can be computed over the filesystem projection is true, but generally considered incidental.)

## Filesystem Outline

Example catalog filesystem outline:

```
{catalogroot}/{moduleName}/_module.json
{catalogroot}/{moduleName}/_releases/{releasename}.json
{catalogroot}/{moduleName}/_mirrors.json
{catalogroot}/{moduleName}/_replays/{HASH}
```

- `_module.json` contains the `CataloguedModule` object — which contains the module name (remember, the filesystem path is a projection only, so what’s in the file is the actual canonical info), and a list of tuples of releaseName and CID of release data.
- The various `_releases/*.json` files contain one `ReleaseInfo` object each.  (Note that the `CataloguedModule` object linked to these by CID — but yes, the filesystem projection uses the releaseName rather than the CID.  It’s a concession to human convenience.  Warpforge will still check that the CID link is correct!)
- The `_mirrors.json` file contains... [well, you know](/api-specs/catalogs).
- The `_replays` directory optionally contains replay documents.  If present, they should've been referenced by metadata in the release objects.  These are in files named by CID.
    - This is actually dogfooding our own extension system!  Other directories like this could exist as well!

### Why like this?

- Q: Why is the module name a directory path?
    - A: Because that’s handy and people like it.
- Q: Why the underscore prefix?
    - A: Because it helps avoid filepath collisions for most use-cases. This means we don’t allow you to start module names or catalog paths with `_`.
- Q: Why `.json` extensions rather than `.wf`?
    - A: Elsewhere, warpforge uses `.wf` for filenames that are major landmarks (ex: `module.wf`).  But these files in a catalog... they aren’t landmarked by the filename, so that reason doesn’t apply here.  (We already know what they are because of other landmark filenames that highlighted the directories above these files; namely, `./.warpforge/_catalog{/,s/*/}`)  So, shrug; but we had no reason *not* to go with a json extension.
        - ... really, this one could’ve gone either way, easily.
- Q: Why are releases getting a separate document each?
    - A: Because they could get relatively large — especially considering that there aren't tight size limits on the metadata map entries.
- Q: Why are the files for each `ReleaseEntry` object worthy of human readable names?
    - A: People tend to want to splunk these, so we'll help.  (That’s true of a lot of things, but it’s *very* true of this data, *very* frequently, so we decided to make a special effort on this one.)
- Q: Why is the mirrors file just one file (rather than one per release, or such)?
    - A: Because often, the mirrors file should contain literally like one entry: a content-addressed warehouse URL!  (A single content-addressed warehouse can easily contain every ware mentioned, for every version released; no problem.  And we hope that this is typical!)
- Q: The `_module.json` file is a document containing tuples of both release *names* and also the *cid?*  What?  Why?
    - A: Yes, `list[struct{name, cid}]` is necessary, because remember, the canonical form of this data doesn't count the filesystem info; the filesystem info is a projection: therefore, we needed links here; those are the canonical form of the data.  Also: we need a list because it needs to be ordered!
- Q: Why are replay files *not* using human readable names?  (Especially considering release files got them?)
    - A: It didn’t seem worth it.  People want to splunk replay files manually rather less often than release files.  (Usually by the time you’re looking at this data, you’re about to do something more complex than bash globbing for other reasons anyway.)  And if we tried to project a useful name... the mangle function is getting significant.  We could’ve put in the extra special effort to put human-readable names on replay files; just seemed both higher cost and lower reward than it did for the release info files; poor ROI in other words.  So we didn’t.
    - A: Also, we implemented the replay system using our own extension mechanism...!  And we wanted to have that extension mechanism use directories full of content-addressed objects, rather than opening up the complexity of a name-mangling callback function.  So, this is the result.

### Can you author this filesystem manually?

Kinda, but not really.  You'll need some tool assistance to do so.

If you write a release json by hand, you'd need to invoke a small tool to compute its CID and add it to the `_module.json`.  You probably can't do that by hand easily.

Similarly, we should have a catalog linting tool that makes sure there aren't extra `_releases/*.json` files floating around that aren't actually referenced, and similarly for replays, etc.