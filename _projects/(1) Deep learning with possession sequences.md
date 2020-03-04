---
name: Deep learning with possession sequences
tools: [R, keras, tensorflow]
image: /Barca0809.jpg
description: A data-driven analysis of Barcelona's historic 08/09 season.
---

## Preface

I'm going to try to explain this in the most intuitive/least mathsy way I can, and do so with reference to specific football examples. Sometimes I'll veer into technical jargon and unnecessarily dense explanation, other times I'll omit detail you're interested in.

If you want me to clarify anything or you think I've got something wrong or you think I've missed something out, please let me know via Twitter or email (details in the page footer) & I'll try my best to help out.

Whenever there are necessarily technical sections, I'll try my best to explain them in football-relevant, intuitive terms. Hopefully you'll get something out of it even if you don't fully understand every single aspect of the methodology.

Also, this is going to be long. Brace yourselves. Hopefully this table of contents can help you to skip bits you don't care about.

### Contents
  1. [Intro to possession models](#introduction-to-possession-models)
     - [xGChain/xGBuildup](#xgchain-and-xgbuildup)
     - [Location-based models](#location-based-models)
     - [Context-based models](#context-based-models)
  2. [Model overview](#model-overview)
  3. [Analysing Barca](#analysing-barca)

## Introduction to possession models

The basic goal of possession models is to ascribe some 'value' to every single action in a possession. The goalscorer is not solely responsible for a goal being scored, nor is the assister -- all of the actions that led up to the ball crossing the line had some impact on the outcome. 

Additionally, there are really valuable actions that don't lead to shots (say a through ball that the keeper just wins the race for). As anyone who plays fantasy football can attest, all assists are not created equal. Possession models aim to build up a framework that allows you to ask exactly how valuable a given action was to a team's ability to score.

Here's a little imaginary sequence I'll refer to throughout this introduction:

**The keeper rolls it out to a centre back, the centre back pings a 50 yard diagonal to a winger, the winger beats a defender and whips in a cross and the striker side-foots in from 6 yards.**

Traditionally, you'd record the fact that the striker scored, the winger assisted, the keeper & the centre back completed a pass each -- that would be about it. Maybe you'd get fancy and compute the xG associated with the shot, or record a successful *long* pass for the centre back. Those basic stats don't really tell you about how the goal came about, or how you might stop it from coming about if you face this team in the future, or how helpful that centre back might be to your buildup play if you're looking to sign one.

This notion that we shouldn't reward *only* goalscorers & assisters for goals has led to some increasingly involved ways of using event data to attach a value to each action in a possession sequence -- what follows is my take on how you might do this for a single season's worth of data.

First, though, I'll give a brief overview of some of the existing models for valuing actions with event data. Because I don't have access to any tracking data, I'll ignore tracking data-based approaches such as Liverpool's expected possession value model (see the Tim Waskett RI christmas lecture for a hint at what they're doing) or [this paper](http://www.sloansportsconference.com/content/decomposing-the-immeasurable-sport-a-deep-learning-expected-possession-value-framework-for-soccer/) from the 2019 Sloan conference.

#### xGChain and xGBuildup

As far as I know, the earliest and one of the most simple ways of doing this was Thom Lawrence's [xGChain/xGBuildup](https://statsbomb.com/2018/08/introducing-xgchain-and-xgbuildup/). To compute the xGC for a player, you just add up the xG values of all possessions the player was involved in. To get to xGB, you exclude the xGs of all shots taken or assisted by the player. This scheme gives equal weight to every action in the sequence -- the keeper's roll out is given as much value as the 50 yard ping. Despite being so simple, the results are utterly sensible & useful for gauging the importance of a player to a team's attacking play. As always, though, it's interesting to try to extend this simple approach and see what happens when we get a bit more granular.

#### Location-based models

The most obvious limitation of xGC/xGB is its assumption that all actions in a possession are created equal. There are a few ways to deal with this. The first way is to use locations on the pitch to guess at the value of an action -- in general, you'd expect actions that get your team higher up the pitch & closer to the goal to be more valuable. You might then credit the CB in the recurring example with having progressed the ball from a low-value area to a high-value area, and likewise with the winger for his cross. This is broadly the approach taken by both Nils Mackay in his [possession-based model](https://mackayanalytics.nl/2016/11/11/what-is-a-possession-based-model-and-why-does-it-matter/) (which is presumably pretty similar to the one Opta are using currently?) & Karun Singh in his extremely neat [expected threat (xT) model](https://karun.in/blog/expected-threat.html).

Here's an image I've lifted from Nils Mackay's blog post. White locations are the most dangerous places to have possession, blue are the least. There'll be one of these later on as a sanity check on my own model.

![alt text](https://github.com/anenglishgoat/anenglishgoat.github.io/raw/master/mackay.png "Nils Mackay's possession value")


For what it's worth, I think these two models are probably the most sensible out there in terms of appropriate levels of granularity & computational burden. If you're looking to implement this kind of possession value model in your own analytics work, these two approaches are likely to be completely fine. They are essentially special cases of the work I'm going to outline below. Something that would be really interesting (which I haven't done) is an ablation study -- can I remove some of the more complex elements from my model and still achieve similar performance? I suspect that the answer is an emphatic 'yes', but I'll leave that for future work.

One fundamental assumption of these location-based models is the Markov assumption. That may sound fancy, but in this context it just means that the value of a position on the pitch is independent of what has already happened in the possession, and it's static (e.g., it's always equally as valuable to be in the right half-space regardless of the situation). Think of it as the game being played on top of the blue & white image above. Every time I make an action, it's value is computed as the difference in values between the start and end locations.

In our recurring example, this would correspond to assuming that the value of the winger's cross is not changed by the fact that it came from a long diagonal from the centre back rather than after a prolonged period of build up.

This assumption is probably fair if we also know the positions of all the other players on the pitch (in that case we'd have a different blue & white picture for every configuration of players, which would be the same whatever had happened before) -- Markov models are probably valid if we've got tracking data. 

Maybe the best way to think about this is in terms of pictures. If I give you a static image of a match situation, like this one from [Last Row View](https://twitter.com/lastrowview) (rise up shepherd and follow!), you can give me a pretty decent estimate of how dangerous the situation is. The small black dot is the ball.

![alt text](https://github.com/anenglishgoat/anenglishgoat.github.io/raw/master/LRV_im.JPG "A static match situation")


Ideally you'd want to know the velocities of the players too, but I'd say you're getting pretty close to a Markov assumption being valid -- given the information you've got about this particular moment in time, it doesn't matter how I got here, the threat is the same.

But I don't have tracking data. The only thing that shows up in the event data is that number 37 (on the ball) has just dribbled past an opposition player in that location. There are lots of possible situations in which this might happen and the threat is different each time -- maybe it's come from a counter, he's dribbled past the last man and is now clean through on goal; maybe the white team are sat in a low block, the striker has come to press, and number 37 has skipped past him but the yellows are not really any closer to scoring. In this case, maybe information about what has happened previously in the possession is helpful for us to get a better idea about how dangerous the current situation is.

#### Context-based models

This is where the next pair of models come in. The model from [Thom Lawrence's Statsbomb conference talk](https://www.youtube.com/watch?v=5j-Ij5_3Cs8) (watch!) and Tom Decroos et al.'s [VAEP](https://www.kdd.org/kdd2019/accepted-papers/view/actions-speak-louder-than-goals-valuing-player-actions-in-soccer) (read!) model both use past actions to compute the value of an action within a possession sequence.

The idea is that this context can help to fill in at least some of the gaps that result from not having tracking data. If we know that number 37's dribble in the above example came after lots of passing around the defence and a few probing forward passes that came straight back out, there's a good chance it's less dangerous than if it follows a long ball from a defender. We're not getting to the same level of richness as the tracking data, but we're at least closer.

Since these two models are the most similar to the one I'll use to analyse the Barca data below, I'll spend a bit of time going over how they work.

##### Notation

I'm going to have to use some shorthand here to stop things getting too cluttered. Whenever I write $P(\textrm{a thing})$, I mean the  probability that $\textrm{a thing}$ happens.


##### VAEP

In the VAEP model, every action has an *offensive* and a *defensive* value. The offensive value is g


## Model overview

## Analysing Barca

