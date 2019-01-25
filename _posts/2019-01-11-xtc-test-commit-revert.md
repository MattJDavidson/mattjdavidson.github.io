---
layout: post
title: test && commit || revert - What I've learned.
---

On Tuesday 8th of January I faciliated a dojo on test && commit || revert. 

We were lucky enough to have Kent answer some questions via webcam at the end of the session.

"Hated the idea so had to try it"
"It's cheap and you will learn something"

"How can I make this change in smaller steps"

## TCR tricks
- code your failures, i.e. this won't pass will help you make progress
- code your exceptions, i.e. this will throw out of bounds. This makes your understanding, or lack thereof, explicit.

## Missing the red step

- TCR tricks show you ways to get around this.
- Pushing green all the time feels liberating.
- I miss seeing red. Would jazz be fun (jazz) without resolving tension (the failing test)

## TCR is new
TCR is a new idea, it is very much in its infancy. 

## Lessons from hosting a TCR dojo
- Choose your kata wisely, we used fibonacci and it isn't a great choice, it's difficult to grow the code eventually you just [write the whole program](https://external-preview.redd.it/DodWFQ9mQkVyWoKFa0ZIu12PYrPo3P2T0taaK-lgJCo.png?width=530&auto=webp&s=034d71021191d7e4229efddf6d0df1e60e5f6477).
- Fizzbuzz or Tennis Scoring are suggested as good development katas.
- Gilded Rose is suggested as a good refactoring based kata.
- I think six pack of beer from [99 Bottles of OOP](https://www.sandimetz.com/99bottles/) would be a good one too.

## Resources to learn more


## Assumptions
- You're comfortable in Detroit style TDD

## Issues
- Commit messages seem fairly meaningless
- Difficult to track a vein of changes through interleaved pushes to master.
  Maybe tooling will help support this in future.

## Ideas to explore

- Which tooling are IDEs lacking to support TCR
- What does it feel like to TCR if you're not used to practicing TDD
- Is TCR an effective way to learn to program?
