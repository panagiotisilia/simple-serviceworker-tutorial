# What is this?

It's a really simple ServiceWorker example. No build systems, (almost) no dependencies.

# Exercises

## 1. Get it running locally

Either clone it via git, or just grab the zip file from TODO.

If you already run a webserver locally, put the files there. Or you can run a web server from the terminal for the current directory by installing [node.js](http://nodejs.org/) and running:

```sh
npm install http-server -g
http-server -c-1
```

Visit the site in Chrome, open the dev tools and look at the console. Once you refresh the page, it'll be under the ServiceWorker's control.

## 2. Go offline

The simplest way to go offline is using Chrome's device emulator.

Alternatively, disable your internet connection & shut down your local web server.

If you refresh the page, it still works, even through you're offline! Well, one of the pictures has failed, but we can fix that soon.

Take a look at the code in `index.html` and `sw.js`, work out how it's put together.

## 3. Fixing that image

The `install` event in the ServiceWorker is setting up the cache, but it's missing a reference to that second image. Add it to the array. The URL is `ice-cream.jpg`. It doesn't need to be a no-cors request like the Flickr image, because it's on the same origin.

Make sure you're online, refresh the page & watch the console. The browser checks for updates to the ServiceWorker script, if anything in the file has changed it considers it to be a new version. The new version is been picked up, but it isn't ready to use.

If you open a new tab and go to `chrome://serviceworker-internals` you'll see both the old & new worker listed.

TODO picture

**Not seeing the new worker?** It could be that your server send the original JS with a far-future `max-age` or similar caching header. Instead, use the node server mentioned in exercise 1 instead.

Follow the instructions in the page's console to get the new version working.

Test your page offline to ensure the image is properly cached & served.

## 4. Faster updates!

The update process you just encountered means only one version of your app can run at once. That's often useful, but we don't need it right now.

In your `install` event, before the call to `event.waitUntil` add:

```js
if (self.skipWaiting) { self.skipWaiting(); }
```

Chrome 40 shipped with ServiceWorker but without `skipWaiting`, so the `if` statement prevents errors there. If you want to see the effects of `skipWaiting`, use a newer version of Chrome, such as [Chrome Canary](https://www.google.com/chrome/browser/canary.html).

`skipWaiting` means your new ServiceWorker won't wait for tabs to stop using the old version before it takes over. If you refresh the page now, the new version should activate immediately. 

`skipWaiting` means your new worker will handle requests from pages that were loaded with the old worker. If that's a problem, or if multiple tabs running different versions of your app/site is a problem, avoid `skipWaiting`.

## 5. Messing around with images

Currently we're responding to all requests by trying the cache & falling back to the network. Let's do something different for particular URLs.

In the `fetch` event, before calling `event.respondWith`, add the following code:

```js
if (/\.jpg$/.test(event.request.url)) {
  event.respondWith(fetch('trollface.svg'));
  return;
}
```

Here we're intercepting URLs that end `.jpg` and responding with a network fetch for a different resource.

Refresh the page, watch the console, and once the new ServiceWorker is active, refresh again. Different images!

## 6. URLs and manual responses

In the previous step, we handled all requests ending `.jpg`, but often you want finer control over which URLs you handle.

In the `fetch` event, add the following code before the code you added in the previous exercise:

```js
var pageURL = new URL('./', location);

if (event.request.url === pageURL.href) {
  event.respondWith(new Response("Hello world!"))
  return;
}
```

Refresh the page, watch the console, and once the new ServiceWorker is active, refresh again. Different response!

This is how you create responses manually!

# Further reading

You're now cooking with ServiceWorkers! To learn more about how they work, and practical patterns you'll use in apps and sites, check out the resources listed on [is-serviceworker-ready](https://jakearchibald.github.io/isserviceworkerready/resources.html).