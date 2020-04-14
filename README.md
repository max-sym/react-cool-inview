> 🚧 This package is in-progress, [API](#api) may be changed frequently. I don't recommend you to use it now. If you're willing to be an early user. Please note any changes via [release](https://github.com/wellyshen/react-cool-inview/releases). Here's the [milestone](#milestone).

# React Cool Inview

A React [hook](https://reactjs.org/docs/hooks-custom.html#using-a-custom-hook) that monitors an element enters or leaves the viewport (or another element) with performant and efficient way, using [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API). It's lightweight and super flexible, which can help you do many things, like [lazy-loading images](#lazy-loading-images) and videos, [infinite scrolling](#infinite-scrolling) web app, [triggering animations](#trigger-animations), [tracking impressions](#track-impressions) etc. Try it you will 👍🏻 it!

[![build status](https://img.shields.io/travis/wellyshen/react-cool-inview/master?style=flat-square)](https://travis-ci.org/wellyshen/react-cool-inview)
[![coverage status](https://img.shields.io/coveralls/github/wellyshen/react-cool-inview?style=flat-square)](https://coveralls.io/github/wellyshen/react-cool-inview?branch=master)
[![npm version](https://img.shields.io/npm/v/react-cool-inview?style=flat-square)](https://www.npmjs.com/package/react-cool-inview)
[![npm downloads](https://img.shields.io/npm/dm/react-cool-inview?style=flat-square)](https://www.npmtrends.com/react-cool-inview)
[![npm downloads](https://img.shields.io/npm/dt/react-cool-inview?style=flat-square)](https://www.npmtrends.com/react-cool-inview)
[![npm bundle size](https://img.shields.io/bundlephobia/minzip/react-cool-inview?style=flat-square)](https://bundlephobia.com/result?p=react-cool-inview)
[![MIT licensed](https://img.shields.io/github/license/wellyshen/react-cool-inview?style=flat-square)](https://raw.githubusercontent.com/wellyshen/react-cool-inview/master/LICENSE)
[![All Contributors](https://img.shields.io/badge/all_contributors-1-orange?style=flat-square)](#contributors-)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](https://github.com/wellyshen/react-cool-inview/blob/master/CONTRIBUTING.md)
[![Twitter URL](https://img.shields.io/twitter/url?style=social&url=https%3A%2F%2Fgithub.com%2Fwellyshen%2Freact-cool-inview)](https://twitter.com/intent/tweet?text=With%20@react-cool-inview,%20I%20can%20build%20a%20performant%20web%20app.%20Thanks,%20@Welly%20Shen%20🤩)

## Milestone

- [x] Detect an element is in-view or not
- [x] `onEnter`, `onLeave`, `onChange` events
- [x] Trigger once feature
- [x] Server-side rendering compatibility
- [x] Support [Intersection Observer v2](#intersection-observer-v2)
- [ ] Unit testing
- [ ] Demo app
- [ ] Demo code
- [ ] Typescript type definition
- [ ] Documentation

## Features

Coming soon...

## Requirement

To use `react-cool-inview`, you must use `react@16.8.0` or greater which includes hooks.

## Installation

This package is distributed via [npm](https://www.npmjs.com/package/react-cool-inview).

```sh
$ yarn add react-cool-inview
# or
$ npm install --save react-cool-inview
```

## Usage

`react-cool-inview` has a flexible [API](#api) design, it can cover simple to complex use cases for you. Here are some ideas for how you can use it.

> ⚠️ [Most modern browsers support Intersection Observer natively](https://caniuse.com/#feat=intersectionobserver). You can also [add polyfill](#intersection-observer-polyfill) for full browser support.

### Basic Use Case

To monitor an element enters or leaves the browser viewport by the `inView` state and useful sugar events.

```js
import React, { useRef } from 'react';
import useInView from 'react-cool-inview';

const App = () => {
  const ref = useRef();
  const { inView, entry } = useInView(ref, {
    threshold: 0.25, // Default is 0
    onEnter: ({ entry, observe, unobserve }) => {
      // Triggered when the target enters the browser viewport (start intersecting)
    },
    onLeave: ({ entry, observe, unobserve }) => {
      // Triggered when the target leaves the browser viewport (end intersecting)
    },
    // More useful options...
  });

  return <div ref={ref}>{inView ? 'Hello, I am 🤗' : 'Bye, I am 😴'}</div>;
};
```

### Lazy-loading Images

It's super easy to build an image lazy-loading component with `react-cool-inview` to boost the performance of your web app.

```js
import React, { useRef } from 'react';
import useInView from 'react-cool-inview';

const LazyImage = ({ width, height, ...rest }) => {
  const ref = useRef();
  const { inView } = useInView(ref, {
    // Stop observe when meet the threshold, so the "inView" only triggered once
    unobserveOnEnter: true,
    // For better UX, we can grow the root margin so the image will be loaded before it comes to the viewport
    rootMargin: '50px',
  });

  return (
    <div className="placeholder" style={{ width, height }} ref={ref}>
      {inView && <img {...rest} />}
    </div>
  );
};
```

> 💡 Looking for a comprehensive image component? Try [react-cool-img](https://github.com/wellyshen/react-cool-img), it's my other component library.

### Infinite Scrolling

Infinite scrolling is a popular design technique like Facebook and Twitter feed etc., new content being loaded as you scroll down a page. The basic concept as below.

```js
import React, { useRef, useState } from 'react';
import useInView from 'react-cool-inview';
import axios from 'axios';

const App = () => {
  const [todos, setTodos] = useState(['todo-1', 'todo-2', '...']);
  const ref = useRef();

  useInView(ref, {
    // For better UX, we can grow the root margin so the data will be loaded before a user sees the loading indicator
    rootMargin: '50px 0',
    // When the loading indicator comes to the viewport
    onEnter: ({ unobserve, observe }) => {
      // Pause observe when loading data
      unobserve();
      // Load more data
      axios.get('/todos').then((res) => {
        setTodos([...todos, ...res.todos]);
        // Resume observe after loading data
        observe();
      });
    },
  });

  return (
    <div>
      {todos.map((todo) => (
        <div>{todo}</div>
      ))}
      <div ref={ref}>Loading...</div>
    </div>
  );
};
```

Compare to pagination, infinite scrolling provides a seamless experience for users and it’s easy to see the appeal. But when it comes to render a large lists, performance will be a problem. We can use [react-window](https://github.com/bvaughn/react-window) to address the problem by the technique of [DOM recycling](https://developers.google.com/web/updates/2016/07/infinite-scroller).

```js
import React, { useRef, useState } from 'react';
import useInView from 'react-cool-inview';
import { FixedSizeList as List } from 'react-window';
import axios from 'axios';

const Row = ({ index, data, style }) => {
  const { todos, handleLoadingInView } = data;
  const isLast = index === todos.length;
  const ref = useRef();

  useInView(ref, { onEnter: handleLoadingInView });

  return (
    <div style={style} ref={isLast ? ref : null}>
      {isLast ? 'Loading...' : todos[index]}
    </div>
  );
};

const App = () => {
  const [todos, setTodos] = useState(['todo-1', 'todo-2', '...']);
  const [isFetching, setIsFetching] = useState(false);

  const handleLoadingInView = () => {
      // Row component is dynamically created by react-window, we need to use the "isFetching" flag
      // instead of unobserve/observe to avoid re-fetching data
      if (!isFetching)
        axios.get('/todos').then((res) => {
          setTodos([...todos, ...res.todos]);
          setIsFetching(false);
        });

      setIsFetching(true);
    },

  // Leverage the power of react-window to help us address the performance bottleneck
  return (
    <List
      height={150}
      itemCount={todos.length + 1} // Last one is for the loading indicator
      itemSize={35}
      width={300}
      itemData={{ todos, handleLoadingInView }}
    >
      {Row}
    </List>
  );
};
```

### Trigger Animations

Another great use case is to trigger CSS animations once they are visible to the users.

```js
import React, { useRef } from 'react';
import useInView from 'react-cool-inview';

const App = () => {
  const ref = useRef();
  const { inView } = useInView(ref, {
    // Stop observe when meet the threshold, so the "inView" only triggered once
    unobserveOnEnter: true,
    // Shrink the root margin, so the animation will be triggered once an element reach a fixed amount of visible
    rootMargin: '-100px 0',
  });

  return (
    <div className="container" ref={ref}>
      <div className={inView ? 'fade-in' : ''}>I'm a 🍟</div>
    </div>
  );
};
```

### Track Impressions

`react-cool-inview` can also play as an impression tracker, helps you fire an analytic event when a user sees an element or advertisement.

```js
import React, { useRef } from 'react';
import useInView from 'react-cool-inview';

const App = () => {
  const ref = useRef();
  useInView(ref, {
    // For an element to be considered "seen", we'll say it must be 100% in the viewport
    threshold: 1,
    onEnter: ({ unobserve }) => {
      // Stop observe when meet the threshold, so the "onEnter" only triggered once
      unobserve();
      // Fire an analytic event to your tracking service
      someTrackingService.send('🍋 is seen');
    },
  });

  return <div ref={ref}>I'm a 🍋</div>;
};
```

## Intersection Observer v2

The Intersection Observer v1 can perfectly tell you when an element is scrolled into the viewport, but it doesn't tell you whether the element is covered by something else on the page or whether the element has any visual effects applied on it (like `transform`, `opacity`, `filter` etc.) that can make it invisible. The main concern that has surfaced is how this kind of knowledge could be helpful in preventing [clickjacking](https://en.wikipedia.org/wiki/Clickjacking) and UI redress attacks (read this [article](https://developers.google.com/web/updates/2019/02/intersectionobserver-v2) to learn more).

If you want to track the click-through rate (CTR) or impression of an element, which is actually visible to a user, [Intersection Observer v2](https://developers.google.com/web/updates/2019/02/intersectionobserver-v2) can be the savior. Intersection Observer v2 introduces a new boolean field named `isVisible`. A `true` value guarantees that an element is visible on the page and has no visual effects applied on it. A `false` value is just the opposite. When using the v2, there're something we need to know:

- Check [browser compatibility](https://caniuse.com/#feat=intersectionobserver-v2), if a browser doesn't support Intersection Observer v2 the `isVisible` will be `undefined`.
- Understand [how visibility is calculated](https://w3c.github.io/IntersectionObserver/v2/#calculate-visibility-algo).
- Visibility is much more expensive to compute than intersection, only use it when needed.

To enable Intersection Observer v2, we need to set the `trackVisibility` and `delay` options.

```js
import React, { useRef } from 'react';
import useInView from 'react-cool-inview';

const App = () => {
  const ref = useRef();
  const { inView, isVisible } = useInView(ref, {
    // Track the actual visibility of the element
    trackVisibility: true,
    // Set a minimum delay between notifications, it must be set to 100 (ms) or greater
    // For performance perspective, use the largest tolerable value as much as possible
    delay: 100,
  });
  // If the browser doesn't support Intersection Observer v2, falling back to v1 behavior
  const visible = isVisible || inView;

  return <div ref={ref}>{visible ? 'Hello, I am 🤗' : 'Bye, I am 😴'}</div>;
};
```

## API

Coming soon...

## Intersection Observer Polyfill

[Intersection Observer has good support amongst browsers](https://caniuse.com/#feat=intersectionobserver), but it's not universal. You'll need to polyfill browsers that don't support it. Polyfills is something you should do consciously at the application level. Therefore `react-cool-inview` doesn't include it.

You can use W3C's [polyfill](https://www.npmjs.com/package/intersection-observer):

```sh
$ yarn add intersection-observer
# or
$ npm install --save intersection-observer
```

Then import it at your app's entry point:

```js
import 'intersection-observer';
```

Or load the polyfill only if needed:

```js
if (!window.IntersectionObserver) require('intersection-observer');
```

[Polyfill.io](https://polyfill.io/v3) is an alternative way to add the polyfill when needed.

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://wellyshen.com"><img src="https://avatars1.githubusercontent.com/u/21308003?v=4" width="100px;" alt=""/><br /><sub><b>Welly</b></sub></a><br /><a href="https://github.com/wellyshen/react-cool-inview/commits?author=wellyshen" title="Code">💻</a> <a href="https://github.com/wellyshen/react-cool-inview/commits?author=wellyshen" title="Documentation">📖</a> <a href="#maintenance-wellyshen" title="Maintenance">🚧</a></td>
  </tr>
</table>
<!-- markdownlint-enable -->
<!-- prettier-ignore-end -->
<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!
