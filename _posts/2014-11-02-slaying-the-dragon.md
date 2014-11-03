---
layout: post
title: "Slaying the C/C++ Dragon"
date: 2014-11-02T17:13:57-08:00
request_feedback: true
---

Work has been pretty busy this week so my off-time has been spent reading more than coding.
And now that I've decided that I'll be making my own "engine" and using C or C++ for a large
portion of it, my next task is to decide between C and C++.

<!-- more -->

I found [a book][c-easily] that sounded like exactly what I needed to see how this could be
done in straight C and it was on Kindle Unlimited so I could just read it for "free". So I
did, it was pretty short. It did a pretty good job of showing me the _how_, I'm just not sure
I agree with all of the _whys_. Or, at least, I don't think the _whys_ make up for the
limitations imposed by going straight C. So, ultimately I've decided I'm going to go with
C++11.

So I grabbed a copy of [_A Tour of C++_][tour-of-cpp] to get some familiarity with C++11, since
all of my C++ books are pretty old. Learning C++ has been a long running challenge for me and
this project is my chance to finally slay that dragon.

I started trying to learn C++ back in high school and had limited success. I started with
a Borland C++ Compiler user's manual, which was probably my first mistake. I tried several times
over the years and a made a few small things, but never kept with it long enough for the
intricacies of the language to really sink in. I've been doing Objective-C professionally for
a few years now though, and that has provided some level of additional comfort with C++, but
this is still going to take quite a bit of learning on my part to actually do this. C++ has
some functionality that Objective-C lacks that I'm really looking forward to getting some
familiarity with (Templates/Generics primarily).

I'm still reading _A Tour of C++_, I'm about halfway through, and I'm feeling more and more
comfortable with C++. I'm hoping to get back to writing some code for the engine next week,
since work should ease up now that our recent deadline/demo has been done and we have a bit
of "slack" time, so I should have more code writing energy when I get home next week.

# Other Stuff

## Pixel Art

I also stumbled across these [video series][how-to-pixel-art] (both the "How To Pixel Art"
series and the "Pyxel Edit Tutorial Mini Series") and am now thoroughly inspired
and excited to make my own pixel art! Which is weird for me, I've always wanted (not just
needed, but _wanted_) to have someone else do art for me, because I'm generally not very
good at it and not really interested in putting in the time to get better at it. However,
after watching those videos I feel like not only _can_ I do some art, but I really _want_
to! So look forward to some of that coming here in the near future.

## Jonathan Blow's New Language

I also watched Jonathan Blow's [latest talk and demo][jblow] about the new game development
focused programming language he wants to create. He has some good points and I look forward
to seeing where he goes with the idea. In his previous talks on the subject he mentioned
how Go-lang was close to being what he wanted out of a programming language for games,
however it's garbage collected and that meant it was unacceptable (to him) for game
development. I bring that up because the syntax in the demo looks pretty similar to Go, so
that's interesting. I sent him an email after the first talk suggesting that he look into
modifying Go (it's an open language) and remove the garbage collection, or work in a way to
disable it and do manual memory management. I didn't expect to get a response (and didn't),
but I still think it's worth looking into, at least.

[tour-of-cpp]: http://www.stroustrup.com/Tour.html
[c-easily]: http://www.amazon.com/C-Easily-SDL2-Stephen-Jones-ebook/dp/B00JVAO3OO
[how-to-pixel-art]: https://www.youtube.com/user/achebit/playlists
[jblow]: https://www.youtube.com/watch?v=UTqZNujQOlA

