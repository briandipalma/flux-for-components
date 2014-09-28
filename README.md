---
layout: index
permalink: /index.html
title: Flux for components.
subtitle: The flux pattern, without the globals.
---

Following some good community reviews of React we're experimenting with React and the Flux application pattern.
We want to see how it measures up against our current Knockout based approach. If you're coming to this post with no knowledge of React and Flux this might not be the best starting point to learn about them. Some of the concepts around Flux will be explained but this is not meant to be an introductionary post.

### What's what.

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
