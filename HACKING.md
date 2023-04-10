HACKING
=======

This file contains advise for how to modify, work within, and contribute to this repo.



Styleguide
----------

### page titles

- Use the `title` property in the frontmatter to set the title that will appear in the page's `<title>` element.
- Put the title of the page, as you want it to be rendered in that page, in an h1 element in the body.
- Optionally, use the `eleventyNavigation.key` property in the frontmatter to override the link name as used in the nav tree.

All three of these can be the same; or, it's acceptable if they differ slightly to keep things fitting on the parts of the screen where they are rendered.

The title should usually be unique (so someone with lots of browser tabs open can tell what's what),
but the `eleventyNavigation.key` can be set to something shorter because it's going to be rendered in context of a nav tree with other words around it.
And the h1 element in the page just has more screen realestate than either of the others to work with, so can be longer, if needed.

For a concrete example: the FAQ page uses "FAQ" as the title and navtitle, but contains "Frequently Asked Questions" as the h1 element within the page itself.

### heading styles

It's preferable to use the underline-style for h1 and h2 in markdown.
They're more visually distinctive.

The h1 element should only be used once.

### heading styles in lists

If making a list of sections (e.g., enumerating all the plugins in a subsystem, or all the projects with comparable features, etc),
consider using an h4 element for members of the list.

(This tends to be the depth of heading anyway, naturally, on most pages.)

### line spacing

Three linebreaks before a new h2 are good style.
This spacing makes it easier to eyeball through the raw markdown quickly.

Two linebreaks before a new h3 may be wise.
Follow the style in the surrounding area if in doubt.

Files should always end in a linebreak.
(Git should nudge you about this, as will most editors.)
