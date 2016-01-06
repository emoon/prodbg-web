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

