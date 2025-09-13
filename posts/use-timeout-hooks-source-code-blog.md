# Under the Hood: How useTimeout and useTimeoutFn Actually Work

```
    /\_/\           
   ( o.o )  - Reading Source Code 
  / >📖< \                    
```

## Table of Contents

1. [Introduction](#1-introduction)
2. [useTimeout Source Code & Explanation](#2-usetimeout-source-code--explanation)
3. [useTimeoutFn Source Code & Explanation](#3-usetimeoutfn-source-code--explanation)
4. [Key Implementation Patterns & Common Pitfalls](#4-key-implementation-patterns--common-pitfalls)
5. [Performance Considerations](#5-performance-considerations)
6. [Summary](#summary)
7. [References](#7-references)

---

## 1. Introduction

In my [previous post](./use-timeout-hooks-blog.md), I showed how `useTimeout` and `useTimeoutFn` from the `react-use` library can simplify timer management in React applications. Now let's dive deeper and examine the actual source code to understand how these hooks work under the hood.

Understanding the implementation helps us:
- **Appreciate the design decisions** behind these hooks
- **Learn React patterns** for managing side effects
- **Avoid common pitfalls** when building similar hooks
- **Optimize performance** in our own timer implementations

## 2. useTimeout Source Code & Explanation

### Complete Source Code

```typescript
import { useEffect, useRef } from 'react';

export function useTimeout(fn: () => void, delay: number | null): void {
  const fnRef = useRef(fn);
  fnRef.current = fn;

  useEffect(() => {
    if (delay === null) return;
    
    const timer = setTimeout(() => {
      fnRef.current();
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [delay]);
}
```

### Detailed Explanation

Let's break down this implementation line by line:

#### 1. Function Signature
```typescript
export function useTimeout(fn: () => void, delay: number | null): void
```
- **`fn`**: A callback function that will be executed after the delay
- **`delay`**: Time in milliseconds to wait, or `null` to disable the timer
- **Return**: `void` - this hook doesn't return anything

#### 2. Callback Reference Management
```typescript
const fnRef = useRef(fn);
fnRef.current = fn;
```
This is a crucial pattern that solves the **stale closure problem**:
- `useRef` creates a mutable reference that persists across re-renders
- `fnRef.current = fn` updates the reference with the latest callback on every render
- When the timer fires, it calls `fnRef.current()` which always has the most recent function

**Why this matters:** Without this pattern, the timer might call an old version of the function that references stale state.

#### 3. Effect with Conditional Logic
```typescript
useEffect(() => {
  if (delay === null) return;
  
  const timer = setTimeout(() => {
    fnRef.current();
  }, delay);

  return () => {
    clearTimeout(timer);
  };
}, [delay]);
```

**Step-by-step breakdown:**
1. **Conditional check**: If `delay` is `null`, return early (no timer created)
2. **Timer creation**: `setTimeout` schedules the callback to run after `delay` milliseconds
3. **Cleanup function**: Returns a function that clears the timer when the component unmounts or when `delay` changes
4. **Dependency array**: Only re-runs the effect when `delay` changes

#### 4. How It Works in Practice

```typescript
// Example usage
function MyComponent() {
  const [count, setCount] = useState(0);
  
  // This will run after 5 seconds
  useTimeout(() => {
    console.log(`Count is: ${count}`);
  }, 5000);
  
  // This timer is disabled
  useTimeout(() => {
    console.log('This never runs');
  }, null);
  
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**Key behaviors:**
- Timer starts immediately when the component mounts (if `delay` is not `null`)
- Timer is automatically cleared when the component unmounts
- Timer is cleared and recreated when `delay` changes
- The callback always has access to the latest state and props

## 3. useTimeoutFn Source Code & Explanation

### Complete Source Code

```typescript
import { useCallback, useEffect, useRef, useState } from 'react';

export interface UseTimeoutFnReturn {
  isPending: boolean;
  start: () => void;
  cancel: () => void;
}

export function useTimeoutFn(
  fn: () => void,
  delay: number,
  options: { immediate?: boolean } = {}
): UseTimeoutFnReturn {
  const { immediate = true } = options;
  
  const fnRef = useRef(fn);
  fnRef.current = fn;
  
  const [isPending, setIsPending] = useState(false);
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);

  const cancel = useCallback(() => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = null;
    }
    setIsPending(false);
  }, []);

  const start = useCallback(() => {
    cancel();
    setIsPending(true);
    timeoutRef.current = setTimeout(() => {
      fnRef.current();
      setIsPending(false);
    }, delay);
  }, [delay, cancel]);

  useEffect(() => {
    if (immediate) {
      start();
    }

    return cancel;
  }, [immediate, start, cancel]);

  return { isPending, start, cancel };
}
```

### Detailed Explanation

This hook is more complex because it provides manual control over the timer. Let's examine each part:

#### 1. Function Signature and Return Type
```typescript
export function useTimeoutFn(
  fn: () => void,
  delay: number,
  options: { immediate?: boolean } = {}
): UseTimeoutFnReturn
```

**Parameters:**
- **`fn`**: Callback function to execute
- **`delay`**: Time in milliseconds (cannot be `null` like `useTimeout`)
- **`options`**: Optional configuration object with `immediate` flag

**Return value:**
```typescript
interface UseTimeoutFnReturn {
  isPending: boolean;  // Whether timer is currently running
  start: () => void;   // Function to start/restart timer
  cancel: () => void;  // Function to cancel timer
}
```

#### 2. State and Reference Management
```typescript
const fnRef = useRef(fn);
fnRef.current = fn;

const [isPending, setIsPending] = useState(false);
const timeoutRef = useRef<NodeJS.Timeout | null>(null);
```

**Multiple refs and state:**
- **`fnRef`**: Same pattern as `useTimeout` for stable callback reference
- **`isPending`**: State that tracks whether a timer is currently running
- **`timeoutRef`**: Stores the actual timer ID for manual cancellation

#### 3. Cancel Function
```typescript
const cancel = useCallback(() => {
  if (timeoutRef.current) {
    clearTimeout(timeoutRef.current);
    timeoutRef.current = null;
  }
  setIsPending(false);
}, []);
```

**What it does:**
1. Checks if there's an active timer (`timeoutRef.current`)
2. Clears the timer using `clearTimeout`
3. Sets the ref to `null` to indicate no active timer
4. Updates `isPending` state to `false`
5. Uses `useCallback` to prevent unnecessary re-renders

#### 4. Start Function
```typescript
const start = useCallback(() => {
  cancel();
  setIsPending(true);
  timeoutRef.current = setTimeout(() => {
    fnRef.current();
    setIsPending(false);
  }, delay);
}, [delay, cancel]);
```

**Step-by-step process:**
1. **Cancel existing timer**: Calls `cancel()` to clear any running timer
2. **Set pending state**: `setIsPending(true)` indicates timer is running
3. **Create new timer**: `setTimeout` schedules the callback
4. **Execute callback**: When timer fires, calls `fnRef.current()` and sets `isPending` to `false`
5. **Dependencies**: Recreates function when `delay` or `cancel` changes

#### 5. Effect for Initialization
```typescript
useEffect(() => {
  if (immediate) {
    start();
  }

  return cancel;
}, [immediate, start, cancel]);
```

**Initialization logic:**
- If `immediate` is `true` (default), starts the timer on mount
- If `immediate` is `false`, waits for manual `start()` call
- Returns `cancel` as cleanup function for component unmount

#### 6. How It Works in Practice

```typescript
// Example usage
function MyComponent() {
  const [count, setCount] = useState(0);
  
  // ✅ Good: Stable callback references
  const handleIncrement = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  const timerCallback = useCallback(() => {
    console.log(`Count: ${count}`);
  }, [count]); // Include count as dependency since it's used in the callback
  
  const { isPending, start, cancel } = useTimeoutFn(
    timerCallback,
    3000,
    { immediate: false } // Don't start automatically
  );
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Timer status: {isPending ? 'Running' : 'Stopped'}</p>
      <button onClick={handleIncrement}>Increment</button>
      <button onClick={start}>Start Timer</button>
      <button onClick={cancel}>Cancel Timer</button>
    </div>
  );
}
```

**Key differences from `useTimeout`:**
- **Manual control**: You decide when to start/cancel
- **State feedback**: `isPending` tells you if timer is running
- **Imperative API**: `start()` and `cancel()` functions
- **No automatic restart**: Timer doesn't restart when dependencies change

## 4. Key Implementation Patterns & Common Pitfalls

### Pattern 1: Callback Reference Stability (Preventing Stale Closures)

Both hooks use this pattern:
```typescript
const fnRef = useRef(fn);
fnRef.current = fn;
```

**What are stale closures?**
A stale closure occurs when a function captures variables from its outer scope, but those variables have changed by the time the function executes. In React, this commonly happens with timers and event handlers.

**Example of a stale closure:**
```typescript
// ❌ Bad: Stale closure problem
function MyComponent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(count); // This will always log 0!
    }, 1000);
    
    return () => clearTimeout(timer);
  }, []); // Empty dependency array
  
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**What happens:**
1. Component mounts with `count = 0`
2. `setTimeout` captures the current value of `count` (which is 0)
3. User clicks button, `count` becomes 1, 2, 3...
4. After 1 second, timer fires and logs `count` - but it still logs 0!
5. The function "remembers" the old value of `count`

