---
layout: post
title:  "Primer: facebook's HTMX from 2010"
date:   2024-02-20
categories:
---

I recently discovered this 2010 JSConf presentation via [HTMX] discord:

<iframe style="width: 100%; aspect-ratio: 16 / 9" src="https://www.youtube-nocookie.com/embed/wHlyLEPtL9o?si=TCFtMn55aK0vtAlH" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# Summary

1. At Facebook they had loads of JavaScript and that was causing slow page loads.
2. Loading the JavaScript async didn’t work well as the page would render, but be non interactive - which sucks from a UX perspective.
3. They realised that the basic operation the JavaScript was performing was making an http request and swapping the new content somewhere in the DOM - so they wrote a 40 line JavaScript function to do this which they included inline in every pages’ `<head>`[^1]. This gave them instant interactivity.
4. They were then going through incrementally replacing as much existing JavaScript with just this library - increasing performance (~5s to ~2.5s page load) and reducing complexity and lines of code.

# Discussion

The motivation was performance. The effect was better performance - but also much less code to write and a simpler overall system.

I'd recommend watching the whole thing, I found it a pleasure.  The author, [Makinde Adeagbo](https://makinde.adeagbo.com/), is a charismatic presenter.  He covers other areas like downloading JS on-demand, native HTML controls and the tooling they used to find interactions that they could apply Primer to.

The actual code can be found here: [https://gist.github.com/makinde/376039](https://gist.github.com/makinde/376039).

The context now than then is very different of course. They were already returning html from the server - so it’s less of a jump to having a generic JavaScript library to do it for you. The point of comparison now would be to a react style SPA with endpoints returning JSON and HTML being generated client-side.

## Why is this interesting?

* The core idea is the same as HTMX, which is currently riding high in the hype cycle - but 14 years ago, and applied within a megacorp. I view the current enthusiasm about HTMX as very much a reaction to SPA misery - both as a user (slow brittle websites) and as a developer (increased complexity and lines of code).  In this case the technology was developed and used at the same time, and within the same organisation as react!
* Like many YouTube videos I'm left with the question "What happened next?".  How widely did they end up applying this?  What happened to the 300 people within facebook at the time that knew about it? It can't have made that much of an impression on them at the time - they didn't disperse and evangelise.  Why did react win inside facebook?  Why, why, why?

  Speculation:

    * React allowed them to scale their features quickly by having many teams work more independently of each other?
    * Something related to the mobile web/apps? At this time it was clear that mobile was going to be the most important client, and maybe they had some ambition for offline only use cases?
    * Pure chance - the right people heard about react within facebook at the right time?
    * The hypermedia approach was too limited to create the UX that facebook wanted?

# Minutiae

Less interesting asides peripheral to the main point of this blog post:

* HTMX is 14kB gzipped.  IMO there's no point in it being any smaller - unless it could be so small it could be included directly in every `<head>` - removing the need for round-trips.  I don't know what a reasonable threshold would be here.  The default initial TCP receive buffer size is 128kB on my machine, so if it's possible to fit the TLS negotiation, HTTP headers, the `<head>` and at least some content in that size you could display your website within 3 round-trips.
* There's a difference in emphasis WRT motivation between this and HTMX: The problem Makinde set out to solve is clear - facebook is too slow. The pleasant side effect: deleted a bunch of JS. Compare with the motivations on the HTMX website.  They're more like capabilities phrased as questions than motivations, and they're certainly more philosophical:

    * Why should only `<a>` and `<form>` be able to make HTTP requests?
    * Why should only click & submit events trigger them?
    * Why should only GET & POST methods be available?
    * Why should you only be able to replace the entire screen?

  Of these the 4th one is critical - the rest are absent from Primer, and arguably relatively unimportant.  However there are critical features in HTMX that Primer doesn't implement[^2] including:

    * Replacing the whole page with working back button behaviour
    * URL bar wrangling
    * Out-of-band swaps
    * Loading indicators
    * etc.

  For "Load more comments" buttons most of the above are unnecessary, for avoiding whole page loads when changing pages they are very much necessary.
* Makinde also describes downloading JavaScript to be executed in response to user actions - rather than loading all possible javascript at page load time.  I haven't touched on that here as it's not a problem I'm interested in - but it may be relevant to you.

# References

* [Twitter: Carson Gross on primer](https://twitter.com/htmx_org/status/1753183384493297751).  There's a bunch of interest in this tweet and the replies.
* [Twitter: Dan Abramov on facebook history](https://twitter.com/dan_abramov2/status/1758121064360497192)
* [Discuss this on Hacker News](https://news.ycombinator.com/item?id=39444432)

# Footnotes:

[^1]: See also [htmz](https://leanrada.com/htmz/)
[^2]: [Hacker news comment: Carson Gross on htmx vs htmz](https://news.ycombinator.com/item?id=39431985)

[HTMX]: https://htmx.org/
