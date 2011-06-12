Shifty - A teeny tiny tweening engine in JavaScript. 
===

Shifty is a tweening engine for JavaScript.  That's it.  Shifty is a low-level library meant to be encapsulated by higher-level tools.  At the most basic level, it:

  * Tweening of `Number`s.
  * Provides extensibility hooks for the tweening.

Shifty is great because it focuses on doing one thing very well - tweening.  It is optimized to run many times a second with minimal processing and memory overhead, which is necessary for smooth animations.  To this end, the Shifty core doesn't do:

  * Keyframing.
  * Drawing.
  * Much else.

If you need functionality like this and more, you can easily extend or wrap Shifty's core with whatever you need.  In fact, there are some extensions included in this repo to do just that.  Currently, there are Shifty extensions for:

  * `shifty.color.js`: Color tweening (RGB/Hex strings).
  * `shifty.px.js`: `px` strings (so you can tween DOM elements).
  * `shifty.queue.js`: Queuing up tweens that execute sequentially.

There is also a file called `shifty.formulas.js` that contains a bunch of ready-to-use easing formulas.

Using Shifty
---

If you just want raw tweening functionality, all you need is `shifty.core.js`.  This is in the `src` directory.  Just drop that in your page, and you are ready to go.  

To use any Shifty extensions or additions, simply load them after `shifty.core.js`.

API
===

The section explains how to use the Shifty core.  For information on each extension, explore the `doc/` directory.

__Making a `tweenable()` instance__

The first thing you need to do is create a `new` instance of `Tweenable` and `init()` it.  Here's a one-liner example:

````javascript
var myTweenable = (new Tweenable()).init();
````

__Why do I make you call `init()`?__  Because I like to make you do mindless busy work.  More importantly, `Tweenable()` is meant to be inherited - properly, in context of JavaScript's prototypal inheritance model - and `Tweenable()` objects need to maintain individual state.  In plain English, they need their own properties that are not shared across instances.  In even plainer English, calling `init()` ensures that multiple `Tweenable()` instances do not share data that they shouldn't be sharing, and your stuff won't break mysteriously.

You can also supply some fun options to `init()`.  They are:

  * `fps`: This is the framerate (frames per second) at which the tween updates.  The default is `30`.
  * `easing`: The default easing formula to use on a tween.  This can be overridden on a per-tween basis via the `tween` function's `easing` parameter (see below).  This value is `linear` by default.
  * `duration`: The default duration that a tween lasts for.  This can be overridden on a per-tween basis via the `tween` function's `duration` parameter (see below).

##Shifty core methods##

__Tweening:__

````javascript
var aTween = myTweenable.tween( from, to );
````

You can optionally add some fun extra parameters:

````javascript
var aTween = myTweenable.tween( from, to, duration, callback, easing );
````

Or you can use the configuration object syntax (recommended!):

````javascript
var aTween = myTweenable.tween({
  from:       {  },            // Object.  Contains the properties to tween.  Must all be `Number`s.  Note: This object's properties are modified by the tween.
  to:         {  },            // Object.  The "destination" `Number`s that the properties in `from` will tween to.
  duration:   1000,            // Number.  How long the tween lasts for, in milliseconds.
  easing:     'linear',        // String.  Easing equation to use.  You can specify any easing method that was attached to `Tweenable.prototype.formula`.
  step:       function () {},  // Function.  Runs each "frame" that the tween is updated.
  callback:   function () {}   // Function.  Runs when the tween completes.
});
````

This method starts a tween.  You can use either format, but the second, longer format give you more hooks and controls.  The method returns an object that you can use to control a tween, as described in the next section.

__Important!__  The object that is passed as the `from` parameter, regardless of which syntax you use to invoke `tween()`, is modified.

##Controlling a tween##

Continuing from above...

````javascript
aTween.stop( gotoEnd );
````

Stops a tween.

  * `gotoEnd`: Boolean.  Controls whether to jump to the end "to" state or just stop where the tweened values currently are.

````javascript
aTween.pause();
````

Pauses a tween.  This is different from `stop()`, as you are to resume from a `pause()`ed state.

````javascript
aTween.resume();
````

Resumes a `pause()`ed tween.

````javascript
aTween.get();
````

Returns a tween's current values.

##Extending Tweenable()##

Shifty's true power comes from it's extensibility.  Specifically, it is designed to be inherited, and to fit easily into any prototypal inheritance chain.  A quick example of how to do that:

