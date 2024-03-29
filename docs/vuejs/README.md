# Vue.js

## Common Gotchas

_Published: April 4, 2017_

Source: [Alligator.io](https://alligator.io/vuejs/common-gotchas/)

> As with any framework, Vue has a few oddities that might take newcomers a little while to get used to, and many stumble over. Here, we’ll attempt to list a number of those and how to work with and/or around them.

### Reactivity

Vue’s reactivity system is great, but it doesn’t handle quite everything. There are a few edge cases that Vue can’t detect. (Yet. Hopefully when ES6 Proxies are widely supported these caveats will be gone.)

* When properties are added or removed from an object, Vue won’t know about it and won’t make them reactive.
* You can’t add new properties to the root data object directly, but you can use `Vue.set(this.data, ‘propname’, value)`
* Vue can’t detect when a particular index value changes in an array from setting it directly through `array[index] = value`. The workaround is to use `Vue.set(array, index, value)`
* Vue can’t detect when the length of an array changes. Use splice instead.

## Global Event Bus

_Published: January 9, 2017_

Source: [Alligator.io](https://alligator.io/vuejs/global-event-bus/)

> The event bus / publish-subscribe pattern, despite the bad press it sometimes gets, is still an excellent way of getting unrelated sections of your application to talk to each other. But wait! Before you go waste a few more precious KBs on another library, why not try Vue’s powerful built-in event bus?

### Initializing

``` javascript event-bus.js
import Vue from 'vue';
export const EventBus = new Vue();
```

All you need to do is import the Vue library and export an instance of it. (In this case, I’ve called it EventBus.) What you’re essentially getting is a component that’s entirely decoupled from the DOM or the rest of your app. All that exists on it are its instance methods, so it’s pretty lightweight.

### Using the Event Bus

#### Sending Events

Say you have a really excited component that feels the need to notify your entire app of how many times it has been clicked whenever someone clicks on it. Here’s how you would go about implementing that using EventBus.emit(channel: string, payload1: any, …).

``` javascript
<template>
  <div @click="emitGlobalClickEvent()"></div>
</template>

<script>
// Import the EventBus we just created.
import { EventBus } from './event-bus.js';

export default {
  data() {
    return {
      clickCount: 0
    }
  },

  methods: {
    emitGlobalClickEvent() {
      this.clickCount++
      // Send the event on a channel (div:click) with a payload (the click count.)
      EventBus.$emit('div:click', this.clickCount);
    }
  }
}
</script>
```

#### Receiving Events

Now, any other part of your app kind enough to give `PleaseClickMe.vue` the attention it so desperately craves can **import EventBus** and listen on the i-got-clicked channel using EventBus.$on(channel: string, callback(payload1,…)).

``` javascript
// Import the EventBus.
import { EventBus } from './event-bus.js';

// Listen for the div:click event and its payload.
EventBus.$on('div:click', clickCount => {
  console.log(`Oh, that's nice. It's gotten ${clickCount} clicks! :)`)
});
```

::: tip Only the First Time
If you’d **only** like to listen for the **first** emission of an event, you can use ```EventBus.$once(channel: string, callback(payload1,…))```.
:::

#### Removing Event Listeners

``` javascript
// Import the EventBus we just created.
import { EventBus } from './event-bus.js';

// The event handler function.
const clickHandler = function(clickCount) {
  console.log(`Oh, that's nice. It's gotten ${clickCount} clicks! :)`)
}

// Listen to the event.
EventBus.$on('div:click', clickHandler);

// Stop listening.
EventBus.$off('div:click', clickHandler);
```

::: tip Remove Event
You could also remove **all** listeners for a particular event using `EventBus.$off(‘div:click’)` with no callback argument.
:::

::: warning Remove All
If you really need to remove every single listener from EventBus, regardless of channel, you can call `EventBus.$off()` with no arguments at all.
:::

## Handle Touch Events in Vue.js with vue-touch

_Published: May 3, 2017_