**The solution - useRef pattern:**
```typescript
// ✅ Good: Always gets the latest value
function MyComponent() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  countRef.current = count; // Update ref on every render
  
  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(countRef.current); // Always logs the current count!
    }, 1000);
    
    return () => clearTimeout(timer);
  }, []);
  
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**Why this works:**
- `useRef` creates a mutable reference that persists across re-renders
- `countRef.current = count` updates the reference with the latest value on every render
- When the timer fires, it reads `countRef.current` which always has the current value
- The ref is mutable, so it doesn't cause the effect to re-run

**Alternative approaches:**
```typescript
// ❌ Bad: Stale closure risk
useEffect(() => {
  setTimeout(fn, delay);
}, [delay]);

// ✅ Good: Stable reference
const fnRef = useRef(fn);
fnRef.current = fn;
useEffect(() => {
  setTimeout(() => fnRef.current(), delay);
}, [delay]);
```

### Pattern 2: Cleanup in useEffect

```typescript
useEffect(() => {
  const timer = setTimeout(callback, delay);
  return () => clearTimeout(timer);
}, [delay]);
```

**Why?** Prevents memory leaks and ensures only one timer runs at a time.

**Common pitfall - forgetting cleanup:**
```typescript
// ❌ Bad: Memory leak
useEffect(() => {
  setTimeout(callback, delay);
  // No cleanup - timer continues after unmount
}, [delay]);

