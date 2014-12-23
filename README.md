---
layout: index
permalink: /index.html
title: Flux for components
subtitle: The Flux pattern, without the globals

---

There is no shared, single version of Flux, the one source of truth that we can point to.
That makes sense, it's a software pattern after all. Not a concrete artifact like a library or
framework. This has encouraged developers to create various implementations of the pattern.
I've explained an approach to the Flux pattern often enough that I felt it was time to put
it into a blog.

## Context

I've been working on Knockout based applications for several years;
since before frameworks like Angular and Ember were mature. Knockout though has been showing some of
it's limitations and age.

Knockout needs the data that it binds to the DOM to be wrapped in Knockout observables.
Knockout observables end up spreading everywhere. If an observable
depends on another observable in a different model class you will need to couple those two
model classes by passing the observable. Classes can reach inside each other and access observables
without a second thought. They would modify the observable value if they needed, there was no
explicit ownership of model data.

For small to medium complexity components this is not an issue, you can hold the structure in your
head. As components scale in complexity though, it becomes harder to correctly reason about your code
structure and data flow.

Flux provides something that Knockout sorely lacks, structure and abstractions for scaling and a
well defined and consistent data flow.

## Constraints

Our applications are real time trading solutions. They must handle spikes in streaming
traffic and high levels of updates. We could have clients with 10+ FX tiles open in a layout, each
receiving updates for a highly active currency pair. 4 updates a second per currency pair was
a benchmark we have been set before. Tiles could be opened up, so they could display
multiple prices (different settlement dates or traded amounts would have different prices).
This meant a tile could have 9 separate prices streaming in, one for each row in a ladder.