````javascript
function Cartoon () {
	this.init();
	console.log('Whoop whoop!  This is my framerate: ' + this.fps);
}

Cartoon.prototype = new Tweenable();
var myCartoon = (new Cartoon()).init();
````

This is awesome because any plugins or extensions that are present on the `Tweenable()` prototype are also available to `myCartoon`, and all instances of `Cartoon` (and `Tweenable`).  You can define these inheritable functions by attaching them to the `Tweenable.prototype` object.  A full example of this:

````javascript
// Add a new method to the `Tweenable` prototype
Tweenable.prototype.logMyProperties = function () {
	Tweenable.util.each(this, function (obj, prop) {
		console.log(prop + ': ' + obj[prop]);
	});
}

// Define a constructor function
function Cartoon () {
	this.init();
	console.log('Whoop whoop!  This is my framerate: ' + this.fps);
}

// Set `Cartoon`'s `prototype` to a `new` instance of `Tweenable`
Cartoon.prototype = new Tweenable();

// Make a new instance of `cartoon`
var myCartoon = (new Cartoon()).init();

// Test the new prototype method
myCartoon.logMyProperties();
````

That's fun, but how do we hook functionality into `Tweenable` instances themselves?

Hooks
---

You can attach various hooks that get run at key points in a `Tweenable` instance's execution.  The API:

````javascript
/**
 * @param {String} hookName The `Tweenable` hook to attach the `hookFunction` to.
 * @param {Function} hookFunction The function to execute.
 */
tweenableInst.hookAdd( hookName, hookFunction )
````

You can attach as many functions as you please to any hook.  Here's an example: 

````javascript
function limitX (state) {
	// Limit x to 300
	if (state.x > 300) {
		state.x = 300;
	}
}

var myTweenable = (new Tweenable()).init();

myTweenable.hookAdd('step', limitX);
````

This snippet will set the function `limitX` for `hookFunction` to be called every frame, after the values have been computed.  You can also remove hooks.  The API:

````javascript
/**
 * @param {String} hookName The `Tweenable` hook to remove the `hookFunction` from.
 * @param {Function|undefined} hookFunction The function to remove.  If omitted, all functions attached to `hookName` are removed.
 */
tweenableInst.hookRemove( hookName, hookFunction )
````

Example, continuing from above:

````javascript
myTweenable.hookRemove('step', stepHook);
````

The hooks you can currently attach functions to are:

  * `step`:  Runs on every frame that a tween runs for.  Hook handler function receives a tween's `currentState` for a parameter.

... And that's it.  They're easy to add in, please make Github issue or pull request if you'd like more to be added.

Filters
---

Filters are used for transforming the properties of a tween at various points in a `Tweenable` instance's lifecycle.  Filters differ from hooks because they get executed for all `Tweenable` instances globally.  Additionally, they are meant to convert non-`Number` datatypes to `Number`s so they can be tweened, and then back again. Just define a filter once, attach it to `Tweenable.prototype`, and all `new` instances of `Tweenable` will have access to it.

Here's an annotated example of a filter:

````javascript
Tweenable.prototype.filter.doubler = {
	// Gets called when a tween is created.  `fromState` is the state that the tween starts at, and `toState` contains the target values.
	'tweenCreated': function tweenCreated (fromState, toState) {
		Tweenable.util.each(obj, function (fromState, prop) {
			// Double each initial state property value as soon as the tween is created.
			obj[prop] *= 2;
		});
	},
	
	// Gets called right before a tween state is calculated.
	// `currentState` is the current state of the tweened object, `fromState` is the state that the tween started at, and `toState` contains the target values.
	'beforeTween': function beforeTween (currentState, fromState, toState) {
		Tweenable.util.each(toState, function (obj, prop) {
			// Double each target property right before the tween formula is applied.
			obj[prop] *= 2;
		});
	},
	
	// Gets called right after a tween state is calculated.
	// `currentState` is the current state of the tweened object, `fromState` is the state that the tween started at, and `toState` contains the target values.
	'afterTween': function afterTween (currentState, fromState, toState) {
		Tweenable.util.each(toState, function (obj, prop) {
			// Return the target properties back to their pre-doubled values.
			obj[prop] /= 2;
		});
	}
}
````

Yes, having `doubler` filter is useless.  A more practical use of filters is to add support for more data types.  __Remember, `Tweenable()` only supports `Numbers` out of the box__, but you can add support for strings, functions, or whatever else you might need.  The `px` and `color` extensions work by filtering string values into numbers before each tween step, and then back again after the tween step.