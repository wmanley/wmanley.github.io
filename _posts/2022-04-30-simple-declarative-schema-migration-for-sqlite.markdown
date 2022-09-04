---
layout: post
title:  "Simple declarative schema migration for SQLite"
date:   2022-04-30
categories:
---

See my colleague [David Röthlisberger]'s website for an article we authored together:
[Simple declarative schema migration for SQLite](https://david.rothlis.net/declarative-schema-migration-for-sqlite/).

The TL;DR: is:

* We store our SQL schema in git and on application startup we modify the existing database so the schema matches what the application was expecting.
* In doing so we don't need to write any specific migration code.
* The types of migration we can perform automatically are limited, but in practice these limits are not particularly onerous.
* If we do need to explicitly step outside these limits (e.g. renaming a column) we can, and the auto-migration system will still help with the parts of a migration that it understands.
* This approach works really well with a branching/CI workflow where multiple versions of the database schema may exist in parallel.

[David Röthlisberger]: https://david.rothlis.net/
