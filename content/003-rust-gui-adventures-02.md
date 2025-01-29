+++
title = "Rust GUI Adventures - Part 2 - Backend (?)"
date = "2025-01-29"
description = "Backend adventures of lescan."

[taxonomies]
categories = ["rust", "lescan"]
tags = ["rust", "backend", "parsing", "lescan"]
+++

<style>
.img-container {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 20px; /* Space between images */
}
</style>

So to write a GUI app like Aegisub, first I need to decide how to handle the data about translations in two contexts:
1. In the memory
2. In the save file

## So what kind of data do I need to store?

To understand our data, let's get a look at some Manga or Webtoon translations.

<div class="img-container">
    <img src="https://i.pinimg.com/564x/08/31/c5/0831c5ee69dfe42102665dfc61227fe3.jpg" alt="Balloon Types" width="300"/>
</div>

*Credits: https://lezlynorman.com/drawing-comics-speech-balloons/*

For nearly all type of comics, we store texts (speeches, thoughts, etc.) in balloons, though there is a lot of exceptions.
For the sake of simplicity, I will refer all non-balloon texts as "balloons" and they will be sub-types of balloons.

There is a lot of types of balloons, but for now we can generalize them as:
* **Dialogue**: Balloons that contains speeches or monologues between characters. Usually they are in a circular shape.
* **Square**: Balloons that shaped as squares. They usually contains thoughts or some kind of information outside of speech.
* **Thinking**: Balloons that contains thoughts of characters. They are usually shaped as clouds but there are a lot of variations.
* **ST**: Short for **S**ound **T**ext. They are usually used for sound effects. They are usually shaped as rectangles.
* **OT**: Short for **O**ver **T**ext. These are the texts that does not belong to any balloon.

We can store types with Enums in Rust:
```rust
#[derive(PartialEq, Debug, Clone)]
pub enum TYPES {
    DIALOGUE,
    SQUARE,
    THINKING,
    ST,
    OT
}
```

Then we need to store the texts in the balloons. A balloon can have multiple texts.
Also a single balloon can be sometimes appear as two balloons connected to each other like the photos below:
<div class="img-container">
    <img src="https://celclipaskprod.s3-ap-northeast-1.amazonaws.com/question/47b4/88353/1/295ea22d20ee733a6a1972ba32e5385c_small" alt="Connected Balloons" width="200"/>
    <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT__Cj9pmQq_Ku8W-mVIg60zj8XdcYK17tWUQ&s"/>
</div>

This means this connected balloons can have multiple texts. So we need to store them as a single entity.

Also a lot of times, after a translator translates a chapter, the translations will be sent to a proofreader to check for errors.
So we need to store the proofreaders' review as well.

Then we came to comments. Sometimes, translators or proofreaders (or whoever works with the file) may want to add some comments to the translations.
So we need to store them as well.

Now the somewhat tricky part, *sometimes* translators or proofreaders may want to add context to the translation by attaching different types of images to it.
In memory representation, we can store the image bytes, but in the save file, we need to store the image itself.

So lets create a struct for Balloon:
```rust
#[derive(Default, Debug, Clone)]
pub struct Balloon {
    pub tl_content: Vec<String>,
    pub pr_content: Vec<String>,
    pub comments: Vec<String>,
    pub btype: TYPES,
    pub balloon_img: Option<BalloonImage>,
}

#[derive(Default, Debug, Clone)]
pub struct BalloonImage {
    pub img_type: String,
    pub img_data: Vec<u8>
}
```

Now we need to store these balloons in a Document. A chapter can contain different pages if it's a manga, and multiple balloons.
I also want to have some metadata about the App, Script etc. So our document struct will look like this:
```rust
#[allow(non_snake_case)]
#[derive(Debug)]
pub struct Document {
    /// sff (Scanlation File Format) version. No big changes expected.
    pub METADATA_SCRIPT_VERSION: String,
    /// If you use this library for an app, it may come in handy to indicate your app's version.
    pub METADATA_APP_VERSION: String,
    /// Some other info you want to give/specify.
    pub METADATA_INFO: String,
    /// There is your balloons m8.
    pub balloons: Vec<Balloon>
}
```

Ah, in-memory representation is done. Now we need to store this in a file. How do we serialize this data though?

There are a lot of serialization formats like JSON, XML, YAML, TOML etc. Let's look at how Aegisub handles this.

Aegisub uses a format called ASS (Advanced Substation Alpha). Below is an example:
```ass
[Script Info]
PlayResY: 600
WrapStyle: 1

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: Expl, Arial,28,&H00FFB0B0,&H00B0B0B0,&H00303030,&H80000008,-1,0,0,0,100,100,0.00,0.00,1,1.00,2.00, 7 ,30,10,30,0

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
Dialogue: 0,00:00:00.00,00:03:00.00,Expl, NTP,0,0,0,,{\pos(20,20)}UwU
Dialogue: 0,00:00:00.00,00:03:00.00,Code, NTP,0,0,0,,{\pos(40,160)}OwO
Dialogue: 0,00:00:00.00,00:03:00.00,Expl, NTP,0,0,0,,{\pos(20,550)}UwU
```

There is 3 main sections in an ASS file. `[Script Info]`, `[V4+ Styles]` and `[Events]`.
* **Script Info** stores the info about .ass script, such as the app name, version etc.
* **V4+ Styles** section stores the subtitle styles in a CSV-like format.
* **Events** section stores the actual subtitles, timecodes and some additional per-subtitle info in a CSV-like format.

I actually don't like the ASS format that much because it is hard to read by human eye when you have a lot of styles and subtitles,
and I think there is a lot of standardised serialization formats out there, fast and used by many so it's unnecessary to just use a different format.

After a little thinking session, I decided to go with XML. Why you ask?
* XML is pretty extensible. Adding new features to files will be a breeze.
* There is a lot of fast XML parsers in a lot of languages.
* I think it is easier to read for humans than JSON.
* I like XML.

Also I want to have a really human-readable format. This will come in handy when translators ship their works to typesetters etc.
This format is basically a txt file and will have balloon texts line by line (every text is on a new line),
different balloons have a blank space between them. Also connected balloons will have "//" characters in between them. A simple example is below:
```
// This will be a normal dialogue
(): Hi, how are you:
// This will be Over Text
OT: I can't say I'm annoyed by him.
```

So, to serialize Document to XML, first, I thought about using Serde crate but i suddenly had an urge to code mine.
So I did. This format is pretty simple, so it was really easy to implement with Rust's `format!()` macro. For the serialization of the images,
I encoded them as BASE64 and stored them in the XML file. For the future, I might want to use Serde for this, but for now, this is enough.

You can see the whole code in the [repository](https://github.com/NandeMD/rsff). See you.