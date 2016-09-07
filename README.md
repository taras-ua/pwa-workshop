# Offline-first progressive web apps workshop

> Devfest Ukraine 2016

Main goal of this workshop is to create a web application using [progressive web apps](https://developers.google.com/web/progressive-web-apps/) technologies and methods.

## Step 0 - Set up

> Github repository

You can download this workshop from the following github repository:

```
git clone https://github.com/jskvara/pwa-workshop
```

> Web server

While you're free to use your own web server, this workshop is designed to work well with the Chrome Web Server. If you don't have that app installed yet, you can [install it from the Chrome Web Store](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb).

After installing the Web Server for Chrome app, click on the Apps shortcut on the bookmarks bar.

![Apps](https://github.com/jskvara/pwa-workshop/blob/master/docs/apps.png)

In the ensuing window, click on the Web Server icon:

![Web server icon](https://github.com/jskvara/pwa-workshop/blob/master/docs/web-server-icon.png)

Click Choose Folder and select `app`. While you're in the Web Server, also select "Automatically show index.html".

![Web server for chrome](https://github.com/jskvara/pwa-workshop/blob/master/docs/web-server-for-chrome.png)

Open a new browser tab and navigate to `localhost:8887`. You should see the content of the `app/index.html` file.


## Step 1 - Application Shell

> In this step we add an Application Shell to our website.

> The app's shell, is the minimal HTML, CSS, and JavaScript that is required to power the user interface of
> a progressive web app and is one of the the components that ensures reliably good performance.
> Its first load should be extremely quick, then immediately be cached.
> This means that the shell does not need to be loaded every time, but instead just gets the necessary content. 

We add a basic html content for our app:

```html
<body>

<header class="header">
    <h1>Progressive web apps workshop</h1>
</header>

<div class="loader">
    <svg viewBox="0 0 32 32" width="32" height="32">
        <circle class="spinner" cx="16" cy="16" r="14" fill="none"></circle>
    </svg>
</div>
```

And we inline css styles just for the application shell into `<head>` element.

```html
    ...
    <title>Progressive web apps workshop</title>
    <style type="text/css">
        body {
            padding: 0;
            margin: 0;
        }

        .header {
            background: #3f51b5;
            color: #fff;
            font-size: 16px;
            padding: 0 16px;
            position: fixed;
            width: 100%;
            z-index: 1000;
        }

        .loader {
            left: 50%;
            top: 50%;
            position: fixed;
            -webkit-transform: translate(-50%, -50%);
            transform: translate(-50%, -50%);
        }

        @keyframes line {
            0% {
                stroke-dasharray: 2, 85.964;
                -webkit-transform: rotate(0);
                transform: rotate(0);
            }
            50% {
                stroke-dasharray: 65.973, 21.9911;
                stroke-dashoffset: 0;
            }
            100% {
                stroke-dasharray: 2, 85.964;
                stroke-dashoffset: -65.973;
                -webkit-transform: rotate(90deg);
                transform: rotate(90deg);
            }
        }

        @keyframes rotate {
            from {
                -webkit-transform: rotate(0);
                transform: rotate(0);
            }
            to {
                -webkit-transform: rotate(450deg);
                transform: rotate(450deg);
            }
        }

        .loader .spinner {
            box-sizing: border-box;
            stroke: #673ab7;
            stroke-width: 3px;
            -webkit-transform-origin: 50%;
            transform-origin: 50%;
            -webkit-animation: line 1.6s cubic-bezier(0.4, 0, 0.2, 1) infinite, rotate 1.6s linear infinite;
            animation: line 1.6s cubic-bezier(0.4, 0, 0.2, 1) infinite, rotate 1.6s linear infinite;
        }
    </style>
</head>
...
```

It's not always necessary to put css into your head, you can use service workers to cache your css files.

More resources:

- [Architect the App Shell](https://developers.google.com/web/fundamentals/getting-started/your-first-progressive-web-app/step-01) 
- [Implement the App Shell](https://developers.google.com/web/fundamentals/getting-started/your-first-progressive-web-app/step-02) 

## Step 2 - Manifest

> In this step we're going to add manifest file to our website and we show different configuration options.

Create a new JSON called `manifest.json` in the same folder as your `index.html` file with the following content:

```json
{
  "short_name": "PWA workshop",
  "name": "Progressive web apps workshop",
  "icons": [
    {
      "src":"images/icon.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ],
  "start_url": "/",
  "background_color": "#000",
  "theme_color": "#536878",
  "display": "standalone",
  "orientation": "portrait"
}

```

Important options:

- background_color (only when application is loading)
- description
- start_url (URL that loads when a user launches the application from a device. ) - you can use: `"./?utm_source=web_app_manifest"`
- display: standalone | fullscreen (games) | browser
- orientation: portrait | landscape
- icons

```json
{
  "icons": [
      {
        "src": "icon/lowres.webp",
        "sizes": "48x48",
        "type": "image/webp"
      },{
        "src": "icon/lowres",
        "sizes": "48x48"
      },{
        "src": "icon/hd_hi.ico",
        "sizes": "72x72 96x96 128x128 256x256"
      },{
        "src": "icon/hd_hi.svg",
        "sizes": "257x257"
      }]
 }
```

- related applications

```
 "related_applications": [{
    "platform": "web",
    "url": "..."
  }, {
    "platform": "play",
    "url": "..."
  }]
```

-dir

For more information, read this article: [Installable Web Apps with the Web App Manifest in Chrome for Android](https://developers.google.com/web/updates/2014/11/Support-for-installable-web-apps-with-webapp-manifest-in-chrome-38-for-Android)
or [W3C specification](https://w3c.github.io/manifest/)

## Step 3 - Service worker

Adding new pages is really easy. At first we need to update `app/routing.html` file and add new route:

```js
if ('serviceWorker' in navigator) {
    console.log('Service Worker is supported');
    navigator.serviceWorker.register('sw.js').then(function() {
        console.log(navigator.serviceWorker);
        return navigator.serviceWorker.ready;
    }).then(function(reg) {
        console.log('Service Worker is ready', reg);
    }).catch(function(error) {
        console.log('Service Worker error', error);
    });
}
```

```js
console.log('Started', self);

self.addEventListener('install', function(event) {
    self.skipWaiting();
    console.log('Installed', event);
});

self.addEventListener('activate', function(event) {
    console.log('Activated', event);
});

self.addEventListener('push', function(event) {
    console.log('Push message received', event);
});
```

## Step 4 - Cache Application Data

> In this step we're going to create a new Polymer element

We add a list of speakers to our devfest page using our own `devfest-speakers` element.

Create a new directory `devfest` in `app/elements` folder. In this directory we create a new file `devfest-speakers.html` with the following content:

```js
var cacheName = 'pwa-ws';
var filesToCache = [];

self.addEventListener('install', function(e) {
    console.log('[ServiceWorker] Install');
    e.waitUntil(
        caches.open(cacheName).then(function(cache) {
            console.log('[ServiceWorker] Caching app shell');
            return cache.addAll(filesToCache);
        })
    );
});
```

```js
self.addEventListener('fetch', function(event) {
    console.log(event.request.url);
    event.respondWith(
        caches.match(event.request).then(function(response) {
            return response || fetch(event.request);
        })
    );
});
```

## Step 5 - Push notifications

We're going to add list of speakers to our element. At first we add JavaScript collection to our element:

1) Make a project on the Google Developer Console


2) Add `gcm_sender_id` to `manifest.json` file:
 
```
  ...
  "name": "Progressive web apps workshop",
  "gcm_sender_id": "593836075156",
  "icons": [
  ...
```

3) Update `main.js`

```js
    ...
    }).then(function(reg) {
        console.log('Service Worker is ready', reg);
        reg.pushManager.subscribe({userVisibleOnly: true}).then(function(sub) {
            console.log('endpoint:', sub.endpoint);
        });
    }).catch(function(error) {
    ...
```

`https://android.googleapis.com/gcm/send/APA91bGdUldXgd4Eu9MD0qNmGd0K6fu0UvhhNGL9FipYzisrRWbc-qsXpKbxocgSXm7lQuaEOwsJcEWWadNYTyqN8OTMrvNA94shns_BfgFH14wmYw67KZGHsAg74sm1_H7MF2qoyRCwr6AsbTf5n7Cgp7ZqsBZwl8IXGovAuknubr5gaJWBnDc`


4) Show notification

```js
...

self.addEventListener('push', function(event) {
    console.log('Push message', event);

    var title = 'Push message';

    event.waitUntil(
        self.registration.showNotification(title, {
            body: 'The Message',
            icon: 'images/icon.png',
            tag: 'my-tag'
        }));
});
```

5) Open notification

```js
...

self.addEventListener('notificationclick', function(event) {
    console.log('Notification click: tag', event.notification.tag);
    event.notification.close();
    var url = 'https://www.youtube.com/watch?v=DfMnJAzOFng';
    // Check if there's already a tab open with this URL.
    // If yes: focus on the tab.
    // If no: open a tab with the URL.
    event.waitUntil(
        clients.matchAll({
            type: 'window'
        })
            .then(function(windowClients) {
                console.log('WindowClients', windowClients);
                for (var i = 0; i < windowClients.length; i++) {
                    var client = windowClients[i];
                    console.log('WindowClient', client);
                    if (client.url === url && 'focus' in client) {
                        return client.focus();
                    }
                }
                if (clients.openWindow) {
                    return clients.openWindow(url);
                }
            })
    );
});

```

6) Subscribe and unsubscribe

```js
var serviceWorkerRegistration;
var sub;
var isSubscribed = false;
var subscribeButton = document.querySelector('button');

if ('serviceWorker' in navigator) {
    console.log('Service Worker is supported');
    navigator.serviceWorker.register('sw.js').then(function() {
        console.log(navigator.serviceWorker);
        return navigator.serviceWorker.ready;
    }).then(function(reg) {
        console.log('Service Worker is ready', reg);
        subscribeButton.disabled = false;
    }).catch(function(error) {
        console.log('Service Worker error', error);
    });
}

subscribeButton.addEventListener('click', function() {
    if (isSubscribed) {
        unsubscribe();
    } else {
        subscribe();
    }
});

function subscribe() {
    serviceWorkerRegistration.pushManager.subscribe({userVisibleOnly: true}).
    then(function(pushSubscription){
        sub = pushSubscription;
        console.log('Subscribed! Endpoint:', sub.endpoint);
        subscribeButton.textContent = 'Unsubscribe';
        isSubscribed = true;
    });
}

function unsubscribe() {
    sub.unsubscribe().then(function(event) {
        subscribeButton.textContent = 'Subscribe';
        console.log('Unsubscribed!', event);
        isSubscribed = false;
    }).catch(function(error) {
        console.log('Error unsubscribing', error);
        subscribeButton.textContent = 'Subscribe';
    });
}
```

## Step 6 - Deploy to firebase

> We're going to show data from API fetched via AJAX

If you’re new to Firebase, you’ll need to sign in using your Google account and install some tools first.

1) Sign in to Firebase with your Google account at https://firebase.google.com/
2) Install the Firebase tools via npm: `npm install -g firebase-tools`

Once your account has been created and you’ve signed in, you’re ready to deploy!

1) Create a new app at https://console.firebase.google.com/
2) If you haven’t recently signed in to the Firebase tools, update your credentials: `firebase login`
3) Initialize your app, and provide the directory where your completed app lives: `firebase init`
4) Finally, deploy the app to Firebase: `firebase deploy`
5) Celebrate. You’re done! Your app will be deployed to the domain: `https://YOUR-FIREBASE-APP.firebaseapp.com`

Further reading: [Firebase Hosting Guide](https://firebase.google.com/docs/hosting/)