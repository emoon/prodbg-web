---
layout: post
title: ProDBG switches to Rust 
---

Test

As some of you know that follows me on twitter I don't like C++ all that much. It has it's usually issues with having header files, crappy templates and the list goes on.
The problem is what alternatives are there? I looked at switching to C# but getting that work great on all platforms looked to be a fair amount of challenge (for example at the time x64 wasn't supported in an easy way on Mac) and some people didn't like the choice.

I'm a bit fan of (Common) Lisp in general. I love how it's macros work and it has an elegance found in few languages. Yet it's a bit alien to many people and would make it harder for people to contribute to the projects. Also If possible I want a language which doesn't have a GC and while Lisp code can be made fast it takes some effort in doing so.

Entering Rust
-------------

I have been following the Rust project for a few years but never done anything with it until it hit 1.0 in May of 2015. When I started to look closer at Rust I want to see if it was something for me. On paper it looks good (listed the ones I found most interesting)

* Guaranteed memory safety
* Threads without data races
* Minimal runtime
* Efficient C bindings
* No garbage collection
* Modules
* Uses LLVM 
* Compiles to native code

I started to try it out to see where it would take me. I also did this presentation about Rust found [here](http://prodbg.com/rust_pres/index.html)

At this point I also started to look at platform support for Rust. It has to support Mac, Windows and Linux. Usually Windows support is okayish at the best for 'new' languages. I was pleased to see that Rust works great on Windows as well so big kudos to the Rust Team for that.

My other question was: "Is Rust used for any larger projects?" Jumping on to a language which will not be used in a few years would suck but also not the end of the world but would be painful.
Turns out that Rust is written in [Rust](https://github.com/rust-lang/rust) and is fairly big. [Servo](https://github.com/servo/servo/wiki/Design) from Mozilla is big as well and is also written in Rust.

Will Rust work for plugins?
---------------------------

My next step was to over the Christmas period try to just add support for Rust to be used in plugins (right now C/C++ is supported) to find out how well this would work when writing plugins in Rust. Turns out that wrapping Rust code on top of a C api based on structs with function pointers works great. Here is an (early) example https://github.com/emoon/ProDBG/blob/master/src/prodbg/tests/rust_api_test/src/lib.rs

While a bunch of the code in this example can be made nicer by 'Rustifying' it a bit this is pretty much just calling down to the C API which was what I wanted to prove.

