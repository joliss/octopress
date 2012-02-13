---
layout: post
title: "How I hand-rolled MVC for a JavaScript game (and it was awesome)"
date: 2012-02-13 12:00
comments: true
categories: javascript mvc
---

*In which*

* *I sing the praises of frameworks like Ember.js,*
* *explain why they don't work so well for games, and*
* *demonstrate how to structure any JavaScript app (game or not, framework or
  not) around MVC for cleaner code.*

I recently decided to improve the [Solitr](http://www.solitr.com/) game
([GitHub](https://github.com/joliss/solitr)) I had hacked together in a
one-day hackathon ([blogged about
here](http://www.solitr.com/blog/2011/11/18-hour-hackathon-making-a-coffeescript-solitaire/)).
The code had turned out rather messy, so I went for a rewrite with better
design, only copying bits and pieces from the first implementation.

A plethora of JavaScript MVC frameworks have cropped up since 2011 --
[Backbone](http://documentcloud.github.com/backbone/),
[Spine](http://spinejs.com/), [Knockout](http://knockoutjs.com/),
[Ember](http://emberjs.com/), <a
href="http://batmanjs.org/">Batman</a>, and
[Serenade](https://github.com/elabs/serenade.js) come to mind. I had used Backbone
before, but I had heard interesting things about Ember, so I decided to give
that a spin. I am rather enthused:

## Why Ember.js Rocks

Probably the most important feature in Ember is **attribute binding**. It
allows attribute changes to automatically propagate through all layers of your
application. For example, say you change `firstName` on your model, then the
`firstName` attributes on your controller and view class are updated (because
they are "bound" to `model.firstName`), and the `fullName` helper attribute on
your view class gets recalculated as well, because it in turn is "bound" to
`view.firstName`.

This is the awesomest thing since sliced bread. It means that when your
controller responds to an event, it only needs to update the model and every
piece of your app that (directly on indirectly) depends on it is notified.

Ember's **in-place template updating** deserves special attention too, and
coming from server-side templating languages, the significance of this feature
wasn't obvious to me at first: When the `fullName` attribute in the previous
example is updated, Ember's template engine will change the attribute inside
the template live in the DOM, and not just re-render the entire template.
That's important because otherwise you lose all GUI state (such as collapsible
elements, or cursors in text fields) whenever a single attribute inside a
template changes. I learned this the hard way with a Backbone application I
worked on: Having to re-render views just because a counter increases is
bothersome for small page elements, but once you get to reloading an entire
pane, it becomes a usability issue, and one that's rather hard to fix.

In other words, unlike on the server side, it's useful for JavaScript
templating engines to be more intelligent than just spitting out a chunk of
HTML.

So attribute binding and template updating are the two things I recommend you
pay attention to when choosing an MVC framework.

## Enter Games: On States and State Transitions

So I started to rewrite Solitr with Ember.js. A day or so into the rewrite
however, I was beginning to question whether Ember's attribute-binding approach
was right for a game. Let me explain why.

When a card moves to a new place, it needs to be animated. So you can't just
update the `left` and `top` CSS properties when the game state (and hence the
location of a card) changes. My first instict was to simply slide the
cards into place whenever the position changes, either by catching the
changes to the position attribute and wrapping it in a call to
jQuery's [.animate()](http://api.jquery.com/animate/), or using the CSS
[transition](https://developer.mozilla.org/en/CSS/transition) property.
However, this doesn't *quite* work: The type of animation you need depends on
the last action: For instance, turning over three cards on the stock, playing a
card you double-clicked, or snapping a card into place after it has been
drag-and-dropped, all require different sliding speeds and easing functions.

Of course you can hack the animation code so that it infers the right type of
animation from the difference between the current and the previous game state.
This might works most of the time, but besides being ugly, there are cases
where it's not possible to know whether a given state change came about
through, say, double-click, drag-and-drop, or undo (all of which are animated
differently).

In this situation, compromising on the quality of the animations is a
no-brainer for a regular web application. But for a game, **animations should
be first-class citizens.** Since they are so ubiquitous in a game (even a
simple one like solitaire), treating them as less than first-class citizen will
yield hundreds of lines of unmaintainable animation spaghetti code.

So how do you make animations first-class citizens? Easy enough: You do not
update the game state and then ask the controller to update the GUI according
to the new state. Instead, you send a Command object ([as described in
GoF's Design Patterns](http://www.ece.ubc.ca/~matei/EECE417/command-pattern.pdf)) to the game
state so that it can update itself, and then send the same Command object to
the animation method so it can update the GUI *based on the type of command*.

In other words, the application is now fundamentally structured around **state
transitions** (commands, that is) rather than **states**.

This approach is probably untypical for "regular" web apps, and frameworks like
Ember.js discourage it by exposing state and hiding state transitions behind
the auto-updating mechanism for object attributes. Besides animations, the only use case I can
think of is when you absolutely need to build an undo stack: With your code
structured around state transitions, you can easily use Command objects for the
undo stack, rather than ad-hoc closures (or clones of the model object). Other
than that, I am not sure if for a non-game app that doesn't require
sophisticated animations, this is worth the effort. 

However, for game code, I have found that this pattern works very well, and it
has turned out to be very maintainable so far.

## Hand-Rolling MVC

As I structured my app around Commands, my reliance on Ember's features
started to disappear, and eventually I was left with Ember's slightly
cumbersome attribute accessor syntax (`card.get('rank')` instead of
`card.rank`) with no benefit from it, as all my attributes had become
independent, and no auto-updating between objects ("binding" in Ember
parlance) was going on anymore.

The only thing Ember was doing for me was templating with Handlebars, but since
all I render are cards (essentially `<div class="card" id="..."></div>`), I
felt safe pulling these one-liners into the JavaScript code without sacrificing
maintainability.

So I removed the dependence on the Ember.js library. What was left was an
application handsomely structured around the MVC pattern, and compared to the
initial version I hacked together, the logical separation into model and
presentation code is the biggest single design improvement.

## A Look at the Code

The application essentially consists of two files,
[models.js.coffee](https://github.com/joliss/solitr/blob/master/app/assets/javascripts/models.js.coffee)
and
[controllers.js.coffee](https://github.com/joliss/solitr/blob/master/app/assets/javascripts/controllers.js.coffee)
(written in CoffeeScript).

I haven't found it necessary to derive either models or controllers from a
common base class.

### Models

The models hold the game state and implement the game (domain) logic. I
deliberatedly kept the models isolated from the rest of the application, so the
code doesn't know about the controllers or views, and also doesn't need a DOM.
This has made the code much more maintainable. I'm also hoping that it will
help me when I write tests, perhaps with Mocha. <!-- XXX
https://github.com/jfirebaugh/matcha ? -->

#### Card Model

The Card model is very simple. It only holds three attributes -- `rank`,
`suit`, and `id` -- and no logic at all. Whether it is face-up or face-down is
inferable from its place in the game (stock cards are always face-down, waste
cards are always face-up, etc.), so that is something that is determined by the
controller that renders it, not stored by the model.

```coffeescript
_nextId = 0

class App.Models.Card
  constructor: (@rank, @suit) ->
    @id = "id#{_nextId++}"
```

#### Game State Model

The game state model, on the other hand, is rather more complicated. It holds
several arrays of cards (stock, waste, tableau piles, foundations), and encodes
the rules of the game (the domain logic, that is). Here is a slightly
simplified version:

```coffeescript
class App.Models.KlondikeTurnThree
  cardsToTurn: 3
  numberOfFoundations: 4
  numberOfTableaux: 7

  # Initialization
  constructor: -> # initialize deck and arrays
  createDeck: -> # create all the cards for the deck
  deal: -> # set up initial layout

  # Rules
  foundationAccepts: (foundationIndex, cards) ->
  tableauPileAccepts: (tableauPileIndex, cards) ->
  isMovable: (card) ->

  # Commands
  executeCommand: (cmd) ->
    # The Big Method that parses and executes commands,
    # thereby managing all state transitions

  # Helpers
  nextAutoCommand: ->
    # Return command to auto-play and auto-flip
    # cards in some situations
  ... other helpers ...
```

While the model implements rule logic (with methods like `foundationAccepts`),
it does not enforce it. The controller has to check whether a given action is
legal in order to give proper user feedback. It then seemed redundant to
implement another such check in the game state model.

For simplicity's sake, I decided not to let model classes proliferate: The piles
that make up the game state model (stock, waste, etc.) are dumb arrays, not
models on their own. Any logic that might conceivably be attached to those
piles is moved up into the all-knowing game state model.

### Controllers

Paralleling the models, there is a simple Card controller performing basic rendering
duties, and a game controller (App.Controllers.KlondikeTurnThree) ordering the
cards around.

#### Card Controller

``` coffeescript
class App.Controllers.Card
  size: {width: 79, height: 123}
  element: null

  constructor: (@model) ->

  # Insert / remove from DOM
  appendTo: (rootElement) ->
  destroy: ->

  # Move around
  setRestingPosition: (position) ->
  jumpToRestingPosition: ->
  animateToRestingPosition: (animationOptions) ->
```

#### Game Controller

``` coffeescript
class App.Controllers.KlondikeTurnThree
  model: null

  constructor: ->

  newGame: ->

  processCommand: (cmd) ->
    # Call @model.executeCommand and @animateCards
  animateCards: (cmd) ->
    # Paralleling executeCommand in the model, this method
    # animates the GUI

  # Housekeeping
  calculateGeometry: () ->
  appendBaseElements: () ->
  registerEventHandlers: ->
  removeEventHandlers: ->

  # The actual event handlers
  turnStock: =>
  redeal: =>
  ...

  # And tons of helper functions dealing with minutiae like
  # drop zone geometry for drag'n'drop ...
  ...
```

To be honest, I'm finding that my game controller is becoming a sprawling
assortment of semi-related methods. I think this in part due to GUIs being
messy business, and in part due to suboptimal design.  Eventually I will have
to split it into more classes, but for now it actually works quite well.

### Views and Templates

The view classes and templates that most frameworks expect you to create have
been folded into the Controller classes.  There is the occasional `$('<div
class="button">...</div>')`, and the cards are rendered by programmatically
by creating DOM nodes:

```coffeescript
class App.Controllers.Card
  appendTo: (rootElement) ->
    @element = document.createElement('div')
    @element.className = 'card'
    @element.id = @model.id
    $(@element).css(@size)
    $(rootElement).append(@element)
```

So far I haven't needed to separate these from the controllers because they are
so miniscule. (I am sure that this will not scale though, and if you are using
an MVC framework that comes with templating functionality, you should
definitely use it from the get-go.)

## Conclusions

* Ember.js rocks because it can propagate (bind) attributes between objects,
  and it auto-updates templates live without rerendering.

* But attribute propagation is suboptimal for games, since animation requires
  that you know about state transitions, not just state.

* For any app, it greatly helps the design to follow MVC:

  1. Create isolated model classes storing application state and implementing
     domain logic.
  2. Create controller classes that talk to the models and the DOM,
     implementing the GUI logic.
  3. Factor templates out of the controllers.
  4. Factor helper methods for the templates into view classes.
