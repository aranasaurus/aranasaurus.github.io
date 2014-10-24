---
layout: post
title:  "Game Dev Week"
date:   2014-10-23 09:42
---

By day I'm a mobile developer working for [Esri](http://www.esri.com) at their [Portland R&D Center](http://pdx.esri.com).
By night (and weekend), when not playing with my kids, I really enjoy making games, though. So recently I took a week off
from the day job to work on a game idea I've been rolling around for quite some time (while the kids are at school). I had
a prototype working but I wanted to give it my full attention for a week and see where I could get it in that time. I'll
talk more about the details of the game in other posts, this one is more about the experience of said "Game Dev Week".

<!-- more -->

I started off the week trying to get the [framework][moai] I had chosen to build and work. I really like the way Moai is
supposed to work: you set up a host for each platform you want to support and write a layer that takes the platform specific
stuff (like inputs) and feeds those things to Moai, then all of your game logic is the same for all platforms but you just
have to write the parts that differ. So I really like the __idea__ of Moai, but I couldn't get it to work. I don't want to
say that there's anything wrong with Moai, per se, but their documentation leaves a lot to be desired and I just wasn't dev
enough to get it working with my environment and requirements. I spent 2 days trying to do this and ended up giving up, so
that's a significant portion of my week essentially wasted.

The original prototype was done in [Löve][love2d] which is really easy to set up and do rapid development with, however it
didn't support mobile (officially) which was why I went with Moai. However I found this [branch][love-ios] of Löve which
touted some support for iOS (and it was branched off a branch with support for Android), so I grabbed that and decided to
use that for the rest of the week. I had to rewrite all of the stuff from my prototype (which was done with nothing but
gamepad controls support), plus it was a prototype, and that's what you do with prototype code. By the end of the week I
had my mobile version with touch controls but not much else than my prototype had (in fact, slightly less), which was kind
of disappointing. And when I went to implement the next part of my game I realized that I was really going to end up needing
3D support, but only for the background. While this may be possible with Löve, it's not what it's designed for and it would
be a hack, even moreso than the mobile support (which is actually quite good), and I just didn't want to go down that road.

So I came to the conclusion that what I'm really going to have to do is set up an OpenGL context of my own and basically do
what I wanted Moai to do, but on a smaller (more focused) scale. I plan on using [SDL][sdl] to handle windowing and input,
but that means using C++, which I don't know very well. I'm considering doing lua bindings for the pieces of SDL that I need,
as I couldn't get the SDL-lua bindings that I found to build on my machine and I didn't want to go down that hole again.

So all that to say: I got the wind taken out of my sails for the current game idea and I need to step back and try again
with something smaller, which brings me to...

# A Game a Week

Inspired by [this post][game-a-week-gamasutra] by [Rami Ismail][rami] ([this blog too!][game-a-week-msminotaur]), I'm going
to start building (and releasing) a game a week. The gist of the idea is that you build lots of _little_ games and put them
out there for people to play. Get the experience of making something and finding out what sucks, what doesn't, and why. But
doing so with tiny projects that you're not sinking too much time/energy into. Another big part of this is that you don't
touch the game again after the release, at least not as part of the Game a Week challenge. You can pick the game up later
and turn it into a full sized project, but each week is supposed to be something completely new. For my first few weeks I'm
going to keep it super duper simple, like Pong and Breakout clone simple, so that I can get an engine of sorts pieced together.
After I get a few of those out of the way and have some tools to work with I'll start getting a little more adventurous with
my ideas.

Along with the game each week I'll be posting a post-mortem about it here, to log the process and provide access to the game!
It's also possible that I'll be posting about the week's game throughout the week, depending on progress, of course.

I'm not sure how long I'll do the challenge, but my initial goal is going to be 8 weeks, starting next week. This post is the
first step in the process and I'll probably get started on the Pong clone this week, even though it'll be a short week, so I
can get the initial project set up stuff out of the way and be ready to go for reals next week.

Anyway, I hope you'll come back and try my games on this challenge, and send me some feedback on [twitter](http://twitter.com/aranasaurus)!

[moai]: http://getmoai.com "MOAI SDK"
[love2d]: http://love2d.org "Löve"
[love-ios]: http://love2d.org/forums/viewtopic.php?f=12&t=76985 "Löve for iOS"
[game-a-week-gamasutra]: http://www.gamasutra.com/blogs/RamiIsmail/20140226/211807/Game_A_Week_Getting_Experienced_At_Failure.php "A Game A Week - Gamasutra"
[game-a-week-msminotaur]: http://msminotaur.com/blog/?cat=4 "Game a Week - msminotaur"
[rami]: http://ramiismail.com "Rami Ismail of Vlambeer"
