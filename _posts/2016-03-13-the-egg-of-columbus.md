---
layout:     post
title:      Turn Over the Table
date:       2016-03-14
summary:    Sometimes the most effective way to solve a problem is to redefine the problem itself
tags:       engineering, effectiveness
image:      egg-of-columbus.jpg
published:  true
---

When you're weighing solutions to a problem, remember to also consider the effect of removing or relaxing requirements and constraints.


## Beware of Implied Constraints

The parable of [The Egg of Columbus](https://en.wikipedia.org/wiki/Egg_of_Columbus) always stuck in my mind since I was first told it as a child, and I see multiple lessons that can be taken from it. 

Columbus challenges the Spanish Nobles to stand an Egg on it's tip, and they all fail because they assume an implicit constraint that the egg should not be damaged. 


I find this useful to bare in mind in engineering, as we often assume implied constraints on how we should design and build systems. These constraints then lead to unnecessarily complicated solutions.

Stand back and consider what implied constraints you're applying, and whether they're actually valid. Sometimes they are, sometimes they aren't.

### Cheating?

The first time I remember using this, was when in my early teens, I was in a school-sponsored engineering contest. One of the challenges the teams were given was to build a lego truck that was able to carry some steel bars down a steep slope, crash into a barrier at the bottom, and survive without any bars spilling.

All the other teams focused on making their trucks as strong and stable as possible, so that it would survive the collision.

_Our team just jerry-rigged brakes for the wheels, and won the prize._


## Requirements are Rarely All Set in Stone

Once you've spent some time analyzing solutions for a problem, you'll often have insights that the originator of the requirements did not. So if you think that removal or weakening of a requirement will reduce the cost or risk, than have confidence and try it.

eg. One requirement is 10% of the value, but is the highest risk due to it being a new area of development, and therefore the estimate is in an uncertain range of 20-60% of the total effort. Is it worth slowing the rollout of the rest of the value, or even the potential for a cancelled project, for this one requirement?

When looking at solutions to a problem, try to step back and see the choice in terms of the ___best combination of value, cost and risk___. Engineers should take some responsibility for the balancing of these 3 - it shouldn't be only left up to management.


## Sometimes the Most Effective Way to Solve a Problem is to Redefine the Problem Itself

Or as President Underwood would say, ___"If you don’t like how the table is set, turn over the table"___. 

![](/assets/fu2016.jpg){:height="250px"}

### An Example of Redefining Unachievable into Achievable

I once joined a team that had a long-standing plan to migrate a large RMI-based system over to J2EE. 

The existing system had around 100 stateful services that were grouped together onto 6 or 7 servers, many of which communicated with each other in the course of fulfilling a single user request.
The main issue that was causing the business problems was resiliency - one problem in one server would generally impact all the others, and there was nothing to fallback to. Full outages ensued while servers were rebooted and fingers were crossed.

Unfortunately this one big problem that was causing serious business pain on a weekly basis, got lumped together with a lot of other issues and desires, and morphed into a goal of __"migrate this system to J2EE"__, with the implicit assumption that that would magically fix everything.

This would require rewriting almost everything, and take an indeterminate amount of time.

That was a _bad_ problem definition (in my experience this is a fairly common smell - a problem defined in terms of an implementation goal).

After living with this for a while, I finally got frustrated enough to propose an alternative. This involved redefining the problem to something that was simpler and more effective to solve. Which in this case was:

1. Make the system able to scale horizontally.
2. Nothing else.

This required rewriting just a subset of the system to no longer assume in-memory data was the single _source-of-truth_, then deploy all services horizontally onto multiple stacks of servers, with load-balancing across them (in a now common architecture). It was completed in about 6 months rather than who-knows-how-may years for the initial plan, and it worked.



