---
layout: post
title:  "Neil Brown on the UNIX philosophy"
date:   2021-12-16
categories:
---

From [an LWN comment](https://lwn.net/Articles/576078/) by [Neil Brown](https://blog.neil.brown.name), kernal hacker and author of the [Ghosts][1] [of][2] [Unix][3] [Past][4] LWN article series (among others):

> One of the big weaknesses of the "do one job and do it well" approach is that those individual tools didn't really combine very well. sort, join, cut, paste, cat, grep, comm etc make a nice set of tools for simple text-database work, but they all have slightly different ways of identifying and selecting fields and sort orders etc. You can sort-of stick them together with pipes and shell scripts, but it is rather messy and always error prone.
>
> I remember being severly disillusioned by this in my early days. I read some article that explained how a "spell" program can be written to report the spelling errors in a file. It uses 'tr' to split into words, then "sort" and "uniq" to get a word list, then "comm" to find the differences. "cool" I thought. Then I looked at the actual "spell" program on my university's Unix installation. It used a special 'dcomm' (or something like that) which knew about "dictionary ordering" (Which ignores case - sometimes). Suddenly the whole illusion came shattering down. Lots of separate tools only do 90% of the work. To do really complete work, you need real purpose-built tools. "do one thing and do it well" is good for prototypes, not for final products.
>
> One thing Unix never gave us was a clear big picture. It was always lots of bits that could mostly be stuck together to mostly work. I spent a good many years as a Unix sysadmin at a University and I got to see a lot of the rough edges and paper over some of them.

I read this when it was first published and it had quite an effect on my thinking.

[1]: https://lwn.net/Articles/411845/
[2]: https://lwn.net/Articles/412131/
[3]: https://lwn.net/Articles/414618/
[4]: https://lwn.net/Articles/416494/
