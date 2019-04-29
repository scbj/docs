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