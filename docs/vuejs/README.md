# Vue.js

## Global Event Bus

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

## Common Gotchas

Source: [Alligator.io](https://alligator.io/vuejs/common-gotchas/)

> As with any framework, Vue has a few oddities that might take newcomers a little while to get used to, and many stumble over. Here, we’ll attempt to list a number of those and how to work with and/or around them.

### Reactivity

Vue’s reactivity system is great, but it doesn’t handle quite everything. There are a few edge cases that Vue can’t detect. (Yet. Hopefully when ES6 Proxies are widely supported these caveats will be gone.)

* When properties are added or removed from an object, Vue won’t know about it and won’t make them reactive.
* You can’t add new properties to the root data object directly, but you can use `Vue.set(this.data, ‘propname’, value)`
* Vue can’t detect when a particular index value changes in an array from setting it directly through `array[index] = value`. The workaround is to use `Vue.set(array, index, value)`
* Vue can’t detect when the length of an array changes. Use splice instead.

## Using SVG Icons

Source: [Alligator.io](https://alligator.io/vuejs/global-event-bus/)

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

Source: [CSS-Tricks](https://css-tricks.com/writing-animations-that-bring-your-site-to-life/)

* [Live Demo](https://pb03.github.io/css-animations-demo/)
* [GitHub Repo](https://github.com/pb03/css-animations-demo)

Web animation is one of the factors that can strongly enhance your website’s look and feel. Sadly, unlike mobile apps, there aren’t as many websites using animation to their benefit as you would think. We don’t want to count yours among those, so this article is for you and anyone else looking for ways to use animation for a better user experience! Specifically, we’re going to learn how to make web interactions delightful using CSS animations.


Here’s what we’re going to build together:

<video width="560" height="240" controls>
  <source src="https://css-tricks.com/wp-content/uploads/2019/02/demo.mov" type="video/mp4">
  Your browser does not support the video tag.
</video> 

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
