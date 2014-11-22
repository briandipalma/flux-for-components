---
layout: index
permalink: /index.html
title: Flux for components
subtitle: The Flux pattern, without the globals
---

There is no shared, single implementation of Flux, the one source of truth that we can all point to.
That makes sense, it's a software pattern after all. Not a concrete artifact like a library or framework.
This has encouraged developers to create variations on and [implementations](http://fluxxor.com/)
of the pattern.

I've explained a variation of the Flux pattern often enough that I felt it was time to put
it into a blog. If you're coming to this post with no knowledge of Flux this will not be the best
starting point to learn about it. Some of the concepts will be explained but this is not meant to be
an introductionary post.

### What's what.

We want to see how it measures up against our current Knockout based approach.

React is a library that provides the "V" in your front end stack. It handles the view and is agnostic as to what technology you use to drive it.
You can plug it into Backbone, Angular or any other stack that provides concepts such as models and controllers or anything that can drive a view.
Knockout is slightly more opinionated and expects the data that it inserts into the DOM to be wrapped in Knockout observables but it also focuses on the view layer.

We've been using a Knockout based approach for several years since before frameworks like Angular and Ember were mature.
Knockout though has been showing some of it's limitations and age

There are no layers, architecture or abstractions with which to reason for complex components, it's all about observables.

Knockout observables end up spreading everywhere, if you are interested in a field value and it's available via the observable you will pass it to all the classes that wish to access that value.

Flux provides something that Knockout sorely lacks, structure.

### Differences from Facebook Flux pattern.

![My helpful screenshot]({{ site.url }}/assets/flux-diagram-white-background.png)
