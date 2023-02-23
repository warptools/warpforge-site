---
title: "Output Discipline"
layout: base.njk
eleventyNavigation: 
    key: Output Discipline
    parent: API Specs
    order: 60
---
This page is about "when to use stdout, and when you use stderr".

It's important to lay down some principles for this in order to build tools with a consistent experience.

## Three Questions

- Is the output going to be piped?  (E.g. is this for machine reading or human eyes?)
    - Assume yes.  Therefore: things that are good to pipe — ideally just a JSON object — should go on stdout.  And just about everything else — logs, etc — should go on stderr.
- Is the expected pipe recipient a simple script, or a whole apparatus?
    - When the expected recipient is simple scripting, we should emit one JSON object on stdout, and everything else on stderr.
    - When the expected pipe recipient is a whole apparatus, a stream of JSON lines on stdout is best — because this leaves us stderr to be only for panics that crash the process entirely (and that's nice to have separated if it's a tools-using-tools situation).  **This mode should generally be engaged by a flag;** there's never any need for it to be the default.
- Is the tool being used in build mode or in chaperone mode?
    - If the tool's purpose is build mode — which is the the norm — it should focus on putting the build information (e.g. RunRecords or what-have-you) on stdout.
    - If the tool is being used in chaperone mode — e.g., it's being used to run some contained process and we want to pipe its output — then the tables are turned, and any build information (e.g. RunRecords or what-have-you) should actually go on stderr.
        - ... and/or there should be a flag for where to route that info, so one can set it to e.g. `--records=/dev/fd/3`.

## Examples

- `warpforge run <formula>` prints the RunRecord json on stdout, and everything else — all system logs and status messages, and all the (wrapped and decorated!) container output — to stderr.  (It's meant to be pipeable, and such that you could pluck results out of the RunRecord with something like `jq`.)
- `warpforge run <module>` is roughly the same, but printing an object of info about the module run to stdout at the end.
- `warpforge run * --apimode`should override the above behaviors, and emit **all** data on stdout as JSON lines.