---
title: "Ecosystem Conventions"
layout: base.njk
eleventyNavigation: 
    key: Ecosystem Conventions
    order: 70
---
## Golden Rule(s)

- **There is not one monolithic ecosystem.**
- ... There is generally one highly recommended set of foundational choices in the ecosystem, though.  We sometimes call this “warpsys”.
- **warpforge is never dependent on anything in warpsys.**  Warpforge is a very nice toolchain for building warpsys; and warpsys is often designed to have mechanical sympathy with warpforge (e.g. nice package path conventions that are easy to mount atomically, etc); but neither is dependent on the other.
- We *absolutely* aim that **warpsys packages should be easy to install and compose anywhere, in any system**.

## Goals

Alright — how do we get packages that can install and compose anywhere, and make users happy, while generating the least fuss possible?

- Build stuff to “play nice”.
    - Okay.  That’s vague.  But it’s still a goal :)
- Build stuff to work immediately when it’s unpacked (not require “post-install hooks”).
    - Post-install hooks means one can’t easily ship read-only systems, and that’s lame.
- Build stuff to be path-agnostic.
    - See: [Goal: Path-agnosticism](https://www.notion.so/Goal-Path-agnosticism-1afbca83896d4ef3bff36c9b1344ee89)
- Build stuff to be co-installable.
    - See: [Goal: Co-installability](https://www.notion.so/Goal-Co-installability-b13a81f48bd94c56a09153770af6d28b)

None of these goals should be hard.  But!  The defaults in a lot of software build toolchains fight pretty hard to get developers to do the wrong things :(  So, we have to build up quite a list of recommendations on how to make things right.

## Our Recommendations

[Goal: Path-agnosticism](https://www.notion.so/Goal-Path-agnosticism-1afbca83896d4ef3bff36c9b1344ee89)

[Goal: Co-installability](https://www.notion.so/Goal-Co-installability-b13a81f48bd94c56a09153770af6d28b)

[Convention: Module, Release, and Content Naming](https://www.notion.so/Convention-Module-Release-and-Content-Naming-fa5600944182421fab2764b84bd54bc1)

[Convention: Package File Layout](https://www.notion.so/Convention-Package-File-Layout-36551029a2aa47dfb47f187fb89d73ce)

[Convention: Typical System Directories](https://www.notion.so/Convention-Typical-System-Directories-231cdec003f74595b709a608b9ae5ad1)

[Wrangling Dynamic Library Linking](https://www.notion.so/Wrangling-Dynamic-Library-Linking-68d36a19f1614785b0d9ebcda6623889)

[Wrangling Executable Collections](https://www.notion.so/Wrangling-Executable-Collections-e9e1844bcfc44d528eed09107d2ebadc)

[Wrangling Shebangs](https://www.notion.so/Wrangling-Shebangs-7629959c30464e4a8b6a8294fe0d95f8)