---
layout: post
title: "18-Hour Hackathon: Making a CoffeeScript Solitaire"
date: 2011-11-07 21:38
comments: true
categories:
atom_id: http://opinionated-programmer.com/?p=930
---

The making of [solitr.com][] (source at [github.com/joliss/solitr][]), a
CoffeeScript solitaire, in one 18-hour hackathon:

## Hackathon Diary

9:15 am: *Shreeeek-shreeek-shreeeeeeeek!*
*Get-up-get-dressed-have-breakfast-coffee-go-go-go.*

9:50 am: Sit down at laptop. What do I name my project? What CSS syntax
do I use? What CSS libraries?

10:45 am: Too much reading, let’s just decide and keep moving! “rails
new solitaire” (it’s 100% static, but I need Rails’s asset handling with
Sprockets), drop in yui-reset. Make controller, route root to
play\#index.

11:10 am: Alright, let’s code up a data model (no ORM, just JS classes).
Cards first. Then a GameState class to hold the card positions. Lots of
googling for CoffeeScript and JavaScript basics in the process.

12:15 pm:
*Microwave-yesterday’s-food-have-lunch-coffee-go-go-go-keep-moving!*

12:35 pm: Finish up data model. Make presentation layer. Code up CSS for
makeshift card rendering. Drop in [Underscore.js][] for more coding
sanity.

4:00 pm: Work-in-progress (completely static): **[1db97387][]**. Next up
is event-handling. Read up on jQuery’s [.on][]. Figure out how to drop
in jQuery 1.7 without making frickin jquery-rails unhappy with its
outdated jQuery. Code up some easy events.

6:30 pm:
*Put-pizza-in-the-oven-go-jogging-yes-it’s-cold-but-you-gotta-keep-running-for-30-minutes-or-your-brain-will-be-fried-by-midnight.
Get-back-exhausted-*pizza-ready-*house-not-burnt-down-very-good-let’s-eat-eat-eat.
Coffee. Back-to-coding-no-time-to-waste!*

7:40 pm: Brain in reasonable condition. 7.5 hours to go, still on
schedule I think. Time to start reading about [dragging][] and
[dropping][]. Painful stuff as expected.

10:35 pm: Houston! Problem! The way I set up the DOM structure is not
looking to work out with drag’n’drop. Check in work-in-progress and
revert to 7:40 pm instantly. Three hours lost, I think I’m off schedule
now. Let’s try again, using HTML/CSS only as a dumb rendering canvas,
with less DOM structure.

11:05 pm: Yup, looking better. Now for dragging and dropping, fingers
crossed. Time’s tight.

2:25 am: First working version: **[c475ae9e][]**. Tired, but brain’s
still producing reasonable code. Time to drop in a pretty card deck with
CSS sprite handling, test, fix undo handling, add double-clicking, “you
win” message, “new game” buttons, make production environment work, test
some more.

5:35 am: It got late, but I’m done: **[18961aa1][]**. More than 2h over
budget, my 18h hackathon has become a 20h hackathon. And dragging of
multiple cards has a rendering issue. Still, I’m very happy with the
result. Brain is fried, off to bed.

(And the next day I set up [solitr.com][] and pushed the code to
[github.com/joliss/solitr][]. But I’ll blog about the whole
domain/DNS/deployment thing some other time in detail.)

## Hackathon Retrospective

Oh my gosh. This was *so* much fun. I’ll definitely do hackathons again.
Being able to whip up a webapp on a whim is such a good feeling. Also, I
picked up a lot of CoffeeScript and jQuery that I wouldn’t have been
exposed to otherwise.

I’ve found that by doing a hackathon, I’m forced to just keep working,
so I’m producing code all day. I’ve rarely gotten so much output from my
brain in one day.

Pulling 18 hour work-days, of course, is rather not sustainable, let alone the
20 real hours that it turned into. But when you’re writing an app from scratch,
it’s only satisfying if at the end you have it actually working, so
compromising on scope is not really an option. I think the better alternative
is trying to get faster at creating new apps. Now that I know my way around the
basics of CoffeeScript and jQuery, I think I might be able to do the same thing
in 16 hours, which seems much more reasonable.

Next time I’ll try to get started faster at the beginning (less agonizing about
things like the name or the CSS library — it’s all trivial to change later),
and I’ll also try to be more disciplined in the last 2 hours to not let
non-essential features creep in.

  [solitr.com]: http://www.solitr.com/
  [github.com/joliss/solitr]: https://github.com/joliss/solitr
  [Underscore.js]: http://documentcloud.github.com/underscore/
  [1db97387]: http://demo.solitr.com/1db973871d8bf7e3c0ddbab2950543df5b1ca296/
  [.on]: http://api.jquery.com/on/
  [dragging]: http://jqueryui.com/demos/draggable/
  [dropping]: http://jqueryui.com/demos/droppable/
  [c475ae9e]: http://demo.solitr.com/c475ae9e78f8f75ed64a06df19cc53b9ff473647/
  [18961aa1]: http://demo.solitr.com/18961aa1cf7be72176c5ba1ad4d3174fbca49eff/

