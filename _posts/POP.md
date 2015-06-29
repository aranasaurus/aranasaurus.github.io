# Intro

My favorite session at WWDC this year was definitely Protocol-Oriented
Programming in Swift. It gave a name to a programming paradigm I've been practicing
(unwittingly) for a little while now, and highlighted the fact that Swift was
designed from the ground up with this paradigm in mind. Perhaps that's why it
works so well. This talk also went over some new language features that would
make working this way even nicer.

I know I'm not the only one who enjoyed this talk for these same reasons. I've
talked with a lot of people about this talk since WWDC and I find that the reactions
boiled down into three basic types:

  1. I loved it! Favorite talk of the year! - These people cite mostly the same
  reasons as me for loving the talk so much.
  2. It blew my mind! I loved it! Favorite talk of the year! - These people were
  excited by the new ideas presented in the talk and were excited to bring the
  approach to their projects as soon as possible, but when they went Googling
  for some more info, they came up empty handed.
  3. That seemed interesting, but it was way over my head... I'm gonna come back
  to that one later.

What I wanted to do with this article is go over some things that I've picked up
while doing Protocol-Oriented Programming before I knew it had a name in the hopes
of giving those in camp 2 a little something to get started.

I may do another article for folks in camp 3, but it will be of a different style
(I'm thinking timestamp annotated notes for the video, like a companion guide).

# Getting started

Protocol-Oriented Programming is definitely not an all or nothing approach. You
can start mixing it in to your project today, either by refactoring pieces of
your current components or by adding a new component designed in this way. Protocols
in Swift can be applied to and used by any type (be it `class`, `enum`, or `struct`),
and so long as the interface they describe can be used by objective-c, they can
be used there too, so you are free to mix them in as slowly or whole-heartedly
as you like!

If you are going to be integrating with an existing system though, note that you
may end up having to alter or refactor some pieces of that system that you aren't
fully converting to POP. This is just the nature of the beast when it comes to
refactoring for dependency injection (don't worry, I'll get to that if you don't
know what "dependency injection" means).

# Start With a Protocol

I know, he said this in the talk, but it's worth repeating and was one of the
things that he said where I thought "Huh... yeah, why didn't I think of that?!".
I used to just start with a `class` (or `struct` or `enum` where it made sense),
and then later I'd create the `protocol` from the public interface of the type I
was creating. Now, I start by thinking about that public interface first, and
creating a `protocol` out of it. Then I go and start adding an implementation
for that protocol in a `class` (or `struct`).

## Naming

I'll usually name my `protocol` the same name as the implementation but with "Protocol"
affixed to the end. So a `Foo` struct would have a `FooProtocol`. I'm not sure
I really like this, but I haven't come up with or heard of a better alternative,
so I've stuck with it so far. I've considered flipping that around and making it
`FooType` and let the `protocol` get the unmodified name, but I'm just not sure
about it. It's really a matter of taste here.

## Same File or Multiple Files?

I'm usually a pretty big stickler for single type per file, but I'm not a big fan
of tons of tiny files that are a pain to navigate through in Xcode, so when the
protocol is small enough (< 5 things in it), I'll usually just leave it at the
top of the file that has the type which implements it. But sometimes I'll find
myself creating multiple protocols for a single type; one for its input related
methods and one for its output related methods, when that makes sense. Usually
when a type has methods that can be separated out that way it's because the type
is being used as a bridge between two layers of the app (e.g. between the UI and
the database or network) so when the object is used elsewhere there is only a
subset of its interface that is important to either side, at which point it makes
sense to have it subdivided in this way. And once that starts happening, it's
probably a good idea to start breaking the protocols and the type into separate
files, else it just gets too busy.

# Value Semantics vs. Reference Semantics
