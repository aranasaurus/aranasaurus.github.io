---
layout: post
title: "Pac-Man Ghost AI Demo"
request_feedback: true
---

Have you ever noticed how the ghosts in Pac-Man don't all chase him in the same fashion? Does that kind of thing
pique your interest?  Well, it does mine! So I built a thing that visualizes their AI strategies and that's what
this post is going to be all about.

<!-- more -->

# History

I forget why I went Googling for the ghost AI's once many years ago, but when I did I found [The Pac-Man Dossier][dossier],
and subsequently read it cover to cover (or top to bottom, as the case may be). I found the info in Chapters
2-4 to be a fantastic study on game design in general, but more specifically game *AI*. The fact that this
game is still challenging and fun to this day is a testament to just how well-tuned these AI algorithms are
(a nod should also be given to all of the other aspects of the game design as well, but I digress). With
just these 4 ghosts the developers have been able to keep Pac-Man players on their toes for 30+ years.

More recently though, I was going through the [Route Finding][route-finding] lesson on [Khan Academy][khan] and it
used Pac-Man and his paranormal buddies as an example use case for path/route finding, which I knew from my time
with the [dossier] was not *entirely* accurate. The ghosts don't actually do any path finding. They just measure
distance, in tiles from their current (more accurately, their next) tile to their target tile. This measurement
ignores walls and other obstacles, it's an "as the crow flies" measurement, which, strictly speaking, is not
path finding at all.

[![Wrong on the internet!][wrong-on-the-internet-image]][wrong-on-the-internet-comic]

# The Demo

One of the cool things about [Khan Academy][khan] is that they have a platform for creating interactive demos which
can be forked (or Split Off, as they call it), shared, even embedded into pages... like this:

<h2><a href="http://www.khanacademy.org/computer-programming/pac-man-ghost-ai-visualisation/6163767378051072">Pac-Man Ghost AI Visualization</a></h2> <script src="http://www.khanacademy.org/computer-programming/pac-man-ghost-ai-visualisation/6163767378051072/embed.js?editor=no&amp;buttons=yes&amp;author=yes&amp;embed=yes"></script> <p style="padding:4px">Made using: <a href="http://www.khanacademy.org/computer-programming">Khan Academy Computer Science</a>.</p> 

That is the visualization I made, not because "Someone is wrong the internet", but because I thought it'd be a
fun exercise to write and, if I was successful, a fun little program to show the elegance of the Pac-Man ghost
AI algorithms without having to read the whole Dossier.

__NOTE__: You can view the (javascript/[processing.js]) code, which is pretty well commented, and even fork it and make changes to it if you
want by clicking the "View on Khan Academy" button.

## Graphics

It runs the AI algorithms for each ghost as laid out in the [dossier] and renders some extra graphics to show
the information they used/calculated to make their choices of where to go:

 * A tile outline of each ghost's color which represents that ghost's current target tile.
 * A transparent representation of the ghost and the direction it will go when it gets to the next tile. The
 ghosts always plan their moves one tile in advance.
 * Lightly outlined tile(s) with a number in it. These tiles represent the tiles that the ghost considered when
 choosing the direction to go in the next tile. The numbers represent the distance from that tile to that ghost's
 current target tile.

The simulation runs by tile. Whereas the game runs fluidly between the tiles, the algorithms themselves only care
about the tiles, so making it advance per tile made it easy to see how things go step by step.

## Controls 

You can use the arrow keys (or WASD) to move Pac-Man step by step through the simulation, or if you hold Shift
and press the movement keys, you can change the direction that Pac-Man is facing and force all of the ghosts to
recalculate their target tile. This is important to note because you'll notice that the target tiles don't
update themselves every step. This is because the ghosts only update their target tile when they have a choice
to make, and since they can never go backward, they only have a choice when they're at an intersection in the
maze. This also allows you to see how you can "control" the paths of some of the ghosts at certain points.

