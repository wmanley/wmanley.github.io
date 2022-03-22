---
layout: post
title:  "Unlock software freedom one by using better tools"
date:   2020-05-25
categories:
---
An idea that I've had burrowing into my mind for quite a few years now is that free software distros are held back by the crap build tools that are used to build software - but the lack of availability of better tools isn’t the cause of the problem.

[Software freedom one is][one]:

> The freedom to study how the program works, and change it so it does your computing as you wish (freedom 1). Access to the source code is a precondition for this.

In practice if you want to make a modification to your system it's a massive ball-ache.  In my opinion just getting to the point where you can find the relevant package and build it is harder than making the change in most cases.  I think it's a real shame that we have these distros full of software with source available, but you'd never think of making small changes because there's such a hurdle to get over, and that hurdle isn't knowing programming, or understanding the code - it's just the inconvenience of building and running your modified version.

Back in 2006 (I think) there was the project called [One Laptop Per Child][olpc].  One of the ideas behind it was that not only was this a tool to enable children in the developing world to do things like wikipedia access, word processing, and spreadsheets, but that it was a tool that they could mould themselves.  They could change any part of the software that made up the device, safely, in such a way that they couldn't brick the device and any changes were easily revertible.  The thing that really captured my imagination was a button on the keyboard labelled "View Source"!  Imagine that!  You're using a piece of software and you want to know how it works, or you want it to work differently and you press the button, and there it is in the editor.  You could make a change and build and run it in one click. Perhaps with another click you could share your changes with the world.

Blam! Suddenly the [four freedoms] aren't some abstract idea the advantages of which are reserved for professional software engineers, but they're available in practice to a much larger audience of interested amateurs.  In particular, freedom 1, the freedom that is least convenient to exercise now:  "The freedom to study how the program works, and change it so it does your computing as you wish" was previously mostly academic to all but the tiniest fraction of users of the software, but is now much broader.

Included in this group of interested amateurs are children.  I can imagine myself as a child pressing that button just to see what happens? what does it look like? What happens if I...?  Suddenly it becomes play rather than study and discipline and work.

I don't know how (or if) the view source button ever worked, but I find the idea behind it is exciting.

We can see what children (and adults) are capable of when given the right environment in their amazing Minecraft creations.  It doesn't matter that the environment is constrained, it just matters that the initial barrier to entry is low enough.  See also the boot to BASIC BBC micro.  You turn the machine on and there you are, ready to start.  Along with similar machines it spawned an entire industry of software engineers.

----

So if the main blocks to this are getting the source and then building the software, what is the solution?  I think Google has built (but more importantly - uses) tools that solve both these problems.

Internally Google[^1] have a build system called Blaze (see also its open-source cousin Basel). It's fast at building software at staggering scale.  Scale larger than any Linux distro.  How is it capable of doing this?  All build steps are reproducible - this in-turn makes them cacheable.  Secondly the tool has a complete view of the entire build-dependency graph, right back to each source-file checked into the repo. What does this mean? It means that when you change a source file, be it a C file or a header or some code-generation bit, all the dependents are built, but no more.  It means that they can be all built in parallel across a fleet of machines, so you get your results promptly.  It means that if a generated file would not be affected by that change it is not built.

Contrast this with a typical Linux distro, binary or source based.  The concept of packages cuts through the build graph at a level of fixed scale.  We define dependencies between packages, and within any particular package there is a build system that defines the finer grained dependencies. This `.so`, depends on this object file that depends on this source file.  Change the source of a man page in openoffice?  You rebuild the whole package including all the source files and running big fat linking steps.  Add a comment to a source file in a library?  Do the dependent packages need to be rebuilt?  That's a decision that needs to be taken manually every time.

With traditional binary distros this is made kind-of tractable by dynamic linking - allowing you to delay the final steps of constructing your system to run-time.  I think dynamic linking makes sense where runtime selection of code is genuinely needed, and IPC has too much overhead.  Examples include using the right GL library for your hardware, or adding new [GStreamer] elements for decoding some new video format.  It's also not so onerous for cases where the ABI is small, clear and the library has few dependencies of its own.  It can be necessary for proprietary software, where rebuilding to fix an issue in a dependency isn't possible . But in a lot of cases it's there just to work around the way that software is packaged, delaying linking to run-time because rebuilding is just too onerous.

Google uses open-source software packages internally.  How do they deal with the fact that OSS comes as packages with their own build systems?  How do they cope with not having global visibility of the build graph? The answer is that they don't. If you want to use an open-source dependency in your google project you need to convert the build system to Blaze.  This only has to happen once for any project.  Popular open-source libraries will already have been converted, so chances are that you can just reuse that effort.  By doing so the huge proprietary megacorp gains more of the benefits of the fact that the software is free (as in freedom) than the free-software distros can!

----

That covers building the software, what about getting the source?  Within google every developer effectively has the entire Google codebase checked out on their machine, as a FUSE filesystem.  It's called ["Client in the Cloud"][1].  So you want the source locally?  It's already there. Imagine this applied to Debian.  You want to make a change to openoffice, but it's huge and you don't have space for the whole thing?  It's ok, it's already there. Microsoft are developing a system for git that can handle repos of the scale of the whole of Debian. It's open-source, but currently Windows only.  See ["VFS for Git"][2].

----

What would such a distro look like?  I’ve seen one example of this from a few years ago [gittup].  It consists of a git repo containing a submodule for every package.  Each package has been modified to build using the tup build system.  They can now make a change to any file and run a build and they get a new system in seconds. Check out the [website](http://gittup.org/gittup/), particularly the section “What gittup.org does that nobody else can”. It gives you a taste for what might be possible. The tools used in gittup wouldn’t scale to the size of Debian, on the other hand we know tools that will, or could.  But it’s not about the tools…

The challenge is the sheer amount of software to be packaged is tremendous.  Having a complete view of the build graph requires converting every package to the same build system. This requires many people, each with some level of expertise in their package, all pulling in the same direction, all choosing the same tooling. It’s a social problem, rather than a technical one.

As Antoine de Saint-Exupéry said:

> “If you want to build a ship, don’t drum up the men to gather wood, divide the work, and give orders. Instead, teach them to yearn for the vast and endless sea.”

So this is my attempt to generate that yearning.

[1]: https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext
[2]: https://vfsforgit.org/
[one]: https://www.gnu.org/philosophy/free-sw.en.html
[gittup]: https://github.com/gittup/gittup
[GStreamer]: https://gstreamer.freedesktop.org/
[olpc]: http://one.laptop.org/
[four freedoms]: https://www.gnu.org/philosophy/free-sw.html


[^1]: Note: I've never worked for Google, this is just my understanding from information google has published and talking to googlers and ex-googlers. Ultimately whether what I say about how things work at google is true or not is irrelevant to the substance of this article.  This is about how the free software ecosystem and distros in particular could be different, and better.  Google is a strawman in this instance I've used to make the ideas more concrete.

