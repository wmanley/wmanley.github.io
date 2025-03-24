---
layout: post
title:  "On Test-Driving HTML Templates"
date:   2025-03-24
categories:
---

I recently came across [this article by Matteo Vaccari of Thoughtworks](https://martinfowler.com/articles/tdd-html-templates.html) via [HTMX] discord.  As part of developing Stb-tester's web interface we've increasingly been moving to a traditional hypermedia approach[^1].  This has meant a different (and I think superior) approach to unit testing than with SPAs.

Some thoughts on the article based on my experience over the last few years unit testing HTML based systems:

1. I definitely agree that testing the HTML output of your system is valuable.

2. Doing the setup by injecting a specifically setup data structure into a template seems too narrow a test - you're only testing the template in this case, but maybe in production you've messed up populating this data structure from the database. Ultimately the properties you're testing in this case aren't that useful from a system/end-user perspective.

   Instead we generally will set up the database in some specific state (typically using the code that will do this in production), and then generate the HTML from that.  This works well in terms of churn - you have to be judicious when changing the database schema, so the test is likely to remain valid while you refactor the innards of your implementation, and it's likely to be testing more of the real code-paths.

   This approach particularly valuable in unit tests because it's trivial to apply coverage tools (or even coverage based fuzzing) to improve confidence in the tests

   I still consider this unit testing, not integration testing, because it's still deterministic, single threaded and not time-dependent (no sleeps, no waiting).

3. Using CSS selectors to check specific properties of the HTML is good because it documents exactly what the properties the test is actually trying to assert. It is tedious to write though, requires maintenance in the presence of changes to the markup that wouldn't actually impact the user, and maybe you'll miss asserting something that's actually important.

   Instead we use characterisation testing. We save the generated HTML* to disk when running the test with `$REGENERATE_TEST_DATA=1` and when running the test later we check that it hasn't changed. This makes the tests easy to update when making changes. It also means that when reviewing a change to the code we can also see the *result* of that change in the git diff

4. The article also touches on characterisation testing under the heading "Bonus level: Stringly asserted". In our case we are also transforming the HTML before saving - by converting it to markdown using pandoc. This preserves more of the details of the HTML, without having to modify it specifically for the tests (`data-test-icon=`, etc).

   There's a lot more that could be said on the subject of characterisation tests, but I'll stop now.

5. The article describes characterisation testing ("stringly asserted") as an alternative to using CSS selectors - but I believe they are complimentary.  You can select an element with a CSS selector, but still render the whole element to text for testing.

6. I'm somewhat sceptical of using a real browser in a unit test.  I've found keeping browser integration tests working difficult - particularly making so the browser in CI works exactly the same as the one on the dev's PC.

   Secondly - and maybe this is just a matter of semantics - but in my mind a unit test should be deterministic, and probably single threaded and synchronous - such that exceptions bubble up from where they were raised, and assertions like "called_once" can be applied.  Using a browser messes with this.  Maybe it would still be possible to keep these properties by mocking out all network calls, and calling back into the test instead.  Could be complicated, but potentially powerful.

   With a hypermedia approach you can get pretty far without involving a browser.  To simulate a click on an `<a href>...` tag you can click by CSS selector, but then look up the href and get that from your backend.  Similarly for the HTMX attributes - as long as they're kept simple enough.

[HTMX]: https://htmx.org/

## Footnotes

[^1]: Traditional Hypermedia to me this means:
    1. Use built-in HTML functionality where possible (forms, links, etc.).
    2. HTML over the wire, not JSON
    3. JavaScript for enhancement - typically with effects scoped to HTML elements explicitly declared in markup, and preferably without dependencies.
