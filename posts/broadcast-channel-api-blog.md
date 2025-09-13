# Using BroadcastChannel API for Cross-Tab Session Synchronization

```
           /\_/\     ←→     /\_/\     ←→     /\_/\
          ( •.• )  Tab 1   ( •.• )  Tab 2   ( •.• )  Tab 3
         / >🌐< \          / >🌐< \          / >🌐< \

              ↕️ BroadcastChannel 📡 ↕️
              (Real-time messaging)
```

## Table of Contents

1. [Background](#1-background)
2. [What Is BroadcastChannel?](#2-what-is-broadcastchannel)
3. [Key Differences](#3-key-differences)
   - [3.1 vs. postMessage](#31-vs-postmessage)
   - [3.2 vs. Web Storage](#32-vs-web-storage-localstorage--sessionstorage)
4. [How to Use It](#4-how-to-use-it)
5. [Pros & Cons](#5-pros--cons)
6. [Other Situations Suitable for BroadcastChannel](#6-other-situations-suitable-for-broadcastchannel)
7. [Summary](#summary)
8. [References](#7-references)

---

## 1. Background

In our application, we need to assign and persist a single `session_id` for users who have disabled cookies and for whom we cannot use any form of device storage (localStorage/sessionStorage/IndexedDB). At the same time, we want all open tabs to report the same `session_id` in their analytics events. To solve this, we leverage the BroadcastChannel API as an in-memory, cross-tab communication bus that carries the `session_id` whenever a new tab opens or when we generate a fresh ID.

## 2. What Is BroadcastChannel?

The BroadcastChannel API provides a simple publish/subscribe ("pub/sub") messaging channel that any browsing context—windows, tabs, iframes—or workers sharing the same origin and storage partition can join by name. Once joined, any context can broadcast a structured-clone message; all other listeners on that channel instantly receive it, without explicit references or server involvement.

## 3. Key Differences

Understanding how BroadcastChannel compares to other communication methods helps you choose the right tool for your specific use case. Let's examine the key differences:

### 3.1 vs. postMessage

| Aspect | postMessage | BroadcastChannel |
|--------|-------------|------------------|
| **Targeting** | One-to-one: reference (e.g. iframe.contentWindow or Worker) + targetOrigin. | One-to-many: any context opens the same channel name. Messages go to all peers. |
| **Cross-origin** | ✅ Supports cross-origin if you set targetOrigin. | ❌ Only same-origin + same storage partition. |
| **Use case** | Frame↔frame or main↔worker communication. | Global state broadcasts (e.g. theme, logout) across tabs. |

**When to use postMessage**: When you need direct communication between specific contexts (like a parent window and its iframe, or the main thread and a web worker). You must have a direct reference to the target context.

**When to use BroadcastChannel**: When you want to broadcast messages to all open tabs/windows without needing to maintain references to each one. Perfect for global state management across multiple contexts.

### 3.2 vs. Web Storage (localStorage / sessionStorage)

| Aspect | Web Storage | BroadcastChannel |
|--------|-------------|------------------|
| **Persistence** | ✅ Survives reloads, restarts, and closed/reopened tabs. | ❌ Ephemeral—in-memory only, lost on reload or close. |
| **Cross-tab sync** | ✅ via storage event (fires in other tabs only). | ✅ real-time, fires in all tabs (sender does not see its own event). |
| **Data size/types** | String key/value pairs only. | Any structured-cloneable data (objects, arrays, Blobs). |
| **Latency** | Slower (I/O + event firing). | Fast, direct in-memory message passing. |

**When to use Web Storage**: When you need data to persist across browser sessions, page reloads, or when you want to store data that survives tab closures. The storage event provides a way to sync changes across tabs, but it's slower and only fires in other tabs (not the one that made the change).

**When to use BroadcastChannel**: When you need real-time, high-frequency communication between tabs and don't require persistence. The sender doesn't receive its own message, making it perfect for broadcasting updates to other contexts while avoiding feedback loops.

## 4. How to Use It

```javascript
// 1. Create or join the channel (in each tab):
const sessionChannel = new BroadcastChannel('session-id');

// 2. On initial load, try to receive an existing session_id:
let currentSessionId = null;
sessionChannel.onmessage = ({ data }) => {
  if (data.type === 'SESSION_ID' && !currentSessionId) {
    currentSessionId = data.value;
    initializeAnalytics(currentSessionId);
  }
};

// 3. If no session_id arrived within X ms, generate one and broadcast it:
setTimeout(() => {
  if (!currentSessionId) {
    currentSessionId = generateUUID();
    sessionChannel.postMessage({ type: 'SESSION_ID', value: currentSessionId });
    initializeAnalytics(currentSessionId);
  }
}, 50);

// 4. When done, clean up:
// sessionChannel.close();
```

**API Methods:**
- `BroadcastChannel(name)`: Joins or creates a channel by name.
- `.postMessage(payload)`: Sends payload to all other listeners.
- `.onmessage = fn`: Handles incoming messages.
- `.close()`: Leaves the channel and frees resources.

## 5. Pros & Cons

### Pros
- **Zero external dependencies**: no server, no cookies, no storage permissions needed.
- **Real-time cross-tab sync**: instant delivery across all open contexts.
- **Structured data support**: send complex objects without manual serialization.
- **Lightweight & in-memory**: minimal overhead, great for high-frequency updates.

### Cons
- **No persistence**: data is lost on page reload or tab close; you must re-broadcast as needed.
- **Same-origin restriction**: cannot communicate across different origins or storage partitions.
- **Not a security boundary**: messages can be read by any same-origin script—avoid sending secrets.

## 6. Other Situations Suitable for BroadcastChannel

Use BroadcastChannel beyond session synchronization for any scenario requiring in-memory, low-latency pub/sub across same-origin contexts:

- **Real-time Collaboration**: Sync cursor positions, document edits, or drawing strokes in collaborative editors open in multiple tabs.
- **Cross-Tab Form State**: Preserve unsaved form data when users switch between tabs or refresh the page, avoiding data loss.
- **Theming & Preferences**: Broadcast theme changes (dark/light mode), language switches, or layout toggles so all tabs update UI instantly.
- **Feature Flags**: Toggle experimental features (feature flags) across open tabs without reloads, useful for A/B testing.
- **Centralized Heartbeats**: Implement a single "heartbeat" or keep-alive signal so that only one tab pings a backend periodically while informing others to standby.
- **Coordinated Actions**: Coordinate actions like batch uploads, downloads, or background tasks to prevent duplicate work across tabs.

## Summary

For cases where you need ephemeral, high-speed broadcast messaging within same-origin browsing contexts—such as session propagation, UI sync, or coordinating background tasks—the BroadcastChannel API offers a clean and efficient solution.

## 7. References

- [MDN Web Docs: BroadcastChannel API](https://developer.mozilla.org/docs/Web/API/BroadcastChannel)
- [WHATWG Living Standard: BroadcastChannel](https://html.spec.whatwg.org/multipage/web-messaging.html#broadcasting-to-other-browsing-contexts)
- [Can I Use: BroadcastChannel](https://caniuse.com/broadcastchannel) 