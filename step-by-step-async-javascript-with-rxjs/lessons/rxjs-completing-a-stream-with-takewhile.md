As it stands right now, our game will run forever. Once I hit start, and start scoring points, it's never going to stop. It's going to keep on counting. You might think you add another `.filter()` or something to make it stop, but `.filter()` does not trigger a complete event on our stream. `.filter()` just tells our stream which things to push through.

What we want is something called `.takeWhile()`, which will look exactly like a `.filter()`, so we'll say `data`. If we want our game to end after three seconds, we can say `data.count` is less than or equal to three. Then, when we start our game, you can see we can do one, two, three.

```javascript
Observable.combineLatest(
	timer$,
	input$,
	(timer, input)=> ({count: timer.count, text:input})

)
	.takeWhile((data)=> data.count <= 3)
	.filter((data)=> data.count === parseInt(data.text))
	.subscribe((x)=> console.log(x));
```

It's never going to score any points for anything beyond that. We also never see when our game actually completes if you want to do something when it's done. What you do there is `.subscribe()` actually takes three functions. This is the function that gets called every onNext or every tick.

This one would get called if there's an error, so log out the error if an error happens. The third function is one that gets called once your stream is complete, so just log out `'complete'`. If I start my stream, I type one, and then wait until three. I'll get a point for three. It completes. Then I can never score again because my stream is done.

```javascript
Observable.combineLatest(
	timer$,
	input$,
	(timer, input)=> ({count: timer.count, text:input})

)
	.takeWhile((data)=> data.count <= 3)
	.filter((data)=> data.count === parseInt(data.text))
	.subscribe(
		(x)=> console.log(x),
		err=> console.log(err),
		()=> console.log('complete')
	);
```
