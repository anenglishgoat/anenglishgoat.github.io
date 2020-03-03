---
name: Deep learning with possession sequences
tools: [R, keras, tensorflow]
image: /Barca0809.jpg
description: A data-driven analysis of Barcelona's historic 08/09 season.
---

# Introduction to possession models

The basic goal of possession models is to ascribe some 'value' to every single action in a possession. The goalscorer is not solely responsible for a goal being scored -- all of the actions that led up to the ball crossing the line had some impact on the outcome. Maybe the keeper rolled it out to a centre back, the centre back pinged a 50 yard diagonal to a winger, the winger beat a defender and whipped in a cross and the striker side footed in from 6 yards. Traditionally, you'd record the fact that the striker scored, the winger assisted, the keeper & the centre back completed a pass each, and that would be about it -- maybe you'd get fancy and compute the xG associated with the shot, or record a successful *long* pass for the centre back. Those basic stats don't really tell you about how the goal came about, or how you might stop it from coming about if you face this team in the future, or how helpful that centre back might be to your buildup play if you're looking to sign one.

This notion that we shouldn't reward *only* goalscorers & assisters for goals has led to some increasingly involved ways of using event data to attach a value to each action in a possession sequence -- what follows is my take on how you might do this for a single season's worth of data.

First, though, I'll give a brief overview of some of the existing models for valuing actions with event data. I'll ignore tracking data approaches, such as Liverpool's expected possession value model (see the Tim Waskett RI christmas lecture for a hint at what they're doing) or [this paper](http://www.sloansportsconference.com/content/decomposing-the-immeasurable-sport-a-deep-learning-expected-possession-value-framework-for-soccer/) from the 2019 Sloan conference.

As far as I know, the earliest and one of the most simple ways of doing this was Thom Lawrence's [xGChain/xGBuildup](https://statsbomb.com/2018/08/introducing-xgchain-and-xgbuildup/). 