![example FX application](https://globalmarkets.bnpparibas.info/gm/features/images/FX/Screenshot_CortexFX_Pricing_Engine.jpg)

We also have tabbed layouts which allowed clients to have background tabs open with a different set
of tiles. These tiles would not be visible and we didn't want hidden components to interfere with the
performance of the application. In essence we wanted each tile component to be separate from the other
tile components. This would allow granular control over every tile's data subscriptions and behaviour.

## Concerns with the Facebook Flux pattern

These constraints meant that we were wary of adopting the Flux pattern as implemented in the example
[Flux applications](https://github.com/facebook/flux/). The general data flow and abstractions,
were appealing but the exact mechanics we wanted to tweak.

The change we made to the pattern was to make the actors of the system class instances.
In the example Flux applications the stores, action creators and dispatchers are all singletons.
This is fine if you only have one instance of a component (let's say a shopping cart). With one
component instance you don't need multiple instances of its action creators or stores.

Things become more complex if you have multiple instances of a component all sharing the same
action creators and stores.

#### Distinguishing data for component instances

If you have multiple instances of a component you can distinguish data by a unique component ID.
You need to add code in stores, action creators and views to distinguish actions by component ID.
With singletons each instance of your component must generate a unique ID and pass that around as
a namespace for its data. You wouldn't want a chat message inputted in one chat window being added to
every single chat thread. This results in some boilerplate code; code that passes around
IDs and filters data based on these IDs.

{% highlight javascript %}
getAllForThread: function(threadID) {
	var threadMessages = [];
	for (var id in _messages) {
		if (_messages[id].threadID === threadID) {
			threadMessages.push(_messages[id]);
		}
	}
	...
}
{% endhighlight %}

The snippet above, from the Flux chat example application, displays the sort of boilerplate
required. Not complex, but removing this accidental complexity is a plus.
Given that our components have a high complexity and significant amounts of state we would need to
add ID boilerplate code in a lot of classes.

#### Performance

The other concern was performance. Issues could arise due to the high volume of data events flowing
into stores. The stores emit change events which trigger a view rerender. As all instances of a view
would be registered to the same store they would all be notified of a store state change.

{% highlight javascript %}
case ActionTypes.CLICK_THREAD:
	ChatAppDispatcher.waitFor([ThreadStore.dispatchToken]);
	_markAllInThreadRead(ThreadStore.getCurrentID());
	MessageStore.emitChange();
	break;
{% endhighlight %}

The snippet above, again from the Flux chat example application, shows how when the store
changes its state it notifies all the component views listening to it.
If you have 10 chat windows and one action triggers an update to its store all 10 chat window
views could trigger a rerender. There are several reasons why when using React it shouldn't result in
too much waste. Firstly it batches Virtual DOM diffing secondly the Virtual DOM prevents unnecessary
DOM updates and finally you could implement custom `shouldComponentUpdate` methods for your components.

Given the high update rates for certain tiles, which could trigger rerender requests for tiles that
might not even be visible we had to consider the possibility of needing those custom
`shouldComponentUpdate` methods.

While the above concerns are tractable, using instances instead of singletons allowed us not to worry
about them.

## Implementing the changes

Deciding to use instances instead of singletons required a different approach to accessing the
actors in a Flux system. With singletons all you had to do was `require` the modules.

{% highlight javascript %}
var MessageStore = require('../stores/MessageStore');
...
var ThreadStore = require('../stores/ThreadStore');
var UnreadThreadStore = require('../stores/UnreadThreadStore');
{% endhighlight %}

The approach we took was to use a factory. The factory API has getters for the actors in the Flux
pattern.

{% highlight javascript %}
class FluxDependenciesFactory {
	constructor(dispatcher) {}

	getDispatcher {}
	getStore(storeName) {}
	getUtility(utilityName) {}
	getActionCreator(actionCreatorName) {}

	registerStore(storeName, storeClass) {}
	registerUtility(utilityName, utilityClass) {}
	registerActionCreator(actionCreatorName, actionCreatorClass) {}
}
{% endhighlight %}

For each instance of a component, such as an FX tile, we would create a new factory
and pass that factory into the component as a view `prop`. The React `Tile` view component would
pass the factory on downward to its child view components. From that point in the view downward
the dispatcher, views, stores and action creators would all share the same dependency instances.
These instances would be separate from the instances in another `Tile` view component.

{% highlight javascript %}
const componentDispatcher = new Flux.Dispatcher();
const fluxDependenciesFactory = new FluxDependenciesFactory(componentDispatcher);

populateFactory(fluxDependenciesFactory);

React.render(
	<Tile factory={fluxDependenciesFactory} />,
	this._mountNode
);
{% endhighlight %}

The `populateFactory` function would register the constructors of the actors
by their name.

{% highlight javascript %}
function populateFactory(fluxDependenciesFactory) {
	...
	fluxDependenciesFactory.registerStore('TenorLadderRowDate', TenorLadderRowDateStore);
	fluxDependenciesFactory.registerStore('TenorLadderRowButton', TenorLadderRowButtonStore);
}
{% endhighlight %}

The first time a request for an actor is received by the factory it will create it; any subsequent
requests for that actor will return the same instance.

{% highlight javascript %}
const store = this.props.factory.getStore('TenorLadderRowButton');
{% endhighlight %}

### Drawbacks

The tradeoffs to this approach are having to pass the factory into your view components and
the code to register the actors the factory creates.

### Extra advantages

There are other benefits with this approach, apart from the ones mentioned above.

* Unit testing doesn't have to rely on mocking required modules. It's trivial to provide mocks to
stores and views as you pass a mock factory into them during testing.
* The factory can pass state in the form of [cursors](https://github.com/swannodette/om/wiki/Cursors)
into stores when it creates them. This is useful for deserialized components.
* It makes server side rendering simpler, as you create classes per request. Using a singleton
would result in shared state for each request until the server is restarted.
* For framework creators this approach could allow the consumers of the framework components to
provide their own versions of the actors. This would require a way for consumers to hook into the
actor registration phase.

## Right tool for the job

There is clearly an overhead to building components in this fashion. So it's important to point out
that this isn't meant to be the one true way to build all components.

* For simple to medium complexity components pure React might suffice. Components where data flows
downward and there is low need for upward communication between components.
* With medium to high complexity components you should start using Flux. This is where events need to
be passed back up the component hierarchy or there is a lot of computed state.

The chosen approach should be as simple as required and no more simple, sometimes it turns out
that singletons are too simple.
