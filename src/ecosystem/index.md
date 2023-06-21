---
title: "Ecosystem"
layout: base.njk
eleventyNavigation:
    key: ecosystem
    order: 60
---

Ecosystem
=========

This section of the documentation contains information about other parts of the ecosystem around `warpforge`.



Warpforge has Limits
--------------------

Why do we have an ecosystem at all?
Because there's a lot to do -- and one project can't do it all.
`warpforge` has limits -- and we want you to be able to do more!

`warpforge` itself focuses on providing environment control and enabling any kind of work to be done.
It remains studiously unopinated about what kind of content it processes.

This gives us a great and extensible foundation, but when it's time to get work done,
there's a lot of questions that still need answering.
For example, questions about how things are compiled; how they're linked together;
and how they're made available and composed together into a working system.

Those are questions that other tools in the ecosystem -- like [`linkwarp`](./linkwarp/), for example --
step up to answer.

_Optionally._

You'll find examples of how to integrate some of these other tools and conventions with your use of Warpforge,
but it's never forced, and there's no close coupling between Warpforge and these tools and conventions.



Package Suites
--------------

Suites of packages that contain premade software and datasets are an important part of the ecosystem around Warpforge!
Although Warpforge is great at building things from source, we also want to share that work.
Not everyone needs to re-write those instructions every time!

One of the largest package suites is the [Warpsys](./warpsys/) catalog.
This catalog contains software like bash, python, gcc, and other common and essential tools --
already named and tracked in standard [catalog](/glossary.md#catalog) format,
ready to use either as direct downloads or via reference in your plots with the catalog ref syntax,
and provided complete with [replays](/glossary.md#replay).

Other [catalogs](/glossary.md#catalog) are possible too!
If you'd like to maintain another catalog and have it listed here, we'll be happy to link to it.
(Join us in the [community](/community.md) channels to discuss this!)

