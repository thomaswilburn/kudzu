Kudzu
=====

A databinding microlibrary

The basics
----------

Kudzu is a simple client-side library for binding data to DOM structures, with some sugar for handling events. It's similar to Vue, but much smaller--Kudzu doesn't support list iteration, reactive properties, or components. It's primitive, but it's tiny and ideal for the use case of creating small, easy-to-update chunks of DOM.

Intro
-----

Kudzu manages the DOM by mapping data to special attributes in your HTML. To use it, you add special attributes to your markup, then pass a root element and a state object to the Kudzu factory function. For example, we might write the following HTML:

```html
<div id="example" :innerhtml="greeting" @click="onClick">Hello, world!</div>
```

Then we create our binding:

```js
var element = document.querySelector("#example");
var binding = Kudzu(element, {
  greeting: "Hello, Kudzu!",
  onClick: function(e) {
    binding.set("greeting", "Clicked!")
  }
});
```

When Kudzu initializes, it will understand that the `greeting` property of our state object should be used to control the innerHTML property of the element. It will also attach an event listener to the element where the `@click` attribute is used, so that it can respond to click events by calling the `onClick` method of our state object. In turn, that method calls `set()` on the binding to update the greeting.

Kudzu attributes
----------------

Although the custom attributes used by Kudzu may look weird to many coders, they're fully supported in HTML and can be combined to create surprisingly complex behavior from only a little markup. There are three main kinds of attributes that Kudzu uses for its data-binding:

* `attr:name` - binds a value on the state object to the "name" attribute or property in the DOM. This type of binding can also use the shorthand `:name` for brevity. Because this binds to both attributes or properties, you can use this to update the contents of an element via its `innerHTML` property, as in the example above, as well as any other property you might want to change.
* `class:name` - toggles a class on or off, depending on whether the value of the "name" property is truthy or not. For example, you might write `class:visible="showModal"` to set the visibility of an element via its CSS class.
* `on:event` - registers an event listener for "event" on the element. Event listeners are passed the event object as their first argument. You can use the shorthand `@event` to register listeners as well.

The value of these attributes tells them where to look on our state object, and can include sub-paths (e.g., given a state object called `data`, `:innerhtml="text.body"` will set the element's contents to the value of `data.text.body`).

Kudzu attributes will only be read at initialization. However, they are "lazy-evaluated," meaning that Kudzu looks up their value dynamically each time they're used. This means that you can replace an event listener at run-time, and the new function will be called in place of the old one, as long as the path on the object is the same.

Updating data
-------------

When you create a binding in Kudzu, an augmented version of your state object is returned. You can read from this object freely, but to make changes, you should use the binding's `set()` method to notify it that an update is required:

```js
// call for a single property:
binding.set("greeting", "Yo");

// set deeper keypaths in the state object
binding.set("text.body", "This space intentionally left blank.");

// set multiple properties by passing a single-level object instead:
binding.set({
  greeting: "Hey there",
  "text.body": "What are the haps, my friend?"
});
```

Kudzu uses two strategies to keep updates efficient. First, it batches all value changes into the next event loop using `requestAnimationFrame`, so you can write to a property as many times as you want, but only the final value will be used. Second, it memoizes its data bindings, and only makes a change if the property differs from the last render cycle.

Removing Kudzu
--------------

In real life, it's almost impossible to get rid of kudzu. For this library, however, if you want to disconnect event listeners and nullify your instance, you can call `destroy()` on the binding object to remove all event listeners and destroy references that it has stored to your DOM. This is useful if you have updated or added new flag attributes to the DOM. Note that the binding's data properties will remain, which means that you can use that object to instantiate new bindings.