We just built this user interface such that when we click refresh it seems to do the following. It does one network request, and then with the JSON response it selects three random users.

But in reality, that's not what happens. Actually, there are three network requests, one for each user. Now, that might sound strange, and if you don't believe me, we can check here.

We can write a `console.log` where we are creating a promise for the network request. We can just put, `'do network request'` as a message. If we see the console and we rerun this, then we're going to see three network requests.

####Console Output
```
"do network request"

"do network request"

"do network request"
```

Now, this might sound strange, but there's a logical explanation to this. If you notice `responseStream` is used three times, once for each of these suggestion streams. 

```javascript
var suggestion1Stream = createSuggestionStream(responseStream);
var suggestion2Stream = createSuggestionStream(responseStream);
var suggestion3Stream = createSuggestionStream(responseStream);
```

These are being subscribed. So in a way, there are three subscribers to the `responseStream`. It's an indirect subscription, but things work as a chain.

If you `.subscribe` to `suggestion1Stream`, it will `.subscribe` to the `responseStream`, which will `.subscribe` to the `requestStream` and so forth. That's why, in the chain, we are are subscribing to `responseStream` three times.

Now, observables works like videos, actually. If you have on your phone one YouTube video, then you are doing one network request for that video. And if your friend has his phone and he's watching the same video, he will do also the same network request for that video.

It works in the same way. You can think of a stream as a video and subscribing as watching. So, all the related side effects will happen separately.

The question is, if we want to really have just one network request, how do we do that? Well, just like in real life, you can have one screen with one video and two people watching that. You can also do this with observables. You can say, this will be a shared observable.

```javascript
var responseStream = requestOnRefreshStream.merge(startupRequestStream)
	.flatMap(requestUrl => {
		console.log('do network request');
		return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl))
	})
	.shareReplay(1);
```

Now, this means that there is internally one `.subscribe`. So, this `shareReplay` will make its automatic `.subscribe`. Whoever subscribes later to this `responseStream`, all kinds of multiple subscribers will share the same inner subscription that this `shareReplay` has.

Why replay `1`? Well, that's because if there happens to be a really late subscriber...let's say someone does a `setTimeout` and doesn't `.subscribe` to the `responseStream` after a long while. Let's say, three seconds or even 10 seconds. Then, this `.subscribe` will get a replayed response JSON. It will not do a new network request, but it will simply replay that same JSON that happened before.

That was the `shareReplay`. We can be confident that this `console.log` will be shown only once.

####Console Output
```
"do network request"
```