---
layout: index
permalink: /index.html
title: Flux for components
subtitle: The Flux pattern, without the globals

---

There is no shared, single version of Flux, the one source of truth that we can all point to.
That makes sense, it's a software pattern after all. Not a concrete artifact like a library or
framework.
This has encouraged developers to create variations on and [implementations](http://fluxxor.com/)
of the pattern.

I've explained a variation of the Flux pattern often enough that I felt it was time to put
it into a blog. If you're coming to this post with no knowledge of Flux this will not be the best
starting point to learn about it. Some of the concepts will be explained but this is not an
introductionary post.

### Context

I've been working on [Knockout](http://knockoutjs.com/) based applications for several years;
since before frameworks like Angular and Ember were mature. Knockout though has been showing some of
it's limitations and age.

Knockout expects the data that it inserts into the DOM to be wrapped in Knockout observables.
There are no layers or abstractions with which to reason for complex components, it's all
about observables. Knockout observables end up spreading everywhere. If a model class is interested
in a field value you will pass the observable to it and to all the classes that wish to access
that value. When the value changes the change cascades through all those classes.

For small components this is not an issue, you can hold the flow of data in your head.
As components scale in complexity though it becomes harder to correctly reason about your components.

Flux provides something that Knockout sorely lacks, structure and abstractions for scaling and a
well defined and consistent data flow.

### Constraints

Our applications are real time trading solutions with the possibility of spikes in streaming
traffic and high levels of updates. We could have clients with 10+ FX tiles open in a layout, each
of those streaming updates for highly active currency pairs. 4 updates a second per currency pair was
a benchmark we have been set before. Tiles could be opened up, so they would have ladders displaying
multiple prices (different settlement dates or traded amounts would have different prices).
This meant a tile could have 9 separate sets of prices streaming in, one for each row in a ladder.

We also had tabbed layouts which allowed clients to have background tabs open with a different set
of tiles. These tiles would not be visible and we didn't want them to interfere with the performance
of the visible ones.

### Differences from the Facebook Flux pattern

![My helpful screenshot](assets/flux-diagram-white-background.png)
