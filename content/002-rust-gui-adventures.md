+++
title = "Rust GUI Adventures - Part 1"
date = "2025-01-29"
description = "Let's write a fair GUI app in Rust."

[taxonomies]
categories = ["gui", "rust", "iced", "lescan"]
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