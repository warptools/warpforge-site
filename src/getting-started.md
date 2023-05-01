---
title: "Getting Started"
layout: base.njk
eleventyNavigation: 
    key: "Getting Started"
    order: 5
---

Getting Started
===============

Getting started with Warpforge may still be a little bumpy -- we're a young project.  But here's an outline of where you can start:


### Simple Quickstart

- Install warpforge!
	- For now, this means `git clone https://github.com/warptools/warpforge && cd warpforge && make install`, and requires a `go` compiler and toolchain already installed.
	- You may also need to `export PATH=$PATH:~/go/bin`, as this is the location `make install` drops the binaries.
	- We tend to symlink or alias `wf` as a shorthand for `warpforge`, too.  (If you don't do this, that's fine, but you might need to lengthen some of the commands you see in demos accordingly.)
- `warpforge -h`
	- This will show you all the subcommands, and basic help menu locally.
- `warpforge quickstart`
	- ... which will error ;)  but hopefully inform you where to go from there.
- `warpforge quickstart example.org/demo-proj`
	- that's better ;)
- And from there, all these commands should do interesting and useful things (as described in the mesages from `quickstart`):
	- `warpforge catalog update`
	- `warpforge status`
	- `warpforge run`
	- (and more)
- If you have trouble getting things to work:
	- Try `warpforge health` to run some quick self-diagnostics!
	- Come to us in some of the [Community](/community.md) channels and please ask for help!
- What's next?
	- You can edit the `plot.wf` file to do different things!
		- Edit the `"action"` fields to execute different commands in the container.
		- Add other filesystems to the container's initial filesystem by adding more `"catalog:*:*:*"` names, and the path you want them to appear at.
		- If you're in a git repo -- you can it its latest commit to the container by using `"ingest:git:.:HEAD"` as an input!
		- Add other paths to the `"outputs"` map if you want to save other parts of the filesystem at the end of the execution.
	- You can add multiple execution steps to the plot!
		- Each new entry in the `"steps"` map gets its own container.
		- Move data between them by using the `"pipe:otherstepname:outputname"` syntax to wire things together!
	- Tag data you produced and get it ready to share by using `warpforge catalog release`!
	- Create more plots and modules!
		- You can use `"catalog:*:*:*"` syntaxes to move information to them (after you've released it, using the command above).
	- Check out the rest of the docs in `warpforge -h` and this website to see what else Warpforge can do for you!



### More references

We have both demos as well as large real-world working systems you can check out for more inspiration:

#### Demos

... are over on the [Demos page](/demos.md)!

#### Real Systems

The biggest public set of packages built with Warpforge are published in something called the Warpsys Catalog.
It's both real packages containing software you can use, and a great source of reference material.

There are a couple different ways you can look at the Warpsys Catalog:

- the rendered HTML catalog website: **https://catalog.warpsys.org/**
	- This is the easiest to navigate -- click through module names to find release info, download links for content, and links to metadata.
	- In addition to content built with Warpforge, you can find *how it was built* by looking at the [replays](/glossary.md#replay).
	  For example, this is a replay showing how the warpsys Bash package is built: [catalog.warpsys.org/warpsys.org/bash/_replays/zM5K3Vgei4...Zv5TQ](http://catalog.warpsys.org/warpsys.org/bash/_replays/zM5K3Vgei44et6RzTA785sEZGwuFV75vCazjhR11RH5veFdMTx7F5cg2c4NA5HXPK8Zv5TQ.html)
- the same Catalog data, but raw, in JSON files, in Git:
	- [**github.com/warptools/catalog**](https://github.com/warptools/warpsys-catalog)
- and the source code that produced these build instructions originally:
	- [**github.com/warptools/warpsys**](https://github.com/warptools/warpsys)
