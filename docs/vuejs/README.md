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