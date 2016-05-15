---
layout: post
title: This wee.. err.. months in ProDBG!
---

I have seriously been really bad at updating this place but I **really** hope to improve on going forward. So what has happed since last time? A lot in fact.

![]({{ site.url }}/images/prodbg_uae_2015_05_15.png)

The is the status of today. It's now possible to have ProDBG (the Rust version) debugging some very basic things. Here it's connected with my fork of [FS-UAE](https://fs-uae.net) where I'm working on adding support for Remote debugging. The way this works is that I have a small server running in FS-UAE which is based on the [GDB serial protocol](https://sourceware.org/gdb/onlinedocs/gdb/Remote-Protocol.html)

This whole FS-UAE backend is also implemented in as a plugin for ProDBG using the Rust version of the API. I will explain the details about how the actual API is implemented at some point but the TL;DR version is that it's C based which makes it easier to write plugins for ProDBG in other languages and currently Rust, C/C++ is supported out of the box.

The main part of the move over to Rust has been completed also. There are still some things to move over which is lacking but nothing major. Also there will still be bunch of external/3rd party code written in C/C++ (like [Scintilla](http://www.scintilla.org), [bgfx](https://github.com/bkaradzic/bgfx), [dear imgui](https://github.com/ocornut/imgui), etc) as it wouldn't make any sense to port them over for the sake of porting.

Still ProDBG is far from being really usable as I have taken a bunch of shortcuts just to get things up and running. The most important parts though which is the API and how plugins talk to each-other there now shortcuts has been taken so I really hope to have some basic native debugging running soon as things are really starting to move again which is great!

Something that I get asked now and then "What if company X releases debugger Y that is better than ProDBG. What happens then?" If there would be a debugger released which fits my needs *exactly* (Free, open source, has Rust support, native, etc) I **may** consider switching but to me it seems very unlikely to happen to fit everything I want. In the end I mostly do this project because I think it's fun and it's mostly for my own benefit and if people find it useful I'm all happy for that!

Also I have set [Gitter chat](https://gitter.im/emoon/ProDBG) for ProDBG so if you want to know more or would like to help out please drop by!

Until next time!

