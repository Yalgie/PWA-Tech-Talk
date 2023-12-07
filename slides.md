---
background: https://images.unsplash.com/photo-1620641788421-7a1c342ea42e?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2748&q=80
class: text-center
highlighter: shiki
lineNumbers: false
theme: dracula
drawings:
  persist: false
transition: slide-left
title: PWA Tech Talk
---

# Progressive Web Apps (PWAs)

## Ft. Web & Service Workers

---
layout: image-right
image: https://media.giphy.com/media/ukGm72ZLZvYfS/giphy.gif
---

# What are PWAs?

- PWAs are a combination of a bunch of different technologies. There isn't one thing that makes a PWA a PWA
- They combine the flexibility of web development with the experience of native apps
- They work across all platforms using a single codebase
- No need to build separate native apps for different operating systems

---
layout: image-right
image: https://media.giphy.com/media/kaiDjO7SmwMww/giphy.gif
---

# A Brief History Lesson

- When Steve Jobs / Apple announced the first iPhone in 2007 they actually pitched the idea of developers writing apps for safari
- The concept gained momentum with the app store in 2008 but it never really came to fruition.
- Google engineers coined "Progressive Web Apps" in 2015
- On February 16, 2023 apple announced support for web-push notifications. Android added this back in 2013.

---
layout: image-right
image: https://ik.imagekit.io/TCF/rn-img.png?updatedAt=1701945568958
---

# PWA vs Native Apps

- Expo / React Native solves a similar issue to PWAs. We only need to write the code once and deploy it to ios/android.
- PWAs however skip the app store entirely and we can avoid working with native code
- PWAs don't have access to a lot of native SDKs that native apps have access to
- PWA install process is still a little janky (Safari only on iOS)

---

# Service Workers and Web Workers
### What's the difference?

Before digging into how PWAs work it's important to understand some of the fundamental tech that makes them tick

_Service Workers_
- Service Workers are what make PWAs actually work, you generally use service workers when dealing with network requests, caching (offline) and notifications.
- You can use service workers in isolation, you don't need to make a PWA in order to use them

_Web Workers_
- While not really related to PWAs I think web workers are worth bringing up quickly, it's also easy to get confused by service and web workers.
- Web workers are used to handle memory intensive tasks that can cause the main thread to be blocked. They are generally not used to perform network requests but they are able to do so
  
---
layout: image-right
image: https://ik.imagekit.io/TCF/ww.png?updatedAt=1701946256326 
---

# Web Workers

- JS is a single threaded language, processing large amounts of data or performing memory intensive tasks can block the event loop and slow down your app
- Web workers let us unblock the main thread and offload the heavy lifting to another background thread
- _Service workers also operate on a separate thread_

---

# Web Workers Example Code

```js {all|3|5|14-16|18|7-10|all}  {lines:true}
// main.js (Main Thread):
if (window.Worker) {
    const myWorker = new Worker('worker.js');

    myWorker.postMessage('Hello, Worker!');

    myWorker.onmessage = function(msg) {
        console.log(msg.data);
        // "Hello, Main Thread!"
    };
}

// worker.js (Web Worker):
onmessage = function(msg) {
    console.log(msg.data);
    // Hello, Worker!

    postMessage("Hello, Main Thread!");
};
```

---

# Service Workers

Service workers are what make PWAs work, they have 6 available events for use:
- _install_: Triggered when the Service Worker is initially installed, used for setting up and caching key assets.
- _activate_: Occurs when the Service Worker starts up, handling cleanup of old caches and preparation for handling fetches.
- _message_: Fired when the Service Worker receives a message, typically used for inter-script communication.
- _fetch_: Invoked for every network request, allowing interception and custom handling of these requests, often for caching strategies.
- _sync_: Triggered when the browser has internet connectivity again, used for executing actions deferred during offline periods.
- _push_: Occurs when a push message is received from a server, enabling the delivery of notifications or updates.

---

# Service Workers `install` & `message`

```js {all|2|3|13-15|17|4|19-20|22|7-9|all}  {lines:true}
// main.js (Main Thread):
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('service-worker.js').then(registration => {
        navigator.serviceWorker.controller.postMessage('Hello, Worker!');
    });

    navigator.serviceWorker.addEventListener('message', event => {
        console.log(event.data); // "Hello, Main Thread!"
    });
}

// service-worker.js (Service Worker):
self.addEventListener('install', function(event) {
    // Cache initial resources
});

self.addEventListener("activate", (event) => {}); // Fired after install

self.addEventListener('message', event => {
    console.log(event.data); // 'Hello, Worker!'

    event.source.postMessage("Hello, Main Thread!");
});
```

---

# Service Workers `fetch`

```js {all|2|8-11|13-16|all}  {lines:true}
// service-worker.js (Service Worker):
const CACHE_NAME = "v1";

... 

self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.match(event.request).then((cachedResponse) => {
        // Return Cached Response
        if (cachedResponse) return cachedResponse;

        return fetch(event.request).then((networkResponse) => {
          // Cache Response
          cache.put(event.request, networkResponse.clone());
          return networkResponse;
        });
      });
    })
  );
});
```

---
layout: image
image: https://ik.imagekit.io/TCF/poggers2.png?updatedAt=1701950914280
---

---

# Service Workers `activate`

```js {all|2|7,13-15|all}  {lines:true}
// service-worker.js (Service Worker):
const CACHE_NAME = "v2"; // v1 -> v2

...

self.addEventListener("activate", (event) => {
  const cacheWhitelist = [CACHE_NAME];

  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

---
layout: image-right
image: https://ik.imagekit.io/TCF/frame%20(1).png?updatedAt=1701949679994
---

# Service Workers `push` & Notifications

- While you can roll your own push notification service it can be a bit of a headache to handle all the permissions and keys. You need to check if the device has disabled permissions and update your database etc.
- It's easier to use an existing service like firebase FCM, OneSignal, MagicBell etc.
- Scan the QR code for a nice example of a PWA w/ notifications

---
layout: two-cols
---
# Manifest

- Another piece that adds to the PWA is the manifest file
- The app manifest file allows you to configure a bunch of stuff like the app icons
- One of the most important options to allow the native app experience is `display: standalone` 

::right::

```json {all|6|all}  {lines:true}
// manifest.json
{
  "name": "Your App Name",
  "short_name": "App",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "path/to/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "path/to/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

---
layout: image-right
image: https://ik.imagekit.io/TCF/lighthouse.png?updatedAt=1701953913664
---
# Lighthouse & Desktop Support

- You can download PWAs on Mac and Windows also!
- Not all browsers support it however
- Googles developer tool Lighthouse let's you debug your PWA

---

# Are PWAs the future?

- App development is expensive and time consuming
- Easily distributable and updatable 
- Install process is still a bit of a hurdle
- Still not 100% support across desktop and mobile
- I don't see native apps going anywhere soon but I think PWAs will become more and more popular and normal to use

---
layout: center
class: text-center
---

# Questions?
