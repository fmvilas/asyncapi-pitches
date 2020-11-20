<h2>On This Page</h2>

<!-- TOC depthFrom:2 depthTo:6 updateOnSave:true -->autoauto- [Summary](#summary)auto    - [Problem Overview](#problem-overview)auto        - [Related issues](#related-issues)auto    - [Solution Overview](#solution-overview)auto    - [Positive Side-Effects](#positive-side-effects)auto- [Problem Details](#problem-details)auto    - [Match these headings to the bullets in your overview](#match-these-headings-to-the-bullets-in-your-overview)auto- [Solution guidelines](#solution-guidelines)auto    - [Define the domain and show a reasonable solution](#define-the-domain-and-show-a-reasonable-solution)auto    - [Solution elements](#solution-elements)auto- [Boundaries](#boundaries)auto        - [Don’t do x](#dont-do-x)auto        - [Please leave y alone for now](#please-leave-y-alone-for-now)auto        - [Watch out for z](#watch-out-for-z)auto- [KPIs](#kpis)auto- [Long-Term Vision](#long-term-vision)auto    - [Cast a vision for where this is going](#cast-a-vision-for-where-this-is-going)autoauto<!-- /TOC -->

## Summary

### Problem Overview

Currently, there are a bunch of things that are tedious and we're either doing them manually (making us waste a lot of time and energy) or not doing them at all yet because that would be a lot of extra work:

1. Deploy Playground (playground.asyncapi.io).
1. Update specification on the website.
1. Publish Generator documentation on the website.
1. Publish JS Parser documentation on the website.
1. Bump versions of package dependencies.
1. Run tests in workflows not only on Linux but Windows too.
1. Validate titles of PRs to make sure they use Conventional Commits.
1. Merge pull requests when they are labeled as `automerge`.
1. Publish a tweet every time there's a release on a repo.

#### Related issues

* https://github.com/asyncapi/asyncapi/issues/423

### Solution Overview

1. Use Github Actions to deploy Playground.
1. --
1. --
1. --


### Positive Side-Effects

1. We'll be able to focus more on the important stuff rather than losing time doing manual tasks that a machine can do for us.
1. We'll get less issues related to the spec on the website being outdated compared to the one on the Github repo.

## Problem Details

### Match these headings to the bullets in your overview

This details section is where the meat is. Show the problem as clearly and thoroughly as you can.
 
* Include screenshots, metrics, user feedback, etc.
* Help the team understand the severity of the problem and who is affected.


## Solution guidelines

### Define the domain and show a reasonable solution

**A well-shaped solution defines the domain** (e.g., the screens or sections of the product you want the project team to work on). This is important for a few reasons:

* **Bets are made orthogonally.** In most cases, we don’t want to assign more than one project team to the same area of the product because that inherently adds external dependencies and complicates communication.

* **A big part of shaping a solution is de-risking it.** It is virtually impossible to identify boundaries and rabbit trails without being pretty firm about where the solution lives.

* **Identifying scopes requires defining the domain.** The project team’s first steps should be to survey the land and [get one piece done](https://basecamp.com/shapeup/3.2-chapter-10). If they also burn precious time deciding which parcel of land to survey, they are immediately at risk of not being able to get started on the first piece.
 
**Your pitch is based on imperfect information.** The project team will uncover issues you never considered, so your solution should be flexible enough to accommodate new takes on the execution.
 
* **Use flexible language.** Directions like “consider”, “explore”, and “try” help the team to see your solution while still providing the freedom to make it their own.

* **Show your solution.** Wireframes or mocks are too concrete, while paragraphs are too abstract. Combining flexible language and [fat marker sketches](https://basecamp.com/shapeup/1.1-chapter-02#case-study-the-dot-grid-calendar) is a great recipe for demonstrating a de-risked solution while remaining open-handed about the details.

* **The project team will likely default to your direction.** People tend to talk about pictures, not paragraphs. By seeding the solution with sketches you are inherently directing their conversation.

### Solution elements

## Boundaries

Re-state the main kernel the team should take away from this pitch and list out boundaries you won’t want them to cross. Be firm – these boundaries should protect their autonomy and outline the rabbit trails you discovered during shaping.

#### Don’t do x
Some boundaries are like fences along a path. They are aimed at helping the team stay on course.

#### Please leave y alone for now
Some boundaries are like construction lanes on a road. They protect other teams or initiatives from getting run over.

#### Watch out for z
Some boundaries are cautionary. They call out dangers inherent to the domain or the execution.

## KPIs

* List out the means by which we will measure success
* Link to any existing dashboards
* Call out any charts or screens the team will need to create


## Long-Term Vision

### Cast a vision for where this is going

While a reasonable solution scoped to a 6-week timeline affords some really positive dynamics, it can also make the solution feel like a band-aid to the team executing the work. In the event that you have some kind of longer term vision, summarize it here.
 
This helps the team better understand how this work fits into the bigger product vision. It also enables them to make smarter decisions and empowers them to provide feedback on that future vision as they uncover new information over the course of the project.