Source: [Alligator.io](https://alligator.io/vuejs/vue-touch-events/)

> It seems these days that more and more people are foregoing their tried-and-true big screens, keyboards and mice, for a tiny new-fangled slab of glass and metal that does nothing but frustrate those of us with large fingers. Because of this, unfortunately, it has almost become unthinkable to develop only for the reasonable platforms (desktops, laptops, and mainframes.) We now have to worry about handling things like taps and swipes with greasy fingers. To make matters worse, the web platform doesn’t quite support that sort of stuff too well. For those of us developing with Vue.js though, never fear. [vue-touch](https://github.com/vuejs/vue-touch/tree/next) is here to make your finger-poking woes go away. It uses [Hammer.js](https://hammerjs.github.io/) under the hood to provide a nice simple API for handling touch events.

### Installation

**NOTE:** ```vue-touch``` for Vue 2.0 is currently (05-03-2017) in beta, and therefore must be installed with the ```@next``` tag.

``` bash
# Yarn
$ yarn add vue-touch@next
# or NPM
$ npm install vue-touch@next --save
```

Now, as always, enable the plugin in your main app file.

_src/main.js_
``` javascript
import Vue from 'vue';
import VueTouch from 'vue-touch';
import App from 'App.vue';

Vue.use(VueTouch);

new Vue({
  el: '#app',
  render: h => h(App)
});
```

### Usage

VueTouch adds a single component, ```v-touch```. It normally renders as a div unless configured otherwise with a ```tag``` property. It wraps your components and sets up the Hammer.js manager and recognizers. All you have to do is bind to the events.

``` javascript
<template>
  <div>
    <v-touch @swipeleft="doSomething">
      <p>I can now be swiped on!</p>
    </v-touch>
    <v-touch @rotate="rotateAThing">
      <p>Rotate me!</p>
    </v-touch>
  </div>
</template>
```

You can pass Hammer.js configuration options to the v-touch component with :```EVENT-options```. (ex. ```:swipe-options```, ```:rotate-options```) Full documentation on the various options can be found here: [https://hammerjs.github.io/getting-started/](https://hammerjs.github.io/getting-started/)

``` javascript
<template>
  <div>
    <v-touch @swipeleft="doSomething" :swipe-options="{ threshold: 200 }">
      <p>I can now be swiped on!</p>
    </v-touch>
    <v-touch @rotate="rotateAThing" :rotate-options="{ pointers: 3 }">
      <p>Rotate me!</p>
    </v-touch>
  </div>
</template>
```

### Events

There are quite a few events, but they pretty much all do what they say on the tin.

* **Panning** - pan, panstart, panmove, panend, pancancel, panleft, panright, panup, pandown
* **Pinching** - pinch, pinchstart, pinchmove, pinchend, pinchcancel, pinchin, pinchout
* **Pressing** - press, pressup
* **Rotating** -rotate, rotatestart, rotatemove, rotateend, rotatecancel
* **Swiping** - swipe, swipeleft, swiperight, swipeup, swipedown
* **Tapping** - tap

### Methods

```v-touch``` components expose a few methods as well.

* **enable(eventName)** will enable the recognizer for the specified event.
* **disable(eventName)** will disable the recognizer for the specified event.
* **toggle(eventName)** will toggle the enabled state of the recognizer.
* **disableAll()** and **enableAll()** will disable and enable all recognizers for the component.
* **isEnabled(eventName)**: Boolean returns the enabled state of a recognizer.

``` javascript
<template>
  <v-touch ref="swiper" @swipe="handleSwipe">
    <p>Swiper, no swiping!</p>
  </v-touch>
</template>

<script>
export default {
  mounted() {
    this.$refs.swiper.disable('swipe')
  }
}
</script>
```

### Config

There’s a bit more you can do in the way of configuration.

* You can enable and disable all recognizers with the :enabled=”boolean” prop, or pass an object to enable and disable them individually.
* There are a few default options that you can pass with the :options=”{}” prop.
* You can also register new events (really just events with default options) using VueTouch.registerCustomEvent(eventName, config). This must be called before Vue.use(VueTouch) and can be used like any other event from vue-touch.

``` javascript
// A custom horizontal swipe event.
VueTouch.registerCustomEvent('horizontal-swipe', {
  type: 'swipe',
  direction: 'horizontal'
})
```

There you go. Have fun adding random gestures and whatnot to your fancy little app-y things.

## Lazy Loading Images with Vue.js Directives and Intersection Observer

_Published: Oct 15, 2018_ by [Mateusz Rybczonek](https://css-tricks.com/author/mateuszrybczonek/)

Source: [CSS-Tricks](https://css-tricks.com/lazy-loading-images-with-vue-js-directives-and-intersection-observer/)

When I think about web performance, the first thing that comes to my mind is how images are generally the last elements that appear on a page. Today, images can be a major issue when it comes to performance, which is unfortunate since the speed a website loads has a direct impact on users successfully doing what they came to the page to do (think conversation rates).

Very recently, Rahul Nanwani wrote up an extensive [guide on lazy loading images](https://css-tricks.com/the-complete-guide-to-lazy-loading-images/). I’d like to cover the same topic, but from a different approach: using data attributes, [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) and [custom directives in Vue.js](https://vuejs.org/v2/guide/custom-directive.html).

What this’ll basically do is allow us to solve two things:

1. Store the `src` of the image we want to load without loading it in the first place.
2. Detect when the image becomes visible to the user and trigger the request to load the image.

Same basic lazy loading concept, but another way to go about it.


I created an example, based on [an example](https://codesandbox.io/s/5v17x4zr64) described by Benjamin Taylor in [his blog post](https://benjamintaylorportfolio.netlify.com/?utm_campaign=Revue%20newsletter&utm_medium=Newsletter&utm_source=Vue.js%20Developers#/post/lazy-loading-images-with-vue-js-directives). It contains a list of random articles each one containing a short description, image, and a link to the source of the article. We will go through the process of creating a component that is in charge of displaying that list, rendering an article, and lazy loading the image for a specific article.

![img](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1538150857/vue-lazy-load-03_cdn0ir.png)

Let’s get lazy! Or at least break this component down piece-by-piece.

### Step 1: Create the ImageItem component in Vue

Let’s start by creating a component that will show an image (but with no lazy loading involved just yet). We’ll call this file `ImageItem.vue`. In the component template, we’ll use a `figure` tag that contains our image — the image tag itself will receive the `src` attribute that points to the source URL for the image file.

``` html
<template>
  <figure class="image__wrapper">
    <img
      class="image__item"
      :src="source"
      alt="random image"
    >
  </figure>
</template>
```

In the script section of the component, we receive the prop `source` that we’ll use for the `src` url of the image we are displaying.

``` javascript
export default {
  name: "ImageItem",
  props: {
    source: {
      type: String,
      required: true
    }
  }
}
```

All this is perfectly fine and will render the image normally as is. But, if we leave it here, the image will load straight away without waiting for the entire component to be render. That’s not what we want, so let’s go to the next step.

### Step 2: Prevent the image from being loaded when the component is created

It might sound a little funny that we want to prevent something from loading when we want to show it, but this is about loading it at the right time rather than blocking it indefinitely. To prevent the image from being loaded, we need to get rid of the `src` attribute from the `img` tag. But, we still need to store it somewhere so we can make use of it when we want it. A good place to keep that information is in a [data- attribute](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes). These allow us to store information on standard, semantic HTML elements. In fact, you may already be accustomed to using them as JavaScript selectors.

In this case, they’re a perfect fit for our needs!

``` html{7}
<!--ImageItem.vue-->
<template>
  <figure class="image__wrapper">
    <img
      class="image__item"
      :data-url="source"
      alt="random image"
    >
  </figure>
</template>
```

With that, our image will not load because there is no source URL to pull from.

That’s a good start, but still not quite what we want. We want to load our image under specific conditions. We can request the image to load by replacing the `src` attribute with the image source URL kept in our `data-url attribute`. That’s the easy part. The real challenge is figuring out when to replace it with the actual source.

Our goal is to pin the load to the user’s screen location. So, when the user scrolls to a point where the image comes into view, that’s where it loads.

How can we detect if the image is in view or not? That’s our next step.

### Step 3: Detect when the image is visible to the user

You may have experience using JavaScript to determine when an element is in view. You may also have experience winding up with some gnarly script.

For example, we could use events and event handlers to detect the scroll position, offset value, element height, and viewport height, then calculate whether an image is in the viewport or not. But that already sounds gnarly, doesn’t it?

But it could get worse. This has direct implications on performance. Those calculations would be fired on every scroll event. Even worse, imagine a few dozen images, each having to recalculate whether it is visible or not on each scroll event. _Madness!_

[Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) to the rescue! This provides a very efficient way of detecting if an element is visible in the viewport. Specifically, it allows you to configure a **callback** that is triggered when one element — called the **target** — intersects with either the device viewport or a specified element.

So, what we need to do to use it? A few things:

* Create a new intersection observer
* Watch the element we wish to lazy load for visibility changes
* Load the element when the element is in viewport (by replacing `src` with our `data-url`)
* Stop watching for visibility changes (`unobserve`) after the load completes

Vue.js has custom directives to wrap all this functionality together and use it when we need it, as many times as we need it. Putting that to use is our next step.

### Step 4: Create a Vue custom directive

What is a custom directive? Vue’s [documentation](https://vuejs.org/v2/guide/custom-directive.html) describes it as a way to get low-level DOM access on elements. For example, changing an attribute of a specific DOM element which, in our case, could be changing the `src` attribute of an `img` element. Perfect!

``` javascript
export default {
  inserted: el => {
    function loadImage() {
      const imageElement = Array.from(el.children).find(
      el => el.nodeName === "IMG"
      );
      if (imageElement) {
        imageElement.addEventListener("load", () => {
          setTimeout(() => el.classList.add("loaded"), 100)
        });
        imageElement.addEventListener("error", () => console.log("error"))
        imageElement.src = imageElement.dataset.url
      }
    }

    function handleIntersect(entries, observer) {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          loadImage()
          observer.unobserve(el)
        }
      })
    }

    function createObserver() {
      const options = {
        root: null,
        threshold: "0"
      }
      const observer = new IntersectionObserver(handleIntersect, options)
      observer.observe(el)
    }
    if (window["IntersectionObserver"]) {
      createObserver()
    } else {
      loadImage()
    }
  }
}
```

OK, let’s tackle this step-by-step.

The **[hook function](https://vuejs.org/v2/guide/custom-directive.html#Hook-Functions)** allows us to fire a custom logic at a specific moment of a bound element lifecycle. We use the `inserted` hook because it is called when the bound element has been inserted into its parent node (this guarantees the parent node is present). Since we want to observe visibility of an element in relation to its parent (or any ancestor), we need to use that hook.

``` javascript
export default {
  inserted: el => {
    ...
  }
}
```

The `loadImage` function is the one responsible for replacing the `src` value with `data-url`. In it, we have access to our element (`el`) which is where we apply the directive. We can extract the `img` from that element.

Next, we check if the image exists and, if it does, we add a listener that will fire a callback function when the loading is finished. That callback will be responsible for hiding the spinner and adding the animation (fade-in effect) to the image using a CSS class. We also add a second listener that will be called in the event that the URL fails to load.

Finally, we replace the `src` of our `img` element with the source URL of the image and show it!

``` javascript
function loadImage() {
  const imageElement = Array.from(el.children).find(
    el => el.nodeName === "IMG"
  )
  if (imageElement) {
    imageElement.addEventListener("load", () => {
      setTimeout(() => el.classList.add("loaded"), 100)
    })
    imageElement.addEventListener("error", () => console.log("error"))
    imageElement.src = imageElement.dataset.url
  }
}
```

We use Intersection Observer’s `handleIntersect` function, which is responsible for firing `loadImage` when certain conditions are met. Specifically, it is fired when Intersection Observer detects that the element enters the viewport or a parent component element.

The function has access to `entries`, which is an array of all elements that are watched by the observer and `observer` itself. We iterate through `entries` and check if a single entry becomes visible to our user with `isIntersecting` — and fire the `loadImage` function if it is. Once the image is requested, we `unobserve` the element (remove it from the observer’s watch list), which prevents the image from being loaded again. And again. And again. And…

``` javascript
function handleIntersect(entries, observer) {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadImage()
      observer.unobserve(el)
    }
  })
}
```

The last piece is the [createObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#Creating_an_intersection_observer) function. This guy is responsible for creating our `Intersection Observer` and attaching it to our element. The IntersectionObserver constructor accepts a callback (our `handleIntersect` function) that is fired when the observed element passes the specified `threshold` and the `options` object that carries our observer options.

Speaking of the `options` object, it uses `root` as our reference object, which we use to base the visibility of our watched element. It might be any ancestor of the object or our browser viewport if we pass `null`. The object also specifies a `threshold` value that can vary from `0` to `1` and tells us at what percent of the target’s visibility the `observer` callback should be executed, with `0` meaning as soon as even one pixel is visible and `1` meaning the whole element must be visible.

And then, after creating the Intersection Observer, we attach it to our element using the `observe` method.

``` javascript
function createObserver() {
  const options = {
    root: null,
    threshold: "0"
  }
  const observer = new IntersectionObserver(handleIntersect, options)
  observer.observe(el)
}
```

### Step 5: Registering directive

To use our newly created directive, we first need to register it. There are two ways to do it: globally (available everywhere in the app) or locally (on a specified component level).

#### Global registration

For global registration, we import our directive and use the `Vue.directive` method to pass the name we want to call our directive and directive itself. That allows us to add a `v-lazyload` attribute to any element in our code.

``` javascript
// main.js
import Vue from "vue"
import App from "./App"
import LazyLoadDirective from "./directives/LazyLoadDirective"

Vue.config.productionTip = false

Vue.directive("lazyload", LazyLoadDirective)

new Vue({
  el: "#app",
  components: { App },
  template: "<App/>"
})
```

#### Local registration

If we want to use our directive only in a specific component and restrict the access to it, we can register the directive locally. To do that, we need to import the directive inside the component that will use it and register it in the `directives` object. That will give us the ability to add a `v-lazyload` attribute to any element in that component.

``` javascript
import LazyLoadDirective from "./directives/LazyLoadDirective"

export default {
  directives: {
    lazyload: LazyLoadDirective
  }
}
```

### Step 6: Use a directive on the ImageItem component

Now that our directive has been registered, we can use it by adding `v-lazyload` on the parent element that carries our image (the `figure` tag in our case).

``` html
<template>
  <figure v-lazyload class="image__wrapper">
    <ImageSpinner
      class="image__spinner"
    />
    <img
      class="image__item"
      :data-url="source"
      alt="random image"
    >
  </figure>
</template>
```

### Browser Support

We’d be remiss if we didn’t make a note about browser support. Even though the Intersection Observe API it is not supported by _all_ browsers, it does cover 73% of users (as of this writing).

Not bad. Not bad at all.

But! Having in mind that we want to show images to _all_ users (remember that using `data-url` prevents the image from being loaded at all), we need to add one more piece to our directive. Specifically, we need to check if the browser supports Intersection Observer, and it it doesn’t, fire `loadImage` instead. This will be our fallback.

``` javascript
if (window["IntersectionObserver"]) {
    createObserver()
} else {
    loadImage()
}
```

### Summary

Lazy loading images can _significantly_ improve page performance because it takes the page weight hogged by images and loads them in only when the user actually needs them.

For those still not convinced if it is worth playing with lazy loading, here’s some raw numbers from the [simple example](https://codesandbox.io/s/5v17x4zr64) we’ve been using. The list contains 11 articles with one image per article. That’s a total of 11 images (math!). It’s not like that’s a _ton_ of images but we can still work with it.

Here’s what we get rending all 11 images without lazy loading on a 3G connection:

![img](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1538150550/vue-lazy-load-01_qwhpsz.png)

The 11 image requests contribute to an overall page size of 3.2 MB. _Oomph_.

Here’s the same page putting lazy loading to task:

![img](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1538150560/vue-lazy-load-02_sslefg.png)

Say what? Only one request for one image. Our page is now 1.4 MB. We saved 10 requests and **reduced the page size by 56%**.

Is it a simple and isolated example? Yes, but the numbers still speak for themselves. Hopefully you find lazy loading an effective way to fight the battle against page bloat and that this specific approach using Vue with Intersection Observer comes in handy.

## Using SVG Icons

_Published: March 21, 2017_

Source: [Alligator.io](https://alligator.io/vuejs/using-svg-icons/)

> While font-based icons ruled the world a year or two ago, embedded SVG icons have since taken the stage (often credited to [this post](https://css-tricks.com/icon-fonts-vs-svg/)) as the best way to include icons in your app. Unfortunately, adding them by hand requires a lot of work and duplicated effort. Thankfully, [vue-svgicon](https://github.com/MMF-FE/vue-svgicon) aims to simplify this and does a wonderful job.

Let’s get started with it shall we?

### Installation & Setup

`vue-svgicon` can be installed via Yarn or NPM as expected:

``` bash
$ yarn add vue-svgicon -D
```

Next, download your preferred Icon font with individual SVG icons per-glyph. I’ll be using the [Material Design Icons set](https://github.com/Templarian/MaterialDesign/tree/master/icons/svg), one of the largest sets available with almost 2,000 individual glyphs, based on Google’s design guidelines but sourced mostly from the community.

Stick those `.svg` files in an `svg-icons` folder at the root of your project. (Outside of the `src` directory.)

Now, `vue-svgicon` needs to convert all of the svg icon files into `.js` files that can be individually loaded, so let’s add a quick **NPM script** to do this for us. Edit your `package.json` file to include the script below:

``` json
{
  ...
  "scripts": {
    ...
    "generate-icons": "vsvg -s ./svg-icons -t ./src/compiled-icons"
  }
  ...
}
```

Then issue the command `npm run generate-icons`. This will output the compiled icons at _src/compiled-icons/[icon-name].js_.

### Usage

Now we need to load the icons in our app. Add the **vue-svgicon** plugin to your main Vue app file:

``` javascript
import VueSVGIcon from 'vue-svgicon'
Vue.use(VueSVGIcon)
```

Now, we can load icons in our components by using the `<svgicon>` element and importing the icon we’re using. As a wonderful bonus, by using **SVG icons** in this way, we can load only the icons we need in the app with almost no effort.

``` javascript
<template>
  <div>
    <span>Icon Demonstration:</span>
    <!-- You can tweak the width, height, and color of the icon. -->
    <svgicon icon="menu" width="22" height="18" color="#0f2"></svgicon>
  </div>
</template>

<script>
// If you really, really need to, you can import the whole iconset in your main.js file with `import ./compiled-icons`.
import './compiled-icons/menu';
</script>
```

There are a few other neat little tricks v**ue-svgicon** has up its sleeve, find out more at the [official repository](https://github.com/MMF-FE/vue-svgicon).


## Writing Animations That Bring Your Site to Life

_Published: Feb 22, 2019_

Source: [CSS-Tricks](https://css-tricks.com/writing-animations-that-bring-your-site-to-life/)

* [Live Demo](https://pb03.github.io/css-animations-demo/)
* [GitHub Repo](https://github.com/pb03/css-animations-demo)

Web animation is one of the factors that can strongly enhance your website’s look and feel. Sadly, unlike mobile apps, there aren’t as many websites using animation to their benefit as you would think. We don’t want to count yours among those, so this article is for you and anyone else looking for ways to use animation for a better user experience! Specifically, we’re going to learn how to make web interactions delightful using CSS animations.


Here’s what we’re going to build together:

[Video](https://css-tricks.com/wp-content/uploads/2019/02/demo.mov)

Before we move ahead, it’s worth mentioning that I’m going to assume you have at least some familiarity with modern front-end frameworks and a basic understanding of CSS animations. If you don’t, then no fear! CSS-Tricks has a great guides on [React](https://css-tricks.com/guides/react/) and [Vue](https://css-tricks.com/guides/react/), as well as a thorough almanac post on the [CSS](https://css-tricks.com/almanac/properties/a/animation/) [animation property](https://css-tricks.com/almanac/properties/a/animation/).

Good? OK, let’s talk about why we’d want to use animation in the first place and cover some baseline information about CSS animations.

### Why would we to animate anything?

We could probably do an entire post on this topic alone. Oh, wait! [Sarah Drasner already did that](https://css-tricks.com/the-importance-of-context-shifting-in-ux-patterns/) and her points are both poignant and compelling.

But, to sum things up based on my own experiences:

* Animations enhance the way users interact with an interface. For example, smart animations can reduce cognitive load by giving users better context between page transitions.
* They can provide clear cues to users, like where we want them to focus attention.
* Animations serve as another design pattern in and of themselves, helping users to get emotionally attached to and engage with the interface.
* Another benefit of using animations is that they can create a perception that a site or app loads faster than it actually does.

### A couple of house rules with animations

Have you ever bumped into a site that animates **all the things**? Wow, those can be jarring. So, here’s a couple of things to avoid when working with animations so our app doesn’t fall into the same boat:

Avoid animating CSS properties other than ```transform``` and ```opacity```. If other properties have to be animated, like width or height, then make sure there aren’t a lot of layout changes happening at the same time. There’s actually a cost to animations and you can see exactly how much by referring to [CSS Triggers](https://csstriggers.com/).
Also, just because animations can create perceived performance gains, there’s actually a point of diminishing return when it comes to using them. Animating too many elements at the same time may result in decreased performance.
Now we can get our hands dirty with some code!

### Let’s build a music app

We’re going to build the music app we looked at earlier, which is inspired by [Aurélien Salomon’s Dribbble shot](https://dribbble.com/shots/5527602-Apple-Music-Smooth-floating). I chose this example so that we can focus on animations, not only within a component, but also between different routes. We’ll build this app using Vue and create animations using vanilla (i.e. no framework) CSS.

::: tip Pro tip!
Animations should go hand-in-hand with UI development. Creating UI before defining their movement is likely to cost much more time. In this case, the Dribbble shot provides that scope for us.
:::

Let’s start with the development.


#### Step 1: Spin up the app locally

First things first. We need to set up a new Vue project. Again, we’re assuming some base-level understanding of Vue here, so please check out the [Learning Vue guide](https://css-tricks.com/guides/vue/) for more info on setting up.

We need a couple of dependencies for our work, notably ```vue-router``` for transitioning between views and ```sass-loader``` so we can write in Sass and compile to CSS. Here’s a detailed [tutorial on using routes](https://css-tricks.com/build-a-custom-vue-router/) and Sass can be installed by pointing the command line at the project directory and using ```npm install -D sass-loader node-sass```.

We have what we need!

#### Step 2: Setting up routes

For creating routes, we’re first going to create two bare minimum components — ```Artists.vue``` and ```Tracks.vue```. We’ll drop a new file in the ```src``` folder called ```router.js``` and add routes for these components as:
``` javascript
import Vue from 'vue'
import Router from 'vue-router'
import Artists from './components/Artists.vue'
import Tracks from './components/Tracks.vue'

Vue.use(Router)
export default new Router({
	mode: 'history',
	routes: [
		{
			path: '/',
			name: 'artists',
			component: Artists
		},
		{
			path: '/:id',
			name: 'tracks',
			component: Tracks
		}
	]
})
```

Import ```router.js``` into the ```main.js``` and inject it to the Vue instance. Lastly, replace the content of your ```App.vue``` by ```<router-view/>```.

#### Step 3: Create the components and content for the music app

We need two components that we’ll transition between with animation. Those are going to be:

1. ```Artists.vue``` a grid of artists
2. ```Tracks.vue``` an artist image with a back button

If you wanna jump ahead a bit, here are some assets to work with:

1. [Images](https://github.com/pb03/css-animations-demo/raw/reference-code/assets.zip) and sample [data](https://github.com/pb03/css-animations-demo/tree/reference-code/data) in JSON format.
2. [Content](https://github.com/pb03/css-animations-demo/tree/reference-code/components) for the components

When all is said and done, the two views will come out to something like this:

![img](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1550533879/music-app-screens_gwhsjq.png)

_Artists.vue (left) and Tracks.vue (right)_

#### Step 4: Animate!

Here we are, the part we’ve really wanted to get to all this time. The most important animation in the app is transitioning from Artists to Tracks when clicking on an artist. It should feel seamless where clicking on an artist image puts that image in focus while transitioning from one view into the next. This is exactly the type of animation that we rarely see in apps but can drastically reduce cognitive load for users.

To make sure we’re all on the same page, we’re going to refer to the first image in the sequence as the “previous” image and the second one as the "current" image. Getting the effect down is relatively easy as long as we know the dimensions and position of the previous image in the transition. We can animate the current image by transforming it as per previous image.

The formula that I’m using is ```transform: translate(x, y) scale(n)```, where ```n``` is equal to the size of previous image divided by the size of current image. Note that we can use a static value of ```n``` since the dimensions are fixed for all the images. For example, the image size in the Artists view is ```190x190``` and ```240x240``` in the Tracks view. Thus, we can replace ```n``` by ```190/240 = 0.791```. That means the transform value becomes ```translate(x, y) scale(0.791)``` in our equation.

![img](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1550533981/music-app-animation-diagram_nkjwbx.png)

_Animating from Artists to Tracks_

Next thing is to find ```x``` and ```y```. We can get these values though click event in the Artists view as:

``` javascript
const {x, y} = event.target.getBoundingClientRect()
```

...and then send these values to the Tracks view, all while switching the route. Since we aren’t using any state management library, the two components will communicate via their parent component, which is the top level component, ```App.vue```. In ```App.vue```, let’s create a method that switches the route and sends the image info as params.

``` javascript
gotoTracks(position, artistId) {
	this.$router.push({
		name: 'tracks',
		params: {
			id: artistId,
			position: position
		}
	})
}
```

[https://github.com/pb03/css-animations-demo/blob/reference-code/App.vue](Here’s the relevant code) from the repo to reference, in case you’re interested.

Since we have received the position and ID of the image in Tracks, we have all the required data to show and animate it. We’ll first fetch artist information (specifically the name and image URL) using artist ID.

To animate the image, we need to calculate the ```transform``` value from the image’s starting position. To set the ```transform``` value, I’m using CSS custom properties, which can be done with CSS-in-JS techniques as well. Note that the image’s position that we received through props will be relative to window. Therefore we’ll have to subtract some fixed offset caused by the padding of the container ```<div>``` to even out our math.

``` javascript
const { x, y } = this.$route.params.position
// padding-left
const offsetLeft = 100
// padding-top
const offsetTop = 30

// Set CSS custom property value
document.documentElement.style.setProperty(
	'--translate', 
	`translate(${x - offsetLeft}px, ${y - offsetTop}px) scale(0.792)`
)
```

We’ll use this value to create a keyframe animation to move the image:

``` scss
@keyframes move-image {
	from {
		transform: var(--translate);
	}
}
```

This gets assigned to the CSS animation:

``` scss
.image {
	animation: move-image 0.6s;
}
```

...and it will animate the image from this transform value to its original position on component load.

![image](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_600,f_auto,q_auto/v1550534025/s_D931AB62E1E68F47894D0A090E9501B59CC83182583371FA7209E5A460E13D9D_1548265695511_1_iqbhne.gif)

_Transitioning from Artists to Tracks_

We can use the same technique when going the opposite direction, Tracks to Artists. As we already have the clicked image’s position stored in the parent component, we can pass it to props for Artists as well.

![image](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_600,f_auto,q_auto/v1550534052/s_D931AB62E1E68F47894D0A090E9501B59CC83182583371FA7209E5A460E13D9D_1548265710457_2_hdq2gd.gif)

_Transitioning from Tracks to Artists_

#### Step 5: Show the tracks!

It’s great that we can now move between our two views seamlessly, but the Tracks view is pretty sparse at the moment. So let’s add the track list for the selected artist.

We’ll create an empty white box and a new keyframe to slide it upwards on page load. Then we’ll add three subsections to it: Recent Tracks, Popular Tracks, and Playlist. Again, if you want to jump ahead, feel free to either [reference or copy the final code](https://github.com/pb03/css-animations-demo/blob/reference-code/components/Tracks--full.vue) from the repo.

![image](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_1000,f_auto,q_auto/v1550534086/music-app-tracks-content_hnedyn.png)

_The Tracks view with content_

Recent Tracks is the row of thumbnails just below the artist image where each thumbnail includes the track name and track length below it. Since we’re covering animations here, we’ll create a scale-up animation, where the image starts invisible (```opacity: 0```) and a little smaller than it’s natural size (```scale(0.7)```), then is revealed (```opacity: 1```) and scales up to its natural size (```transform: none```).

``` scss
.track {
	opacity: 0;
	transform: scale(0.7);
	animation: scale-up 1s ease forwards;
}

@keyframes scale-up {
	to {
		opacity: 1;
		transform: none;
	}
}
```

The Popular Tracks list and Playlist sit side-by-side below the Recent Tracks, where Popular tracks takes up most of the space. We can slide them up a bit on initial view with another set of keyframes:

``` scss
.track {
	...
	animation: slide-up 1.5s;
}

@keyframes slide-up {
	from {
		transform: translateY(140px);
	}
}
```

To make the animation feel more natural, we’ll create a stagger effect by adding an incremental animation delay to each item.

``` scss
@for $i from 1 to 5 {
	&:nth-child(#{$i + 1}) {
		animation-delay: #{$i * 0.05}s;
	}
}
```

The code above is basically looking for each child element, then adding a 0.05 second delay to each element it finds. So, for example, the first child gets a 0.05 second delay, the second child gets a 0.10 second delay and so on.

Check out how nice and natural this all looks:

[Video](https://css-tricks.com/wp-content/uploads/2019/02/content.mov)


### Bonus: micro-interactions!

One of the fun things about working with animations is thinking through the small details because they’re what tie things together and add delight to the user experience. We call these micro-interactions and they serve a good purpose by providing visual feedback when an action is performed.

Depending on the complexity of the animations, we might need a library like [anime.js](https://animejs.com/) or [GSAP](https://greensock.com/gsap). This example is pretty straightforward, so we can accomplish everything we need by writing some CSS.

#### First micro-interaction: The volume icon

Let’s first get a volume icon in SVG format ([Noun Project](https://thenounproject.com/search/?q=heart) and [Material Design](https://material.io/tools/icons/?icon=favorite&style=baseline) are good sources). On click, we’ll animate-in and out its ```path``` element to show the level of volume. For this, we’ll create a method which switches its CSS class according to the volume level.

``` html
<svg @click="changeVolume">
	<g :class="`level-${volumeLevel}`">
		<path d="..."/> <!-- volume level 1 -->
		<path d="..."/> <!-- volume level 2 -->
		<path d="..."/> <!-- volume level 3 -->
		<polygon points="..."/>
	</g>
</svg>
```

Based on this class, we can show and hide certain ```path``` elements as:

``` scss
path {
	opacity: 0;
	transform-origin: left;
	transform: translateX(-5px) scale(0.6);
	transition: transform 0.25s, opacity 0.2s;
}

.level-1 path:first-child,
.level-2 path:first-child,
.level-2 path:nth-child(2),
.level-3 path {
	opacity: 1;
	transform: none;
}
```

![image](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_900,f_auto,q_auto/v1550534497/music-app-volume_q9xlm2.gif)

_The animated volume control_

#### Second micro-interaction: The favorite icon

Do you like it when you click on Twitter’s heart button? That’s because it feels unique and special by the way it animates on click. We’ll make something similar but real quick. For this, we first get an SVG heart icon and add it to the the markup. Then we’ll add a bouncy animation to it that’s triggered on click.

``` scss
@keyframes bounce {
	0%, 100% {
		transform: none;
	}
	30% {
		transform: scale(1.3);
	}
	60% {
		transform: scale(0.9);
	}
}
```

Another fun thing we can do is add other small heart icons around it with random sizes and positions. Ideally, we’d add a few ```absolute```-positioned HTML elements that a heart as the background. Let’s Arrange each of them as below by setting their ```left``` and ```bottom``` values.

We’ll also include a fade away effect so the icons appear to dissolve as they move upward by adding a keyframe animation on the same click event.

``` scss
@keyframes float-upwards {
	0%, 100% {
		opacity: 0;
	}
	50% {
		opacity: 0.7;
	}
	50%, 100% {
		transform: translate(-1px, -5px);
	}
}
```

![image](https://res.cloudinary.com/css-tricks/image/upload/c_scale,w_900,f_auto,q_auto/v1550534803/music-app-favorite_wklb20.gif)

_The animated favorite button_

### Summing up

That’s all! I hope you find all this motivating to try animations on your own websites and projects.

While writing this, I also wanted to expand on the fundamental animation principles we glossed over earlier because I believe that they help choose animation durations, and avoid non-meaningful animations. That’s important to discuss because doing animations **correctly** is better than doing them at all. But this sounds like a whole another topic to be covered in a future article.