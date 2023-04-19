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

To see more complex examples of Warpforge usage, check out:

- the [Demos page](/demos.md)!
- the Warpsys Catalog!
	- as source modules in a workspace: 
[**github.com/warptools/warpsys**](https://github.com/warptools/warpsys)
	- as released catalog info, in JSON in git: 
[**github.com/warptools/catalog**](https://github.com/warptools/catalog)
	- as released catalog info, in HTML-presented form: [catalog.warpsys.org](http://catalog.warpsys.org)
- the [examples directory](https://github.com/warptools/warpforge/tree/master/examples) in the project source repo!
