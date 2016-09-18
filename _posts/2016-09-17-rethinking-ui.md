---
layout: post
title: Rethinking UI
---

### TL;DR

Thinking about switching to Qt, unsure how to deal with plugins. Please comment on it [here](https://github.com/emoon/ProDBG/issues/327)



### Long version

Sometimes you have to take a step back and look at the bigger picture when implementing something. Does the assumptions you once made still hold true? Is the path forward more complicated than anticipated with the given choices you made?

A while ago I made the call to use dear imgui for ProDBG. There are many good reasons for doing that:

### Upsides

* Full control over all the rendering
* Cross-platform
* Small library and great style of programming (less state tracking on the user side).

### Drawbacks

* Creating complex UI with resizing (layout) is more work than in other UI libs.
* More work is required around the UI itself

I was prepared to put in the required time to it. For example: Me and Graham Wihlidal (hi!) ported Scintilla to dear imgui. I even had some contracted work on improving the docking system, implement memory view and more. The amount of work for this was more than I hoped (isn't it always?)

Now when looking at the road a head I did a deep dive in what needs to be done to get to a place I want to be. I started looking at text. It turns out that text is hard, very hard. dear imgui supports custom fonts, the ability to load new fonts, etc. But getting proper text rendering that looks great for most languages is quite daunting task. While *most* code is written in English and that is easy to support I still want comments in other languages like Arabic, Japanese, Chinese, etc to work and look correct. Other things that I will need to do is to add caching. Right now the UI is being re-rendered all the time and that is really bad if you are on a laptop on battery. This is solvable as well but all the small issues add up.  Now getting this done is not impossible by any mean but it's not really where I want to spend my time.

So then comes the question: Where do I go next? Should I (and my contributors) still spend time working on this or is there an alternative?

### Alternatives

Good UI libraries aren't that easy to come across. Some options would be

* Qt
* wxWidgets
* GTK+

I have used both Qt and wxWidgets (for C++) in the past and ~3 years ago ProDBG actually used Qt. One of the reasons I stopped using that was performance issues. Now it turns out that the way some of the code we did at that point could have been optimized quite a bit. After spending more time and investigate these libs some more I think that Qt is the only one that makes sense. I want the UI to look the same across platforms and Qt provides that.

One problem now is my language of choice (Rust) really doesn't support Qt. Good news is that some work is being done on that front so that should be a solvable problem.

### Plugins

The largest issue at hand then is how to deal with plugins. The way my plugins works today is that a C struct with function pointers is being passed in to the plugins and it provides all the functionality that is needed.  The good thing about providing an API like this is that plugins doesn't need to link with the host library that provides the UI.  It makes plugin development much easier as it doesn't directly link with the same libs as the host application.  It also make it possible to upgrade with bug fixes on the host side while plugins stay the same.  If going towards Qt this becomes much more complicated so I have been thinking about how to solve this.

1. Expose only a small part of Qt in a C API. Problem is how to deal with signals/slots?
2. Just have plugins link with the same version of the Qt as host is using.
3. Have plugins being totally separate executables and have them communicate with the host app.
4. ????

I will dive into option 3 a bit more. This one is quite appealing but the question is how well it would work in practice. One would need to parse the cpp headers to auto-generate RPC bindings for both the host and the client side. This is similar to the approach that is being used to create the Qt bindings for Rust. Upside here is that it would be possible to generate these bindings to many langs that can run fully separated.
My biggest concern is performance and how fine-grained the communication can be. Say that a user has created some widget. And want to get the state for it. In order to get the state one would need to:

1. Create a RPC message & send it
2. Host gets the message and replies to it. Depending on the type of message this may need to be done single threaded (Unsure what you can do multi-threaded with Qt)
3. Data is being sent back to plugin.

Given that communication will 99% of the time be done on a local machine I would assume this is fairly fast but I just wonder if it will be fast enough. Host can certainly be multi-threaded and have a thread for each plugin connection. Still this is a concern if lots of small calls are being done from the plugins to the host.

I'm extremely torn on which approach to go with here. Having plugins linking directly with Qt would suck. I much prefer an approach where that wouldn't be needed. If you have any suggestions around this please comment [here](https://github.com/emoon/ProDBG/issues/327) (you can also contact me on twitter but I prefer to use the GitHub issue as one place to have the discussion)
