# Hire a programmer for your next wedding? My pickleball tournament journey

My friends are getting married this weekend! Spotting Janae's and my camp counselor essence, the groom asked that we help facilitate a day-before gathering featuring the fastest growing sport in the U.S. - the one sport where I can get crushed by women in their 60's and not feel bad about it. We have two hours, seven courts, and 56 players showing up to this thing. When you have a hammer, everything looks like a nail, and when I have Claude, everything looks like a problem that requires a computer. I sought out to make the golden tournament structure, fine-tuned to Friday. Today I'll walk you through constraint programming, the method to the madness.

## What Even Is Constraint Programming?

Constraint programming (CP) is the art of telling a computer exactly what rules a solution must obey, then letting it figure out how to find one — ideally a good one. It's philosophically different from approaches like gradient descent, which nudge a solution continuously toward something better. CP is more like a logic puzzle solver: it builds up partial assignments, prunes branches that can't possibly work, and backtracks when it hits a dead end. Google's CP-SAT solver is a particularly sophisticated version of this — it combines SAT-solving techniques with linear programming relaxations to search enormous spaces with impressive efficiency. You define your variables, declare your constraints, specify an objective to minimize, hand it all to the solver, and then go make a cup of tea while it does things you absolutely could not do by hand. When it comes back with an answer, it's not a guess or an approximation — it's a provably valid schedule, optimal to within whatever time limit you gave it. This is, in my opinion, one of the more underappreciated tools in the applied engineer's toolkit, and I was delighted to have an excuse to use it.

## Hard Rules and Soft Preferences: The Architecture

The key design insight in any constraint model is separating what *must* be true from what you'd merely *like* to be true. Hard constraints are non-negotiable: every court has exactly four players per round, every player plays at least a minimum number of rounds, no two players are ever partners more than once, and — to keep things speedy in between rounds — no player moves more than two courts between consecutive rounds. The soft objective, then, is about co-court appearances: this is a mingling event, and I wanted to make sure that everyone was seeing new faces. Ideally, the same two players never share a court more than once, but when they do, we want those repeats distributed as fairly and sparsely as possible. How to express that mathematically turned out to be the interesting part.

## The Cost Function: An Iterative Descent Into Specificity

I built the cost function in stages, each one prompted by looking at the solver's output and seeing things I wanted to tweak. Stage one was a flat linear penalty on co-court repeat appearances — every time two players shared a court more than once, add one to the objective. The solver dutifully minimized this, but treated a pair's second and third co-court appearance as equally bad, but seeing the same person three times in a tournament definitely detracts from the sense of variety more than seeing them twice. Stage two introduced an exponential staircase: no penalty for appearing together once, a penalty of 1 for a second appearance, 4 for a third, 13 for a fourth, using `3^(k-2)` for each appearance beyond the first. Better. Then I noticed the solver was concentrating repeat appearances on individual players — one person racking up multiple distinct repeat pairings while everyone else enjoyed variety — so stage three added a second staircase penalizing per-player accumulation. The weights between these two levels needed careful calibration: the pair-level penalty had to strictly dominate the player-level penalty, meaning no amount of player concentration badness could ever make a pair's third appearance together look acceptable. By the end, I found the golden schedule.

## What I Learned From Building This With Claude

I didn't build this alone — I worked through the implementation iteratively with Claude, which felt like a superpower. I didn't have to worry about the technicalities of formalizing the constraints into CP-SAT syntax; I could use English. I'm glad I had the knowhow to debug when I needed to, but for an independent project that just needs reasonable output, it was incredibly fast.

Marriage is a beautiful commitment. I'm glad I got to be a part of a weekend they'll remember forever.

### The Schedule
Decided on two equivalent waves of 28 players each, using the following schedule:

| Number | Round 1 Partner | Round 2 Partner | Round 3 Partner |
| ------ | --------------- | --------------- | --------------- |
| 1      | 2 (Court 1)     | 17 (Court 3)    | 6 (Court 4)     |
| 2      | 1 (Court 1)     | 3 (Court 1)     | 18 (Court 2)    |
| 3      | 4 (Court 1)     | 2 (Court 1)     | 26 (Court 3)    |
| 4      | 3 (Court 1)     | 9 (Court 2)     | 7 (Court 1)     |
| 5      | 6 (Court 2)     | 13 (Court 3)    | 19 (Court 5)    |
| 6      | 5 (Court 2)     | 14 (Court 2)    | 1 (Court 4)     |
| 7      | 8 (Court 2)     | 11 (Court 1)    | 4 (Court 1)     |
| 8      | 7 (Court 2)     | 22 (Court 4)    | 15 (Court 6)    |
| 9      | 10 (Court 3)    | 4 (Court 2)     | 13 (Court 2)    |
| 10     | 9 (Court 3)     | 18 (Court 4)    | 14 (Court 3)    |
| 11     | 12 (Court 3)    | 7 (Court 1)     | 17 (Court 1)    |
| 12     | 11 (Court 3)    | 15 (Court 5)    | 16 (Court 7)    |
| 13     | 14 (Court 4)    | 5 (Court 3)     | 9 (Court 2)     |
| 14     | 13 (Court 4)    | 6 (Court 2)     | 10 (Court 3)    |
| 15     | 16 (Court 4)    | 12 (Court 5)    | 8 (Court 6)     |
| 16     | 15 (Court 4)    | 19 (Court 6)    | 12 (Court 7)    |
| 17     | 18 (Court 5)    | 1 (Court 3)     | 11 (Court 1)    |
| 18     | 17 (Court 5)    | 10 (Court 4)    | 2 (Court 2)     |
| 19     | 20 (Court 5)    | 16 (Court 6)    | 5 (Court 5)     |
| 20     | 19 (Court 5)    | 28 (Court 7)    | 24 (Court 7)    |
| 21     | 22 (Court 6)    | 27 (Court 6)    | 25 (Court 6)    |
| 22     | 21 (Court 6)    | 8 (Court 4)     | 28 (Court 5)    |
| 23     | 24 (Court 6)    | 26 (Court 5)    | 27 (Court 4)    |
| 24     | 23 (Court 6)    | 25 (Court 7)    | 20 (Court 7)    |
| 25     | 26 (Court 7)    | 24 (Court 7)    | 21 (Court 6)    |
| 26     | 25 (Court 7)    | 23 (Court 5)    | 3 (Court 3)     |
| 27     | 28 (Court 7)    | 21 (Court 6)    | 23 (Court 4)    |
| 28     | 27 (Court 7)    | 20 (Court 7)    | 22 (Court 5)    |


