---
layout: post
title:  "A short stop on The Long Road"
author: James Lucas & Rio Kisjantoro
tags: gamejam unity
---

<img src="{{ site.baseurl }}/assets/images/posts/3/LongRoadJumpHD.gif" class="centered-full">\
[The Long Road](https://www.glasstomegames.co.uk/home/the-long-road)

Last weekend we participated in [Mini Jam #63](https://itch.io/jam/mini-jam-63-future). Mini Jam is a weekend-long game jam that runs every two weeks and features a fun twist. All games are given a theme and an additional limitation they must adhere to. This time, the theme was "Future" and the limitation was that all games must use only colours in a [fixed colour palette](https://lospec.com/palette-list/arcade-standard-29).

The Long Road is a turn-based survival game. The player is traversing across an ever-changing, post-apocalyptic world and is confronted by random text-based events. The player must lead their group of survivors and make tough decisions as they fend off bandits, starvation and so much more. 

- 48 hours for development
- 143 Commits, ~2300 LOC
- ~1000 words of story content
- 1 instance of funky beats


# Designing The Long Road

We begin each of our game jams the same way: with a short call to bounce around ideas.

A typical call normally has a few rejected ideas or concepts before we land on a genre that we like and then we build on that. Rio suggested some kind of survival game based on a grid that populates as you move around the world. The "Future" theme led us to create a world that you don't want to settle down in; a world where you have to keep on moving to survive. Once we had this core idea, we started building on it by suggesting gameplay loops and mechanisms for the player.

Turn-based gameplay would lend itself well to the grid style that we were already proficient in thanks to Sprawl. From there, we figured out that we probably need resources. Food came first and naturally after that population or group size came into play. We knew that we wanted a simple and engaging game, so we restricted it to 3 resources and settled on weapons for the last one. This allowed us to introduce conflict and allow the player to deal with it in a few different ways. 

At this point, we were trying to find the essence of the game. What made the player care about the world and their character? After recently picking up ["As Far As The Eye"](https://store.steampowered.com/app/1119700/As_Far_As_The_Eye/) we were quite fond of their option system when encountering new tribes. What if we mixed in some old school text-based adventure and created a large directory of events that could be randomly chosen from when the player goes to a tile?


## Scoping the game

Scoping for a game jam is always tricky. We tend to overshoot slightly and reign things in as the days go by --- this time was a little different though as James would be working with a new (noisy) family member in the house. It was hard to tell how much time we would have available, so we tried to think about minimum viable products that we could shoot for from the start.

# Obeying the palette limitation

We love 3D games, and enjoy being one of typically a few developers who submit 3D games to jams. But making a 3D game where every pixel on the screen must match a colour palette is hard! We came up with a fairly nice solution, using a cel shader and mapping each colour in the palette to another darker shade to simulate lighting.

Here is the colour palette we needed to use, with the shadow mapping displayed underneath.

<img src="{{ site.baseurl }}/assets/images/posts/3/Palette_long_road.png" class="centered">
<img src="{{ site.baseurl }}/assets/images/posts/3/Palette_shadows_long_road.png" class="centered">

The final look we achieved with this approach was much nicer than we expected! (Though admittedly, forcing a hard transition between the tones does leave some noticeable artifacts when the light direction changes.)

<img src="{{ site.baseurl }}/assets/images/posts/3/CelShading.gif" class="centered">\
Our two-tone cel shader in action


# Coding the game

We'll break down the development of the codebase chronologically. 

We managed to spend about 12 hours over the weekend working on the code (thank you, Emily!). This was more than we expected and enabled us to complete almost everything on our wish list.

## Hours 0-2 (Saturday)

<img src="{{ site.baseurl }}/assets/images/posts/3/Grid-Plane-Movement.gif" class="wrap-left">

James spent the first couple hours working on the animated grid movement. This consisted of an input system that detected key presses and told the grid (and camera) to move. These move commands were processed in a queue to keep the movement smooth.

Most of these two hours were spent trying to figure out how to mutate the data stored in the grid. We can add/delete rows or columns of tiles on any of the four sides. The code to do this ended up being a little more involved than first anticipated...

## Hours 3-6 (Saturday)

The Long Road features a procedurally generated world, interactive random encounters, and a resource management system dictating your survival. All of these components interact with each other and so the game required careful backend design to prevent spaghetti code hell.

The core of the game is built using a central message bus. Systems in the game can send and subscribe to events in the message bus to interact with each other. The game is turn-based and James designed the code so that the states making up a turn serve as the heartbeat. When the turn starts, or action is taken, or conflicts are triggered/resolved, an event is fired so that other systems can process the data and respond accordingly.

## Hours 6-9 (Sunday)

Now we could start working on content. By now Rio had built up most of the UI and a fair chunk of the models for the tiles we would be using. I got to work developing the tile event system.

<img src="{{ site.baseurl }}/assets/images/posts/3/LongRoadEarlyConflicts.gif" class="wrap-right">

Each tile should be given a random event to be triggered when the player visited it. Some events do nothing, some award a small number of resources (Food, Scavengers, or Weapons), and some trigger interactive conflicts. I started with just the resource awarding events and finally the game started taking its shape.

You can also see a placeholder player model here, a small sphere that moves with the grid.

## Hours 9-12 (Sunday)

By now, James had convinced his wife, Emily, to help work on the writing and she had produced several fun interactive events to be placed in the game. With most of the tile event system in place, it was easy to plug in the interactive events. Hooking up the UI elements for this was also fairly painless and didn't take too long.

The last few touches were focused on improving the visuals and tweaking the RNG to be a bit more balanced. Unfortunately, we didn't have enough time to do this well and the version we submitted is still a little punishing.

# Debrief

We're happy with how much we achieved in this short amount of time. The most prominent feedback we've received is that the RNG elements needs to be rebalanced. We wholeheartedly agree with that, and expect that this would be an ongoing challenge in development. Nonetheless, the game is a lot of fun to play and [you can play it for free](https://necrosaint.itch.io/the-long-road)!

<div class="centered">
<iframe width="560" height="315" src="https://www.youtube.com/embed/W9NBeaDsniY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# Looking to the future

We love how The Long Road turned out and see a bright future for the game. We think that adding some dynamic story telling to the game and possibly some roguelike elements would help it shine. To that end, one of our good friends is an excellent writer and has agreed to spend a couple of months fleshing things out.

We're still very much focused on developing [Sprawl](https://www.glasstomegames.co.uk/home/sprawl), but plan to pick up development on this game in the future (and may dabble before then...). So keep your eye open for more updates soon!
