---
layout: post
title: "Pac-Man Ghost AI Visualisation"
request_feedback: true
---

Have you ever noticed how the ghosts in Pac-Man don't all chase him in the same fashion? Does that kind of thing
pique your interest?  Well, it does mine! So I built a thing that visualizes their AI strategies and that's what
this post is going to be all about.

<!-- more -->

# History / Why I Built It

I went Googling for this very subject once many years ago and found [The Pac-Man Dossier][dossier], and subsequently
read it cover to cover (or top to bottom, as the case may be). I found the info in Chapters 2-4 to be a
fantastic study on game design in general, but more specifically game *AI*. The fact that this game is still
challenging and fun to this day is a testament to just how well-tuned these AI algorithms are (a nod should
also be given to all of the other aspects of the game design as well, but I digress). With just these 4
ghosts the developers have been able to keep Pac-Man players on their toes for 30+ years.

More recently I was going through the [Route Finding][route-finding] lesson on [Khan Academy][khan] and it
used Pac-Man and his paranormal buddies as an example use case for path/route finding, which I knew from my time
with the Dossier was not *entirely* accurate. The ghosts don't actually do any path finding. They just measure
distance, in tiles from their current (more accurately, their next) tile to their target tile. This measurement
ignores walls and other obstacles, it's an "as the crow flies" measurement, which, strictly speaking, is not
path finding at all.

[![Wrong on the internet!][wrong-on-the-internet]][xkcd]

# The Visualization

One of the cool things about [Khan Academy][khan] is that they have a platform for creating interactive demos which
can be forked (or Split Off, as they call it), shared, even embedded into pages... like this:

<h2><a href="http://www.khanacademy.org/computer-programming/pac-man-ghost-ai-visualisation/6163767378051072">Pac-Man Ghost AI Visualisation</a></h2> <script src="http://www.khanacademy.org/computer-programming/pac-man-ghost-ai-visualisation/6163767378051072/embed.js?editor=no&amp;buttons=yes&amp;author=yes&amp;embed=yes"></script> <p style="padding:4px">Made using: <a href="http://www.khanacademy.org/computer-programming">Khan Academy Computer Science</a>.</p> 

That is the visualization I made, not because "Someone is wrong the internet", but because I thought it'd be a
fun exercise to write, and, if I was successful, a fun little program to show the elegance of the Pac-Man ghost
AI without having to read the whole Dossier.

# How It Works

TODO: write this section

# Conclusion

Turns out I was definitely right about the fun to write part. Whether or not it's fun *to play* or demonstrates
the AI in an easy to understand fashion, I'll let you be the judge of. I hope it's as fun for everyone else as
it is for me to play around with, because every time I open the damn thing I end up running Pac-Man around for
way more time than I intended to! Overall, I'm going to consider the project a success :)

[dossier]: http://home.comcast.net/~jpittman2/pacman/pacmandossier.html#Chapter_3 "The Pac-Man Dossier"
[route-finding]: https://www.khanacademy.org/computing/computer-science/algorithms/intro-to-algorithms/a/route-finding "Route Finding, Computer Science - Khan Academy"
[khan]: https://www.khanacademy.org "Khan Academy"
[wrong-on-the-internet]: //imgs.xkcd.com/comics/duty_calls.png "xkcd - Wrong on the Internet"
[xkcd]: http://xkcd.com/386/
