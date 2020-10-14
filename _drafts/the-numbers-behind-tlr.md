---
layout: post
title:  "The numbers behind The Long Road"
author: James Lucas
tags: unity gamedesign
---

<img src="{{ site.baseurl }}/assets/images/posts/3/LongRoadJumpHD.gif" class="centered-full">\
Balancing games is difficult, but difficult problems are fun. In this post, I'll dig into some of the maths behind [The Long Road](https://www.glasstomegames.co.uk/home/the-long-road).

The Long Road (TLR) blends elements of roguelike games and classic text-adventure games. You play as a group of scavengers wandering a post-apocalyptic wasteland; collecting food, weapons, and more scavengers in a bid to survive. But, how long should you survive in the name of fun?

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

# A balancing act

There is a tension in TLR --- [a cursed problem](https://www.youtube.com/watch?v=8uE6-vIi1rQ) --- players want to survive as long as possible, but doing so renders the game boring. One popular way to solve this problem, is by injecting some randomness. Now players are a dice-roll away from thriving (or failing).

But how do we maintain this knife-edge? Give the player too much and their dice rolls become meaningless. Be too harsh and every run feels horribly unfair. To better understand how to successfully design TLR, we will need to dig into the numbers.

# The dynamics of scavenging

There are three resources in The Long Road: food, weapons, and scavengers. Scavengers eat your food to survive: more scavengers need more food and without it they die. Weapons enable combat options in conflicts --- outcomes are better with them than without... The game ends only when you run out of scavengers.

TLR is turn-based, allowing the player to choose one of four directions to move in during each turn. Each new tile you explore may give you some resources, a conflict, or nothing at all. At the end of the turn, your scavengers consume some of your food (or starve and die). There's a lot of randomness here, but we want to know how long the player should expect a game to last. Lets start out with some simple first steps.

## Wanderers

For now, forget about the randomness and look at a simplified problem. If scavengers found no resources, this is how long they would last on the long road.

Let $$X_t$$ be our food at the start of turn $$t$$, and $$Y_t$$ be our scavenger count. Each scavenger consumes $$\alpha > 0$$ food, so that $$X_{t+1} = \max(X_{t} - \alpha Y_{t}, 0)$$. Additionally, each excess unit of food that is consumed leads to a scavengers death: $$Y_{t+1} = Y_{t} + \min(0, (X_{t} - \alpha Y_{t}))$$.

This is already looking pretty complicated, but we can make better sense of this by picking some specific values. If we take $$\alpha = 0.5$$ for example, then in each turn we consume food equal to half of our scavengers. And if we have zero food left, then half of our scavengers die.

The quantity we care about here is how many turns we can last before we hit game over, i.e., the smallest $$t$$ with $$Y_{t} \leq 0$$. First, how many turns before we run out of food? This is given by $$X_{0} / (\alpha Y_{0})$$. After this point, the scavenger count evolves like,

$$Y_{t} = (1 - \alpha) Y_{t-1} = (1 - \alpha)^{t} Y_0.$$

Now notice that if $$\alpha \geq 1$$, then it takes only one turn for all of the scavengers to starve to death once the food has run out. If the scavengers ration better ($$\alpha < 1$$), then we need,


$$t > \frac{- \log(Y_0)}{\log (1 - \alpha)},$$

turns for all of the scavengers to starve ($$Y_t < 1$$). So the total number of turns is roughly,

$$\frac{X_{0}}{\alpha Y_0} + \frac{- \log(Y_0)}{\log (1 - \alpha)}.$$

<details>
<summary>Why <i>roughly</i>?</summary>
Remember that the food and scavenger counts are integers but we're ignoring rounding effects here for simplicity.
</details>

Looking at this quantity, we see that adding more scavengers decreases the total number of turns. While adding more food increases it. On the other hand, as scavengers get more greedy ($$\alpha$$ increases) then they last fewer turns.

## Scavengers

Let's make things slightly more interesting. Now our scavengers have learned that to survive they need to find more food, and friends. In each turn, food will increase by $$C > 0,$$ and scavengers will increase by $$K > 0$$. Our new dynamics are given by,

$$X_{t+1} = \max(X_{t} - \alpha Y_{t}, 0) + C,$$

and,

$$Y_{t+1} = Y_{t} + \min(0, (X_{t} - \alpha Y_{t})) + K.$$


How many turns can the scavengers last now?

First, notice that if $$C > \alpha Y_t$$ then the scavengers gain food on each turn. But the number of scavengers is always increasing by $$K$$. The food will stabilize when $$\alpha Y_t = C$$, and the scavengers will stabilize when $$\alpha Y_t - X_t = K$$. Solving these together, we get,

$$X_t = C - K,\:\:\: Y_t = C/\alpha.$$

So long as $\alpha < 1$, each turn some scavengers die and are replaced anew. But sadism aside, this isn't very exciting for the player --- they can never lose! How can we fix this?

## Scavengers scavenge, sometimes



## Biomes

In addition to these three resources, TLR has three different biomes to explore. Each has their own set of rules governing how the player's resources change. Forests provide the player with more food but fewer weapons. The wastelands provide very few resources but are generally low-risk. And the cities provide high resources, but have a high chance of dangerous conflicts.