// ✅ Good: Always cleanup
useEffect(() => {
  const timer = setTimeout(callback, delay);
  return () => clearTimeout(timer);
}, [delay]);
```

### Pattern 3: useCallback for Stable References

```typescript
const cancel = useCallback(() => {
  // implementation
}, []);
```

**Why?** Prevents unnecessary re-renders when the function is passed as a prop or dependency.

**Common pitfall - unstable function references:**
```typescript
// ❌ Bad: New function on every render
const handleTimeout = () => {
  console.log('Hello');
};

// ✅ Good: Stable reference
const handleTimeout = useCallback(() => {
  console.log('Hello');
}, []);
```

## 5. Performance Considerations

### 1. Function Reference Optimization

**The core issue:** Functions created inside components are recreated on every render, causing unnecessary re-renders and effect re-runs.

**Problem - unstable references:**
```typescript
// ❌ Bad: Function recreated on every render
function MyComponent() {
  const [start, cancel] = useTimeoutFn(
    () => console.log('Hello'), // New function every render
    1000
  );
  
  // This causes the effect to re-run unnecessarily
  useEffect(() => {
    start();
  }, [start]); // start changes on every render
}
```

**Solution - stable references:**
```typescript
// ✅ Good: Stable function reference
function MyComponent() {
  const callback = useCallback(() => console.log('Hello'), []);
  const [start, cancel] = useTimeoutFn(callback, 1000);
  
  useEffect(() => {
    start();
  }, [start]); // start is now stable
}
```

### 2. Effect Dependency Optimization

**Minimize effect re-runs by being selective about dependencies:**

```typescript
// ❌ Bad: Effect runs on every render
useEffect(() => {
  // timer logic
}, [someValue, anotherValue, thirdValue]); // Too many dependencies

// ✅ Good: Only essential dependencies
useEffect(() => {
  // timer logic
}, [delay]); // Only when delay changes
```

**When to include dependencies:**
- **Include**: Values that the effect actually uses
- **Exclude**: Values that don't affect the timer behavior
- **Use refs**: For values that shouldn't trigger re-runs but are needed in the callback

### 3. Timer-Specific Optimizations

**Avoid creating timers unnecessarily:**
```typescript
// ❌ Bad: Timer created even when not needed
useTimeout(() => {
  if (shouldRun) {
    doSomething();
  }
}, 1000);

// ✅ Good: Conditional timer creation
useTimeout(() => {
  doSomething();
}, shouldRun ? 1000 : null);
```

**Batch timer operations:**
```typescript
// ❌ Bad: Multiple timers
useTimeout(() => setA(true), 1000);
useTimeout(() => setB(true), 1000);
useTimeout(() => setC(true), 1000);

// ✅ Good: Single timer with batch update
useTimeout(() => {
  setA(true);
  setB(true);
  setC(true);
}, 1000);
```

## 6. Summary

The `useTimeout` and `useTimeoutFn` implementations demonstrate several important React patterns:

1. **Ref-based callback management** prevents stale closures
2. **Effect cleanup** prevents memory leaks
3. **State management** provides UI feedback
4. **useCallback optimization** prevents unnecessary re-renders

Understanding these patterns helps us build better custom hooks and avoid common timer-related bugs. The source code shows how a few lines of well-structured React code can replace complex manual timer management.

## 7. References

- [react-use: useTimeout source](https://github.com/streamich/react-use/blob/master/src/useTimeout.ts)
- [react-use: useTimeoutFn source](https://github.com/streamich/react-use/blob/master/src/useTimeoutFn.ts)
- [React: useRef Hook](https://react.dev/reference/react/useRef)
- [React: useCallback Hook](https://react.dev/reference/react/useCallback)
- [React: useEffect Hook](https://react.dev/reference/react/useEffect)