You can also use the 1-4 number keys (or the first letter of each ghost's name: Blinky, Pinky, Inky, Clyde) to
toggle the visibility of each ghost's decision making graphics. The target tile will always be visible, this
refers only to the numbers and the next tile graphics.

# How It Works

## First, a bit about Pac-Man

For those that haven't read through the [dossier] I'd like to give a brief overview of a few points about how the
game works. In playing the game you may have noticed that the ghosts don't just chase Pac-Man the whole time
(regardless of their strategy for getting *to* him), but sometimes they actually run *away* from him even if 
Pac-Man hasn't eaten an energizer dot. This is because the ghosts have three distinct modes that they operate
in, depending on various other factors in the game.

Here's a quote from the [dossier][modus-operandi] that summarizes the three modes quite nicely:

> Ghosts have three mutually-exclusive modes of behavior they can be in during play: chase, scatter, and frightened. Each mode has a different objective/goal to be carried out:

> 1. __CHASE__—A ghost's objective in chase mode is to find and capture Pac-Man by hunting him down through the maze. Each ghost exhibits unique behavior when chasing Pac-Man, giving them their different personalities: Blinky (red) is very aggressive and hard to shake once he gets behind you, Pinky (pink) tends to get in front of you and cut you off, Inky (light blue) is the least predictable of the bunch, and Clyde (orange) seems to do his own thing and stay out of the way.
> 2. __SCATTER__—In scatter mode, the ghosts give up the chase for a few seconds and head for their respective home corners. It is a welcome but brief rest—soon enough, they will revert to chase mode and be after Pac-Man again.
> 3. __FRIGHTENED__—Ghosts enter frightened mode whenever Pac-Man eats one of the four energizers located in the far corners of the maze. During the early levels, the ghosts will all turn dark blue (meaning they are vulnerable) and aimlessly wander the maze for a few seconds. They will flash moments before returning to their previous mode of behavior.

Another important note to go along with that is that they can never reverse direction... except when they
switch between the above modes. They can also change their movement *speed* at certain numbers of remaining
dots in the level.

There are other finer points of detail in the [dossier], but I think that should cover what we need to know
for this project.

## The Ghosts and their target tiles

As previously stated the ghosts only update their target tile at intersections in the maze. This target
tile calculation is really where the difference in behavior comes from. Here's a brief explanation of each
of their target tile algorithms (in chase mode). For more in-depth analysis check out [Chapters 3-4][dossier3]
in the [dossier] or the [source code][pac-man-khan] for the demo.

 * __Blinky__ - The red ghost is the most straight forward. Blinky's target tile is always Pac-Man's current tile.
 * __Pinky__ - The pink ghost is pretty straight forward too. Pinky's target tile is always a tile 4 tiles ahead
 of Pac-Man's current tile. See the note below about the bug which is replicated in the demo.
 * __Inky__ - The blue ghost takes a three step process to figure out its target tile:
   1. Find the tile 2 tiles in front of Pac-Man.
   2. Find the vector between Blinky and the tile in step 1.
   3. Double the length of that vector. The other end of that vector is Inky's new target tile.
 * __Clyde__ - The orange ghost has two different modes for picking his next target tile. He switches between these
 modes based on his proximity to Pac-Man:
   * When Clyde is within 8 tiles of Pac-Man: Clyde targets a tile just off the maze at the bottom left.
   * When Clyde is farther than 8 tiles away from Pac-Man: Clyde targets Pac-Man's tile, the same way Blinky
   does.

 __NOTE__: Pinky's and Inky's "*x* tiles in front of Pac-Man" measurements work as you'd expect in every direction
 except for up. There was a bug in the original game that when they did this measurement in the up direction
 they actually measured *x* tiles up __AND__ *x* tiles to the left. This bug is replicated in the demo (on
 purpose).

## Differences from the game in the demo

I really wanted this thing to be as focused on the Chase mode strategies of the ghosts as possible. In order
to do that I made the following choices:

 1. There are no pellets. The pellets are very important for the game, but not so much for the chase mode 
 strategies of the ghosts. Also I really didn't want the added visual clutter.
 2. There are no mode or speed changes. I wanted to focus on the chase mode strategies specifically, since
 the other modes are relatively uninteresting. Since there are no pellets there are also no speed changes.
 3. No collision detection between ghosts and Pac-Man. I didn't want this to be a playable version of the
 game so the death sequence just didn't make a lot of sense to try and imitate and there's a handy Restart
 button for anytime you want to reset everything. I may go back and add some kind of messaging when a ghost
 touches Pac-Man later, but for now it seems fine without it.
 4. Tile based movement. The real Pac-Man runs fluidly at so many frames per second, and the characters all
 move from tile to tile pixel by pixel. I wanted to be able to show how things are done step by step and the
 algorithms are all tile based so I decided to just advance the game world by one step every time Pac-Man
 enters a new tile. There are some edge cases where this means that this demo will behave differently from
 the actual game (like [here][just-passing-through]), but I figured this was a trade off I was willing to
 make.
 5. Distances for the algorithms are scaled. In the actual game the mazes have more tiles in them than in
 the demo, so for my implementation of the algorithms I scale the distances used in them to account for that.

# Conclusion

Turns out I was definitely right about the fun to write part. Whether or not it's fun *to play* or demonstrates
the AI in an easy to understand fashion, I'll let you be the judge of. I hope it's as fun for everyone else to
play around with as it is for me, because every time I open the damn thing I end up running Pac-Man around for
way more time than I intended to! Therefor, I'm going to consider the project a success overall.

[dossier]: //home.comcast.net/~jpittman2/pacman/pacmandossier.html "The Pac-Man Dossier"
[dossier3]: //home.comcast.net/~jpittman2/pacman/pacmandossier.html#Chapter_3 "The Pac-Man Dossier"
[route-finding]: //www.khanacademy.org/computing/computer-science/algorithms/intro-to-algorithms/a/route-finding "Route Finding, Computer Science - Khan Academy"
[khan]: //www.khanacademy.org "Khan Academy"
[pac-man-khan]: //www.khanacademy.org/computer-programming/pac-man-ghost-ai-visualisation/6163767378051072 "Pac-Man Ghost AI Visualization - Khan Academy"
[wrong-on-the-internet-image]: //imgs.xkcd.com/comics/duty_calls.png "xkcd - Wrong on the Internet"
[wrong-on-the-internet-comic]: //xkcd.com/386/
[modus-operandi]: //home.comcast.net/~jpittman2/pacman/pacmandossier.html#CH2_Modus_Operandi "The Pac-Man Dossier"
[just-passing-through]: //home.comcast.net/~jpittman2/pacman/pacmandossier.html#CH3_Just_Passing_Through "The Pac-Man Dossier"
[processing.js]: //processingjs.org "Processing.js"
