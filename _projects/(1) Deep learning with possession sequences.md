---
name: Deep learning with possession sequences
tools: [R, keras, tensorflow]
image: /Barca0809.jpg
description: A data-driven analysis of Barcelona's historic 08/09 season.
---

# Introduction to possession models

The basic goal of possession models is to ascribe some 'value' to every single action in a possession. The goalscorer is not solely responsible for a goal being scored -- all of the actions that led up to the ball crossing the line had some impact on the outcome. Here's a little imaginary sequence I'll refer to throughout this introduction:

**The keeper rolls it out to a centre back, the centre back pings a 50 yard diagonal to a winger, the winger beats a defender and whips in a cross and the striker side-foots in from 6 yards.**

Traditionally, you'd record the fact that the striker scored, the winger assisted, the keeper & the centre back completed a pass each -- that would be about it. Maybe you'd get fancy and compute the xG associated with the shot, or record a successful *long* pass for the centre back. Those basic stats don't really tell you about how the goal came about, or how you might stop it from coming about if you face this team in the future, or how helpful that centre back might be to your buildup play if you're looking to sign one.

This notion that we shouldn't reward *only* goalscorers & assisters for goals has led to some increasingly involved ways of using event data to attach a value to each action in a possession sequence -- what follows is my take on how you might do this for a single season's worth of data.

First, though, I'll give a brief overview of some of the existing models for valuing actions with event data. I'll ignore tracking data approaches, such as Liverpool's expected possession value model (see the Tim Waskett RI christmas lecture for a hint at what they're doing) or [this paper](http://www.sloansportsconference.com/content/decomposing-the-immeasurable-sport-a-deep-learning-expected-possession-value-framework-for-soccer/) from the 2019 Sloan conference.

As far as I know, the earliest and one of the most simple ways of doing this was Thom Lawrence's [xGChain/xGBuildup](https://statsbomb.com/2018/08/introducing-xgchain-and-xgbuildup/). To compute the xGC for a player, you just add up the xG values of all possessions the player was involved in. To get to xGB, you exclude the xGs of all shots taken or assisted by the player. This scheme gives equal weight to every action in the sequence -- the keeper's roll out is given as much value as the 50 yard ping. Despite being so simple, the results are utterly sensible & useful for gauging the importance of a player to a team's attacking play. As always, though, it's interesting to try to extend this simple approach and see what happens when we get a bit more granular.

The most obvious limitation of xGC/xGB is its assumption that all actions in a possession are created equal. There are a few ways to deal with this. The first way is to use locations on the pitch to guess at the value of an action -- in general, you'd expect actions higher up the pitch & closer to the goal to be more valuable. You might then credit the CB in the recurring example with having progressed the ball from a low-value area to a high-value area, and likewise with the winger for his cross. This is broadly the approach taken by both Nils Mackay in his [xG-added model](https://mackayanalytics.nl/2016/11/11/what-is-a-possession-based-model-and-why-does-it-matter/) & Karun Singh in his [expected threat (xT) model](https://karun.in/blog/expected-threat.html). 

For what it's worth, I think these two models are probably the most sensible out there in terms of appropriate levels of granularity & computational burden. If you're looking to implement this kind of possession value model in your own analytics work, these two approaches are likely to be completely fine. They are essentially special cases of the work I'm going to outline below. Something that would be really interesting (which I haven't done) is an ablation study -- can I remove some of the more complex elements from my model and still achieve similar performance? I suspect that the answer is an emphatic 'yes', but I'll leave that for future work.



