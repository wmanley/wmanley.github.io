---
layout: post
title:  "Seen on HN: Invert shell and terminal"
date:   2021-03-31
categories:
---

Here's a [comment from Ericson2314](https://news.ycombinator.com/item?id=26617656)
that I agree with wholeheartedly:

> Invert the shell and terminal: every shell command (with unredirected streams)
> gets it's own pty.

Read the whole comment for more.

I think this could work well - but unlike many projects intended to improve the
terminal/shell experience it could be implemented in a backward compatible way -
and as such have a chance of being widely adopted.

You'd need to implement a new protocol between terminal and shell.  Maybe the
shell would be in charge - allocating PTYs and multiplexing the output back to
the terminal.  Or maybe the terminal would be in charge where it would allocate
PTYs for children and send them to the shell using FD passing.  There are
advantages both ways.

Integral to its success though would be getting the support for this new
protocol **upstream in bash**.  Bash is the most widely deployed shell and the
only way to get to the point where you can log in to a new machine (or over SSH)
and expect this to "just work".  Without that it would remain a niche and could
join the graveyard of other improved shells/terminals that approximately no-one
uses.

As such the protocol would have to be minimal and easily implementable in C.
Systemd's protocols could be an inspiration.  I've implemented both sides of
`sd_notify` and `LISTEN_FDS` before, in more than one programming language and
it was very straightforward.
