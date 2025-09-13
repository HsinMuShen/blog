# React Timer Hooks: How useTimeout and useTimeoutFn Simplify Your Code

```
     /\_/\             ⏰ useTimeout     →     ⏰ useTimeoutFn
    ( •.• )           (Declarative)         (Imperative)
   / >⏰< \           |                     |
                      └─ Auto cleanup      └─ Manual control
                      └─ Single execution   └─ Start/Cancel/Reset
                      └─ Simple API         └─ isPending state
```

## Table of Contents

1. [Background](#1-background)
2. [Introducing useTimeout and useTimeoutFn](#2-introducing-usetimeout-and-usetimeoutfn)
3. [Benefits over setTimeout](#3-benefits-over-settimeout)
4. [Debounced Auto-Save Example](#4-debounced-auto-save-example)
5. [Implementing a Multi-Message Tooltip](#5-implementing-a-multi-message-tooltip)
6. [Summary](#summary)
7. [References](#7-references)

---

## 1. Background

In my recent work, I needed to build a tooltip component that could display a sequence of messages, each for exactly 5 seconds, before automatically flipping to the next one. At first I reached for the native `setTimeout` API, but juggling manual cleanup and restarting timers for multiple messages quickly became error-prone. My colleagues suggested switching to the React-friendly hooks **`useTimeout`** and **`useTimeoutFn`** instead of plain `setTimeout`—so I gave them a try.

## 2. Introducing `useTimeout` and `useTimeoutFn`

Both hooks come from the popular [react-use](https://www.npmjs.com/package/react-use) library.

### `useTimeout(callback, delay)`

Schedules a single run of `callback` after `delay` ms, and automatically cleans up the timer if your component unmounts or if you pass `null` as `delay`.

```ts
useTimeout(
  () => {
    console.log('Fires once after delay');
  },
  5000
);
```

### `useTimeoutFn(callback, delay, options?)`

Gives you manual control over a timer. It returns a tuple:

```ts
const [isPending, start, cancel] = useTimeoutFn(
  () => { /* your callback */ },
  3000,
  { immediate: false }
);
```

- **`isPending`**: `boolean` — whether a timer is currently scheduled
- **`start()`**: `void` — begin (or restart) the timer
- **`cancel()`**: `void` — clear the timer

**Options:**
- `immediate` (default `true`): start on mount or wait for you to call `start()`
- `updateOnEnd`: whether to trigger a re-render when the timer completes

## 3. Benefits over setTimeout

### Automatic cleanup
No more "zombie" timers firing after a component unmounts—cleanup is handled for you.

### Stable callback reference
Internally, these hooks store your latest callback in a ref, so the timer always invokes up-to-date code without redeclaring effects.

### Declarative or manual control
- `useTimeout` lets you declare "run this after X ms."
- `useTimeoutFn` gives you imperative methods (`start`/`cancel`) to control exactly when timers run or reset.

### Less boilerplate
No juggling refs, `useEffect` dependencies, or worrying about stale closures.

## 4. Debounced Auto-Save Example

Here's how you might build an autosave that fires 2 seconds after the user stops typing—a classic debounce pattern:

### With useTimeoutFn (Recommended)

```jsx
import { useTimeoutFn } from 'react-use';

function AutoSave({ text }) {
  const [isPending, start, cancel] = useTimeoutFn(
    () => api.save(text),
    2000,
    { immediate: false }
  );

  // Restart the timer on every `text` change
  useEffect(() => {
    cancel();
    start();
  }, [text, cancel, start]);

  return <textarea value={text} onChange={…} />;
}
```

### With setTimeout (Traditional approach)

```jsx
import { useEffect, useRef } from 'react';

function AutoSave({ text }) {
  const timeoutRef = useRef(null);

  useEffect(() => {
    // Clear any existing timer
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    // Set new timer
    timeoutRef.current = setTimeout(() => {
      api.save(text);
    }, 2000);

    // Cleanup on unmount or text change
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, [text]);

  return <textarea value={text} onChange={…} />;
}
```

**Comparison:**
- `useTimeoutFn` handles cleanup automatically
- `setTimeout` requires manual ref management and cleanup
- `useTimeoutFn` provides `isPending` state for UI feedback
- `setTimeout` requires additional state management for pending status

## 5. Implementing a Multi-Message Tooltip

For my tooltip feature, I needed to cycle through an array of messages, showing each for 5 seconds:

### With useTimeoutFn (Recommended)

```tsx
import { useState, useEffect } from 'react';
import { useTimeoutFn } from 'react-use';

function TooltipCarousel({ messages }: { messages: string[] }) {
  const [index, setIndex] = useState(0);
  const [isPending, start, cancel] = useTimeoutFn(
    () => setIndex(i => (i + 1) % messages.length),
    5000,
    { immediate: true } // or just omit this line since it's the default
  );

  // Restart the 5 s timer whenever `index` changes
  useEffect(() => {
    cancel();
    start();
  }, [index, cancel, start]);

  return <div className="tooltip">{messages[index]}</div>;
}
```

**Even simpler version (using default immediate: true):**

```tsx
import { useState, useEffect } from 'react';
import { useTimeoutFn } from 'react-use';

function TooltipCarousel({ messages }: { messages: string[] }) {
  const [index, setIndex] = useState(0);
  const [isPending, start, cancel] = useTimeoutFn(
    () => setIndex(i => (i + 1) % messages.length),
    5000
    // immediate: true is the default, so no need to specify it
  );

  // Restart the 5 s timer whenever `index` changes
  useEffect(() => {
    cancel();
    start();
  }, [index, cancel, start]);

  return <div className="tooltip">{messages[index]}</div>;
}
```

### With setTimeout (Traditional approach)

```tsx
import { useState, useEffect, useRef } from 'react';

function TooltipCarousel({ messages }: { messages: string[] }) {
  const [index, setIndex] = useState(0);
  const timeoutRef = useRef(null);

  // Start timer when index changes
  useEffect(() => {
    // Clear any existing timer
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    // Set new timer
    timeoutRef.current = setTimeout(() => {
      setIndex(i => (i + 1) % messages.length);
    }, 5000);

    // Cleanup on unmount
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, [index, messages.length]);

  // Start the first cycle on mount
  useEffect(() => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    timeoutRef.current = setTimeout(() => {
      setIndex(i => (i + 1) % messages.length);
    }, 5000);
  }, []);

  return <div className="tooltip">{messages[index]}</div>;
}
```

**Comparison:**
- `useTimeoutFn` provides cleaner, more readable code
- `setTimeout` requires manual timer management with refs
- `useTimeoutFn` automatically handles cleanup on unmount
- `setTimeout` requires explicit cleanup in useEffect return
- `useTimeoutFn` gives you `isPending` state for loading indicators
- `setTimeout` requires additional state management for pending status

## Summary

Wrapping native timers in React hooks like `useTimeout` and `useTimeoutFn` gives you safer, cleaner, and more declarative control over delayed behavior. You avoid stale closures, manual cleanup, and dependency-array headaches—and with `useTimeoutFn`, you also gain full imperative control (including an `isPending` flag for UI feedback).

Next time, I'll dive into the actual source code of `useTimeout` and `useTimeoutFn`, showing how these hooks work under the hood. Stay tuned!

## 7. References

- [react-use: useTimeout](https://github.com/streamich/react-use/blob/master/docs/useTimeout.md)
- [react-use: useTimeoutFn](https://github.com/streamich/react-use/blob/master/docs/useTimeoutFn.md)
- [MDN Web Docs: setTimeout](https://developer.mozilla.org/docs/Web/API/setTimeout)
- [React Hooks: useEffect](https://react.dev/reference/react/useEffect)
