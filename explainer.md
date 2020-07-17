# Explainer for FetchEvent.handled promise

## Introduction
The FetchEvent.handled API gives Service Worker a way to do some work after the fetch event is consumed. Detailed discussion of the background
of adding this API can be found in https://github.com/w3c/ServiceWorker/issues/1397.

## When and how do we use this feature
Developers may need to do some bookkeeping work after a response is responded to the browser. They can do it this way:
```
self.addEventListener('fetch', function(event) {
   event.respondWith(async function() {
     let response = await loadResponse(event.request);
     doBookkeeping(event.request.url);  // do bookkeeping for request.
     return response;
   } ());
}
```
The above doBookkeeping() method delayed the response to be returned for processing. While with FetchEvent.handled added. The code can be change to:
```
self.addEventListener('fetch', function(event) {
   event.handled.then(() => {
     doBookkeeping(event.request.url);  // do bookkeeping for the request.
   });
   event.respondWith(async function() {
     let response = await loadResponse(event.request));
     return response;
   }());
}
```
The  bookkeeping work runs after the promise passed to repsondWith() is handled, and that means the bookkeeping task is not delaying the response being returned.

Below is a quoted comment from https://github.com/w3c/ServiceWorker/issues/1397#issue-426624529
> The work around for this issue is to try to use a microtask or task to get your completion work to execute after the browser is done. This is pretty much guesswork,
though, since I don't think we clearly define how browsers process the response on the SW thread. For example, in chrome we end up internally queuing a microtask after
the respondWith promise completes. So if you want to run your completion work after you need two microtasks to get behind the browser. And of course other browsers might
be different. Providing an explicit API would avoid this sort of confusion.

## List of when to resolve and reject the promise

Returns a promise that resolves:

 - When respondWith() is not called.
 - When the promise provided to respondWith() is resolved.

Returns a promise that rejects:

 - When respondWith() is not called but the fetch event is cancelled.
 - When the promise provided to respondWith() is an invalid response.
 - When the promise provided to respondWith() is rejected.
