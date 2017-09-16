---
layout: post
title: Developer's Music Making Guide
---

Unfortunately, I missed Ludum Dare 39 because of some terrible flight arrangement and delay. For those of you who have never heard of Ludum Dare, it's a game jam that requires you to create an entire game from scratch in 48 hours. That includes all the assets like sprites, sound effects, and background music.

As a developer (and an unqualified designer), I can do sprites, but music is something that I never thought I could do. My games are doing *"fine"* without music until some day I decided to give music making a go. My original plan was to try it in Ludum Dare 39, which obviously didn't happen. So during that delayed flight, I wrote this guide to make sure: 1) I can pick it up for Ludum Dare 40; 2) to inspire you, as a developer (or anyone really), to try making some music.

I know nothing about music: I don't play any instrument, I haven't taken any music courses, I don't even know how do you call those music symbols, and I am usually the one eating all the snacks in a karaoke party. So if you are like me, you should definitely continue reading. Lots of "beginner's guide" regarding music making either include lots of jargons, or assume you have some basic knowledge — *not* this one. I will also make this a "quickstart" guide with lots of pragmatic advice and methods.

## Tools

Most of my amateur (amateur means they don't do it for a living) music producer friends use a paid software called **[FL Studio](https://www.image-line.com/flstudio/)**, which is quite powerful and frankly a bit overkill for our purpose. There is also a free online alternative (needs Flash) called [Audiotool](https://www.audiotool.com), which is not really beginner friendly (screenshots below, from official website).

![Audiotool UI](https://at-cdn-s01.audiotool.com/site-static/img/content/laptop.png?v=1502961473692)

I have spent quite some time trying to figure out all those equipments and failed to get anything done. Finally after lots of searching, I have discovered [Bosca Ceoil](http://boscaceoil.net) (unfortunately also needs Flash), which is really beginner friendly. Even though the functions are very limited, it's more than enough for making game music, especially 8 bit / chiptune style.

In this guide, I will not go into the details of Bosca Ceoil, but here is an excellent [video tutorial](https://www.youtube.com/watch?v=fZeZ75gM9p4) for understanding the interface. Note, when you use the standalone version, **DO NOT** press Esc, as it will exit without prompt or saving.

**TL;DL**: Download [Bosca Ceoil](http://boscaceoil.net)

## Scale

A scale is basically a group of notes that sound really well together. When you open Bosca Ceoil, it looks like a spread sheet with lots of marks on the left — those are notes. And on the bottom right, you see the scale. The default scale is a bit confusing, as it is marked as "Normal". However, it's not really normal in the sense that no one really writes music in this scale.

![Bosca Ceoil](https://monosnap.com/file/gXiJ5uiiyXYmwjbrftRdYy9dAAHCos.png)

Instead, most people write music in **Major** and **Minor** scales. The major scale is the most common one and it's also the scale referred to in the *Do-Re-Mi* song from The Sound of Music. Most people would feel that tunes in major scale sounds positive (e.g. happy) and in minor scale sounds negative (e.g. sad). So you can choose a scale based on the emotion you want to express in your music.

That's all you need to know about scale. However, there are two traps:

1) In practice, minor scale is usually used with variations. To simplify this, Bosca Ceoil actually uses something variation called "jazz minor scale", which might be different from minor scale referred to elsewhere.

2) In music notations (on the left), there is sharp (♯), which means higher in pitch, and flat (♭), which means lower in pitch. Bosca Ceoil always use sharp, which I believe is because the flat symbol is not included in the font. However, in some other references, they might use flat. So to confuse you further, look at the screenshot above, there is no **B♯**, why? Because **B♯** sounds the same as **C**. However, the **F♯** shown above can also be written as **G♭** because they sound the same. Confused now? Fortunately these are just some background stories, you do not really need to understand this to make music (and have fun).

**TL;DR**: Choose a scale before start: major for happy tunes and minor for sad tunes.

## Component of a Song

Melody, chords and beats are probably the most common "components" you will find in a song. When you listen to a song, you can easily tell the melody and the beat. However, the chords might not be very straightforward. According to Wikipedia, a chord is "two or more (usually three) notes that are heard as if sounding simultaneously". Chords usually act as the harmony of the song, adding "depth" to the melody.

Because chords have fewer variations than melody, after long time of studies, people have summarised some "best practice" of chord combinations (or formally known as chord progressions).

So here is the golden question: which part do you start with? There are two common ways:

1. **Start with melody and add the chords.** If you feel inspired and already have a melody in your mind, this is the way to go.
2. **Start with a chord progression and fill in the melody.** This may sound counter-intuitive but it actually works well especially if you have a tight deadline and just want to produce some tunes that sound ok, which is what people usually face in Ludum Dare. Also there is not much rule when writing melody, which can be overwhelming for beginners. However, choosing a chord can help guide your melody. **So I will focus on this method in this guide.**

**TL;DR**: For beginners with a deadline, start with a chord progression and then fill in the melody.

## Chord Progressions

[Autochords](https://autochords.com/) is a brilliant website to generate some chord progressions for your music.

![Autochords](https://monosnap.com/file/axDwTqdyC08ghw0FwNdvv3XI9pOKJZ.png)

For the screenshot above, I have generated the canon progression in C major. You can easily add this chord progression to Bosca Ceoil and have a listen, making sure it sounds the same as the website.

When adding the chords, you might have multiple choices: you can have a lower C or a higher C. In general, making the notes, or at least *the highest note (e.g. the G note in C chord)*  as "close" as possible will produce a better result. In fact, you can be bold and add whatever chords you prefer, as long as the notes, or the highest note in the chords are reasonably "close", it won't sound weird.

**TL;DR**: Start with a common chord progression, or not, as long as the notes, or at least the highest note are reasonably "close".

## Beats

There are three basic types of beats:

1. Bass Drum (aka. Kick)
2. Snare Drum / Clap
3. High Hat

You can open Bosca Ceoil and select "Simple Drumkit" as instrument to try them out. When composing a beat pattern, there is a simple rule to follow in order to avoid "off beat". **Start with a kick, and either have one snare on 9th or two snares on 5th and 13th**.

![1-9](https://monosnap.com/file/pJb6jyiLfE1fFiuA3OOCOroYsnTGvg.png)

![1-5-13](https://monosnap.com/file/a5q2XU5JUrpSr67SzQPlc8G4OLPYoy.png)

Consider these as the "basic" patterns and feel free to add on to them. However, adding too much might also cause it to go "off beat" so be careful.

**TL;DR**: Use **1 (Kick) — 5 (Snare) — 13 (Snare)** or **1 (Kick) — 9 (Snare)** as base of your beats, don't add too much though.

## Melody

Melody is the *easiest* and *trickiest* part. Easiest because you don't have to follow any rule: there is no "wrong" melody. Trickiest because there is no rule to follow, just like giving a beginner a blank paper to draw Eiffel Tower.

So the first advice is: **go wild**. However, if things get a bit *out of control* or a bit *too boring*, you can try the following advice:

1. **Use notes from the chords**. For example, if your chord is C, you are quite safe to use notes from C chord: C E G. 
2. **Don't jump too much**. For example, instead of going from C directly to G, try going from C to E first and then to G.
3. **Copy and tweak.** In the **4 x 4** sheet, fill first 4 with some notes (e.g. C D F D), and then copy paste this pattern to the rest cells. Then start tweaking to add some variations, like moving them a bit higher or lower, omitting some note, etc. This might sound stupid, but it is actually quite [widely used](https://en.wikipedia.org/wiki/Motif_(music)) in professional music composing.
4. **Change the length of a note**. In Bosca Ceoil, you can use mouse scroll change the length of a note. This adds variations if you feel the song is a bit "dull".
5. **Add a second layer**. Sometimes, you can try adding some notes in a higher or lower octave, to add more "depth" to your melody. However, don't add too much or your melody will be "jumping too much".
6. **Change instrument**. This will definitely affect how your melody "feels" and changing instrument may give you inspirations.
7. **Change speed**. Changing the speed of the melody (beats per minute, or BPM) can dramatically change the melody, which may also give you inspirations.

**TL;DR**: Go wild.

## Combining and Arrangement

Hopefully by now, you've already got your chords, melody and beats. It's time to combine them. In fact, they should all be combined automatically. However, there is one more thing: *make sure your chords will not steal your melody's thunder*. A good way is to lower the chords, or some of the notes in the chords. Or you can make use of the "low pass filter pad" in Bosca Ceoil. Changing the instrument of the chords might also help.

After you've got all the basic elements of short song, you can expand it to make it longer. A good technique is *copy and tweak*, things you can consider:

1. **Tweak your melody**. This is the most straightforward way and you will be surprised how little you need to change to add reasonable variation.
2. **Tweak your chord**. You can shift the chord, use a different progression or add more [sophisticated](https://en.wikipedia.org/wiki/Seventh_chord) chords.

![Arrangement](https://monosnap.com/file/ukj98tnVtYrPlF7AqkMRPQkJtkgGio.png)

For the screenshot above, I have only tweak some of the melody (*in blue color*) to add the variations. I have also added bass (*in red color*).

One thing that we did not cover is the ending. As most game music will be played on an infinitely loop, we don't actually need an ending. There are actually [several ways](https://en.wikipedia.org/wiki/Cadence_(music)) to end a song. The safest way, in the example of *C major scale* is to end with a C chord and a longer C note.

**TL;DR**: Copy and tweak

## Give it a Go

Of all the advices in the guide, here is the single most important one: **start writing music now**. Here is the bottom line: you can forget all about the rules, simply open up Bosca Ceoil and start messing around (formally s as *try and error*), I can guarantee you, you will end up with some results that can surprise yourself. Not only that, I find the process of writing music quite therapeutic and enjoyable. The result, even though may not be some masterpiece, is definitely *good enough* for your Ludum Dare game.

I've created some random fragments (*motifs*) while writing this guide. Altogether they take less than an hour and I am definitely going to adapt them to my next game. [**Have a listen**](https://soundcloud.com/insraq/sets/rand-pieces-1), if you think the result is okay, remember you can achieve that too!

