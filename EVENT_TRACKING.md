# Using the ESPI Card Builder event emitter

In this short documentation, you will learn how to use the event dispatcher from the ESPI Card Builder to plug it any tracking system. 

* [The basics](#the-basics)
* [Listening for events](#listening-for-events)
  * [Using ESI](#using-esi)
  * [Using embedded iframe](#using-embedded-iframe)
  * [Using both ESI and embedded iframe](#using-both-esi-and-embedded-iframe)
  * [Listening with multiple stacks](#listening-with-multiple-stacks)
* [Anatomy of an event](#anatomy-of-an-event)
* [List of emitted events](#list-of-emitted-events)
* [Connecting to analytics providers](#connecting-to-analytics-providers)
  * [Google Analytics](#google-analytics)
  * [Piwik](#piwik)
  * [Net-Metrix](#net-metrix)

## The basics

You should always first listen to the `'window.cardstacks.ready'` event fired on the `window` object when the card stack is finally ready. At this point, you can start listening for events.

```js
window.addEventListener('window.cardstacks.ready', function () {
  // Card stack is ready; start listening for events.
});
```

## Listening for events

### Using ESI

In order to make the event listening process easy, the ESPI Card Builder only emits a single type of event, called `cardEvent` which contains different data based on what fired the event in the first place (card swiping, jump between cards, keyboard usage, etc.).

Although to make sure everything is correctly namespaced, the ESPI Card Builder does not fire these events on the global scope but on the `cardstacks.events` object from the `window` object. You can then listen for events as such:

```js
window.cardstacks.events.on('cardEvent', function (event) {
  // `event` contains the event data
});
```

Theoretically, you do not have to make sure that the `cardstacks` (and in a further extend, the `events`) object exist as it is ensured when first listening to `'window.cardstacks.ready'`.

### Using embedded iframe

The ESPI Card Builder emits events in its own window, making the capture of these events in the parent window (your page) slightly different when using an iframe to embed a stack.

To make the events *bubble up* in the parent window, the ESPI Card Builder is using the [PostMessage API](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage).

You then have to listen to the `'message'` event on the `window` object, and from them treat the event stored in `event.data`.

```js
window.addEventListener('message', function (event) {
  // `event.data` contains the ESPI Card Builder event
  // (provided that the PostMessage comes for a stack)
});
```

**Important security notice:** it is highly recommended that you check the origin of the event PostMessage event before processing it. To do so, you can perform a check against `event.origin`, making sure it comes from the ESPI Card Builder hosting:

```js
window.addEventListener('message', function (event) {
  // Do not proceed if it does not come from ESPI
  if (event.origin !== 'http://cards.edenspiekermann.com') {
    return;
  }

  // `event.data` contains the ESPI Card Builder event
});
```

Failure to check the `origin` property enables cross-site scripting attacks and Edenspiekermann will not be responsible for this. Make sure it comes from the correct domain before proceeding.

### Using both ESI and embedded iframes

If you want to make your code agnostic no matter whether you use ESI rendering or embedded iframes, you can build a small function to handle this:

```js
(function (callback) {
  window.addEventListener('window.cardstacks.ready', function () {
    window.addEventListener('message', function (event) {
      if (event.origin !== 'http://cards.edenspiekermann.com') {
        return;
      }

      callback(event.data);
    });

    window.addEventListener('cardEvent', callback);
  });
}(function onStackEvent (event) {
  // Do something with the event here
});
```

### Listening with multiple stacks

If you happen to have several stacks embedded in the same page (with ESI or iframes, it doesn’t matter), it works all the same on the listening side. You can know from which stack the event comes from by checking the `stackId` property from the event.

## Anatomy of an event

All events fired by the ESPI Card Builder look alike to make things easier. Therefore, an event is always shaped like this:

| Key       | Type          | Role                             |
|-----------|---------------|----------------------------------|
| eventType | string        | The name of the event            |
| stackId   | number        | The ID of the stack              |
| cardIndex | number        | The index of the card            |
| cardType  | string        | The type of the card             |
| cardCount | number        | The number of cards in the stack |
| eventData | object / null | Relevant extra event data        |

## List of emitted events

Each event fired by the ESPI Card Builder starts with the event category (*stack*, *card*, *media* or *share*). Below is the list of all the events fired by the ESPI Card Builder.

| eventType          | Occasion                                |
|--------------------|-----------------------------------------|
| stack.load         | When the stack has finished loading     |
| card.click.prev    | When clicking `←` button                |
| card.click.prev    | When clicking `←` button                |
| card.key.prev      | When pressing `←` key                   |
| card.key.next      | When pressing `→` key                   |
| card.swipe.prev    | When swiping left                       |
| card.swipe.next    | When swiping right                      |
| card.jump          | When using a Jump Card                  |
| card.choice.select | When clicking a choice on a Choice Card |
| share.url-overlay  | When opening the sharing overlay        |
| share.facebook     | When sharing the stack on Facebook      |
| share.twitter      | When sharing the stack on Twitter       |
| share.google       | When sharing the stack on Google        |
| share.whatsapp     | When sharing the stack on Whatsapp      |
| media.play         | When playing the media of a Media Card  |

## Connecting to analytics providers

The agnostic solution provided by the event dispatcher from the ESPI Card Builder makes it easy to connect it to any analytics provider on your side. The basic idea is to listen to event, and when captured, handle the data to the analytics provider.

For the sake of simplicity, in this section we will only care about a custom `onStackEvent` function accepting an event fired by a stack as a parameter. Please refer to [this section](#using-both-esi-and-embedded-iframe) to know how to use it efficiently.

### Google Analytics

```js
function onStackEvent (event) {
  var type = event.eventType;
  // e.g. `card`, `stack`, `share`, `media`
  var category = type.split('.')[0];
  // e.g. `swipe.next`, `play`
  var action = type.substr(type.indexOf('.') + 1);
  // e.g. `'{ "cardType": "choice", "cardIndex": 42 }'`
  var label = JSON.stringify({
    cardType: event.cardType,
    cardIndex: event.cardIndex
  });
  // e.g. `42`
  var value = event.stackId;

  try {
    ga('send', 'event', category, action, label, value);
  } catch (e) {
    console.error(e);
  }
}
```

## Piwik

```js
function onStackEvent (event) {
  var type = event.eventType;
  // e.g. `card`, `stack`, `share`, `media`
  var category = type.split('.')[0];
  // e.g. `swipe.next`, `play`
  var action = type.substr(type.indexOf('.') + 1);
  // e.g. `'{ "cardType": "choice", "cardIndex": 42 }'`
  var label = JSON.stringify({
    cardType: event.cardType,
    cardIndex: event.cardIndex
  });
  // e.g. `42`
  var value = event.stackId;

  try {
    _paq.push(['trackEvent', category, action, label, value]);
  } catch (e) {
    console.error(e);
  }
}
```

### Net-Metrix

The first thing you will need is to inject the initial tracking pixel and to provide a function to refresh it so it can be used on the ESPI Card Builder events. Here is a sample function, feel free to build your own:

```js
function injectNetMetrix () {
  var getNetmetrixSource = function () {
    return  'http://.wemfbox.ch/cgi-bin/ivw/CP'
    + '?r=' + escape(document.referrer)
    + '&d=' + (Math.random() * 100000)
    + '&x=' + screen.width + 'x' + screen.height;
  }

  var id = 'netmetrix-tracking-pixel';
  var image = document.getElementById(id);

  if (image) {
    image.src = getNetmetrixSource();
  } else {
    image = document.createElement('img');
    image.id = id;
    image.src = getNetmetrixSource();
    image.width = 1;
    image.height = 1;

    document.body.appendChild(image);
  }
}

// Initial tracking pixel injection
injectNetMetrix();
```

Now coming back to the `onStackEvent` function:

```js
function onStackEvent (event) {
  // Events triggering a new “page view”
  var whitelist = [
    'card.click.prev',
    'card.click.prev',
    'card.key.prev',
    'card.key.next',
    'card.swipe.prev',
    'card.swipe.next',
    'card.jump',
    'card.choice.select'
  ];

  if (whitelist.indexOf(event.eventType) > -1) {
    injectNetMetrix();
  }
}
```

If you desire to only track unique card changes, you can build the `onStackEvent` function as follow:

```js
function onStackEvent (event) {
  var indexed = [];
  var whitelist = [
    'card.click.prev',
    'card.click.prev',
    'card.key.prev',
    'card.key.next',
    'card.swipe.prev',
    'card.swipe.next',
    'card.jump',
    'card.choice.select'
  ];

  if (
    whitelist.indexOf(event.eventType) > -1
    && indexed.indexOf(event.cardIndex) === -1
  ) {
    injectNetMetrix();
    indexed.push(event.cardIndex);
  }
}
```
