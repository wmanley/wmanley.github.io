---
layout: post
title:  "pip and cargo are not the same"
date:   2022-02-23
categories:
---
I often see Rust's cargo package manager dismissed in online discussions by
analogy to pip and npm.  Cargo/rust doesn't suffer from many of the problems
that pip does.  Some of these reasons are not just about cargo, they're also
about how Rust is different to Python and about how Cargo and Rust integrate:

1.  There are only venvs with Cargo, so you can't install crates into a location
    where it will interfere with other unrelated rust programs.
2.  Rust programs are "almost statically linked".  Typically they only dynamically
    link against libc, and possibly a few more common libraries like openssl.
3.  You don't need to worry about reproducing the build environment on the host
    that will be running the executable.  Usually just copying the executable
    over is sufficient (see (2)).
4.  There is a 1 to 1 relationship between cargo crate names and what gets `use`d
    in the rust file. With pip your pypi package `moo` can include Python package
    `foo`, or whatever else it likes.
5.  Similarly you can't `use` something in your rust code that you haven't asked
    for in your `Cargo.toml` - transitive dependencies of your dependencies
    (mostly) don't affect you.
6.  Rust provides many tools for managing privacy, so the public interface of a
    package is explicit. This makes it much harder to accidentally depend on
    something the crate author considers an implementation detail, making a cargo
    update much less likely to break your application than the Python equivalent.
7.  There is a culture of taking compatibility seriously in the rust ecosystem.
    Crates are expected to maintain API compatibility for major versions and
    [Cargo requires the same][semver].
8.  Cargo requires lockfiles, and can generate a lockfile just based on the
    `Cargo.toml` and the crates.io index.  Pip has `pip freeze`, but that just
    captures the packages you've got installed.  So it requires that the packages
    be installed in the first place, and won't include packages that are required
    on other OSs for example.  [Pipenv] helps here.
9.  Rust packages tend to be more self-contained than Python ones.  Often Python
    packages will be bindings to existing libraries, while Rust ones will be pure
    rust.  This means that you run into issues of missing system dependencies far
    less often with Cargo than Pip.
10. Rust maintains much better backwards compatibility than Python.  Upgrading
    to a newer version of Rust for a dependency is very unlikely to break your
    build - and if it will break anything it will break at build time, rather
    than run time.  Upgrading Python often causes your code or dependencies to break
    and requires you upgrade Python on your deployment target as well.  Rust
    doesn't even need to be installed on your deployment target.
11. A rust executable can include different versions of the same crate in the
    same rust executable - so you don't need your transitive dependencies to all
    agree on the same version if there are compatibility issues.

I don't have any first hand experience with npm, but I believe at least some of
the above will apply there too.

[semver]: https://doc.rust-lang.org/cargo/reference/semver.html
[Pipenv]: https://pipenv.pypa.io/en/latest/
