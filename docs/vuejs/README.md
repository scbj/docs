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

<video width="560" height="240" controls>
  <source src="https://css-tricks.com/wp-content/uploads/2019/02/content.mov" type="video/mp4">
  Your browser does not support the video tag.
</video> 

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