+++
title = "Rust GUI Adventures - Part 1"
date = "TBD"
description = "Let's write a fair GUI app in Rust."

[taxonomies]
categories = ["gui", "rust", "iced"]
tags = ["rust", "gui"]
+++

So if you are into fansubbing, you probably know that there are some tools that are used to make the process easier.
One of them is [Aegisub](http://www.aegisub.org/) and it's written in C++. It's a really powerful tool that has
a lot of features that are useful for fansubbing. I was using it for a long time and simply impressed by it's
broad capabilities.

After dropping fansubbing for good, I was into scanlations. I did some translation stuff for a few years for fun and
honestly, I can say tools in this are at best primitive compared to fansubbing tools. Some scanlation groups use
only notepad/Word for translations and Photoshop for typesetting. On the typesetting aspect, Photoshop is a really
powerful tool and you can't want more from it but for translations, it's a nightmare. I thought "Why not make a
tool that will make the translation process easier?"

I had 2 options for the language of the tool: Python or Rust (I only know these 2). I chose Rust because I wanted
my app to be performant and I thought it would be more fun to write it in Rust. (I also need to mention that I have a little to no experience with GUIs)
I started to search for GUI libraries for Rust and found a few options:

* [gtk4](https://gtk-rs.org/gtk4-rs/stable/latest/book/): I guess this is a really powerful library, and it has a really good set of widgets. Why not use it? Idk. I just don't feel like it.
* [xilem](https://github.com/linebender/xilem): This does seem too complicated (and experimental).
* [fltk-rs](https://github.com/fltk-rs/fltk-rs): I don't know why but I don't like it. Maybe because it's too C++-ish. Also looks like GTK4.
* [iced](https://iced.rs/): I liked it's simplicity and it's declarative UI approach. Also it's really easy to use (:)). Elm architecture seems really nice. Also
System74 decided to use it for Pop!_OS Cosmic DE so it must be good, right?

I choosed Iced because it's simple and I like it's architecture. Somehow it seems to have too few widgets but I think I can manage it.
I will write about my experiences with Iced and my app in the next posts.


+++
title = "Rust GUI Adventures - Part 1"
date = "TBD"
description = "Let's write a fair GUI app in Rust."

[taxonomies]
categories = ["gui", "rust", "iced"]
tags = ["rust", "gui"]
+++

If you're into fansubbing, you've probably heard of tools that make the process easier.
One of the most notable is [Aegisub](http://www.aegisub.org/), a powerful application written in C++.
It’s packed with features that are incredibly useful for fansubbing, and I used it for a long time.
I was always impressed by its broad capabilities.

After stepping away from fansubbing, I got into scanlations.
For a few years, I dabbled in translation work for fun, and honestly,
the tools available for scanlations are, at best, primitive compared to fansubbing tools.
Some scanlation groups rely solely on Notepad or Word for translations.
While typesetters have incredibly powerful commercial tool like Photoshop,
translators are stuck with Notepad or Word, which makes translation process tedious.
That got me thinking: *Why not create a tool to make the translation process easier?*

I had two language options for building this tool: Python or Rust (since those are the only two I know). I chose Rust because I wanted my app to be performant, and I thought it would be more fun to write it in Rust. (I should also mention that I have little to no experience with GUIs.) I began searching for GUI libraries in Rust and found a few options:

- **[gtk4](https://gtk-rs.org/gtk4-rs/stable/latest/book/)**: This seems like a powerful library with a robust set of widgets. Why didn’t I choose it? I’m not sure—it just didn’t feel right to me.
- **[xilem](https://github.com/linebender/xilem)**: This one seemed too complicated and experimental for my needs.
- **[fltk-rs](https://github.com/fltk-rs/fltk-rs)**: For some reason, I didn’t vibe with it. Maybe because it feels too C++-ish, and it reminded me of GTK4.
- **[iced](https://iced.rs/)**: I loved its simplicity and declarative UI approach. It’s also incredibly easy to use. The Elm architecture it’s based on seems really elegant. Plus, System76 chose it for their Pop!_OS Cosmic DE, so it must be good, right?

I ultimately chose **Iced** because of its simplicity and architecture. While it seems to have a limited number of widgets, I think I can work with it. In the next posts, I’ll share my experiences with Iced and the development of my app.