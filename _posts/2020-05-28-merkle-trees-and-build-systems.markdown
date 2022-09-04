---
layout: post
title:  "Merkle trees and build systems"
date:   2020-05-28
categories:
---

An article written by my colleague [David Röthlisberger] in part describing how the build system that we built at stb-tester works and the philosophy behind it:
[Merkle trees and build systems](https://lwn.net/Articles/821367/).

> In traditional build tools like Make, targets and dependencies are always *files*. Imagine if you could specify an entire *tree* (directory) as a dependency: You could exhaustively specify a "build root" filesystem containing the toolchain used for building some target as a dependency of that target. Similarly, a rule that creates that build root would have the tree as its *target*. Using [Merkle trees](https://en.wikipedia.org/wiki/Merkle_tree) as first-class citizens in a build system gives great flexibility and many optimization opportunities. In this article I'll explore this idea using [OSTree](https://ostree.readthedocs.io/), [Ninja](https://ninja-build.org/), and [Python](https://www.python.org/).

[David Röthlisberger]: https://david.rothlis.net/
