---
title: "Contributor's Guide"
layout: base.njk
eleventyNavigation: 
    key: Contributor's Guide
    order: 30
---
## Where to discuss

You’ve probably already seen the [Community](https://www.notion.so/Community-676332742afa4276be571f7d035d55db) page.  Links to the chat rooms and the code repos are there!

You can use github issues for discussions, or, just come hang out in the chat — or, both!

(It’s often a good idea to nudge in the chat if an issue or PR doesn’t get attention.  Notification firehoses being what they are, sometimes things just get lost otherwise :))

## How to Push

Github has the neat feature of supporting permissionless contributions through forking!

1. Fork the repo.
2. Push your contributions there.
3. Open a PR back to the upstream repo.
4. We can review and, if it’s good, merge it into the upstream repo.

If you’re newer to Git or Github: See the [Git and Github pro-tips](https://www.notion.so/Git-and-Github-pro-tips-c876f2ad73e0491d87e48b7c31a62812) section for some tips about how exactly to go about it.

If you’re a core contributor (or if you just ask very nicely): you may also be able to push directly to the repos you’re working on.  If so: nothing’s stopping you from using the above strategy anyway ;) but regardless of how you do it, *do* be very careful about the main/master branches.  You should still use PRs to manage changes!

## How Merging is Decided

We have a low-formality structure right now.  It’s early days; the minimum of bureaucracy we can get away with and still function smoothly, is what we’re going to do :)

When ready to push code:

- Open a PR to the repo.
- *Tell people about it in the chat,* if you want a fast review.
- *Nudge people about it in the chat,* if you’re still not getting attention.
- If the tests pass, and it gets approving reviews, we can probably merge it!

If you’re a core contributor pushing code:

- It’s mostly the same thing!
- It’s still good practice to open a PR for your changes, and solicit review.
- If this is low-impact code, and you’re really sure, or there’s some informal discussion in chat and this all seems fine, you may be able to merge without formal review.  (But if you make a mess, it’s yours to clean up, too.)
- At the end, either you or your reviewer can hit the “merge” button.  (Discuss amongst yourselves what makes you most comfortable. :)  It never hurts to say “I’ll merge this after review” or “feel free to merge if green”, just to make it explicit!)

As a core contributor who’s reviewing code:

- *Look immediately* when the PR is made.  Just briefly.
    - Is it a new contributor?  *You have to click a button before CI even runs*, so, check for safety and then do that!
- Leave comments about what you reviewed!
    - Including the parts that look good!
    - (Feel free to comment about what you skimmed, too — it’s truth that this happens; be honest.  Truth helps everyone know where the bar is set.)

## Tips and Tricks

### Git and Github pro-tips

#### For working in a fork:

Usually, your fork repo is called “origin”.  It’s useful to add an alias for “upstream” that points at the upstream:

- `git remote add upstream [https://github.com/warpfork/warpforge](https://github.com/warpfork/warpforge)`

Now you can do other things like `git fetch upstream` or `git checkout master && git pull upstream/master` to quickly get content from the main upstream repo.

#### Checking out other people’s PRs locally:

Try this:

- `git fetch upstream pull/17/head && git checkout FETCH_HEAD`
    - Replace “17” with the PR number you’re interested in
    - Look up a bit if “upstream” isn’t defined :P