---
layout: post
title: ProDBG switches to Rust 
---

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

Can ProDBG be written in Rust *without* rewriting everything at once?
------------------------------------------------------------------

I started playing around with exposing parts of the C++ code and just calling "prodbg_main" the first and only thing in the Rust main code. After fixing some linking errors it all worked! That is one of the upsides with Rust as being a compiled language you can link with C/C++ code without needing some DLL bridge.

Next I started expose some more parts of the code. I also wrote a "plugin handler" which loads the C dlls and just shows one of them on the screen. I also added reloading of plugins at the same time so now when compiling a plugin (which can be written in Rust or C/C++) it gets reloaded on the fly which is nice.

The code can be found here which is just a testbench for playing around before diving into the main code https://github.com/emoon/ProDBG/blob/rust/src/ui_testbench/src/main.rs

So what now?
------------

After playing around with Rust the language has certainly grown on me. It's a great language which offers some great features (see my presentation) again about esp borrowing/lifetime. The cool thing with borrowing/lifetime is that the Rust compiler can reason about truly non-mutable state which allows enabling noalias passes in LLVM which can make Rust code outperform C code.

Taking part in the Rust community and following it where the language is heading gives me great hope for the future so I have decided to take the step of writing ProDBG in Rust. Now that being said I will still rely on some external code (like dear imgui, Scintilla, bgfx) still being in C++. There is no reason for me to try to reimplement that at all. Also the exposed API to plugins will still always be C. It makes it *much* easier to add support for other languages in the future (Lua, Python, JS, .NET, etc)

Drawbacks
---------

Switching to a new language which hasn't been used by many people compared to C/C++ always has it's sets of challanges.Slower compiler (which is being worked on), Harder to debug (no good front-end yet, but I guess that is a good motivation for me :) less people know the language and one never knows what will happen it the future. That being said I still believe that Rust is on the right trajectory and it feels right for me to switch to it. 

The largest drawback though is that it will slow down the development of the project initially which sucks. I had hoped to have a 0.1 at this point and this will delay it futher.

Final words
-----------

So there you have it. I haven't taken the decision lightly because it affects the contributors I have. Notice that the API will still be in C and I will accept contributions of plugins that is written in C/C++. 
Still I think it's the right thing to do and here is a great read http://www.ncameron.org/blog/my-thoughts-on-rust-in-2016/ on what happened during 2015 and looking at whats a head in 2016. 


