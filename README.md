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

Knockout needs the data that it binds to the DOM to be wrapped in Knockout observables.
There are no layers or abstractions with which to reason for complex components, it's all
about observables. Knockout observables end up spreading everywhere. If an observable depends on
another observable in a different model class you will need to couple those two
model classes to pass the observable. Classes would reach inside each other and access observables
without a second thought. They would modify the observable value if they needed, there was no
explicit ownership of model data.

For small to medium complexity components this is not an issue, you can hold the structure in your head.
As components scale in complexity though, it becomes harder to correctly reason about your code structure
and data flow.

Flux provides something that Knockout sorely lacks, structure and abstractions for scaling and a
well defined and consistent data flow.

### Constraints

Our applications are real time trading solutions with the possibility of spikes in streaming
traffic and high levels of updates. We could have clients with 10+ FX tiles open in a layout, each
of those streaming updates for highly active currency pairs. 4 updates a second per currency pair was
a benchmark we have been set before. Tiles could be opened up, so they could display
multiple prices (different settlement dates or traded amounts would have different prices).
This meant a tile could have 9 separate sets of prices streaming in, one for each row in a ladder.

We also had tabbed layouts which allowed clients to have background tabs open with a different set
of tiles. These tiles would not be visible and we didn't want them to interfere with the performance
of the visible ones. In essence we wanted each tile component to be separate from all the other tile
components. This would allow granular control over every tile's data subscriptions and behaviour.

### Concerns with the Facebook Flux pattern

These constraints meant that we were wary of adopting the Flux pattern as implemented in the example
[Flux applications](https://github.com/facebook/flux/). The general data flow and abstractions,
were appealing but the exact mechanics we wanted to tweak.

![Flux data flow.](assets/flux-diagram-white-background.png)

The change we made to the pattern was to make the actors of the system class instances.
In the example Flux applications the Stores, ActionCreators and Dispatchers are all Singletons.
This is fine if you only have one instance of a component or have a low flow of data
through the Dispatcher. With one component instance (let's say an application menu) you wouldn't
need multiple instances of its ActionCreator or Store. If you had a low flow of data and
multiple instances of a component you could distinguish data by a component unique ID.

In the case of a component with a high update rate, high complexity and significant amounts
of state the Singleton approach seemed awkward.
