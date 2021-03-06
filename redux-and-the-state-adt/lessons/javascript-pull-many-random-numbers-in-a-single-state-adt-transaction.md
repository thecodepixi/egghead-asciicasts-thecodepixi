Instructor: [00:00] We begin with this `getDeck` function that returns a state instance with a pair in the resulting, containing a pile of cards generated by the 12 permutations of these 2 arrays in `state`. Using `fst`, we'll peek in on the left side of the pair to find an empty array just itching to be populated with one of the 12 cards in the array that we have on the right.
 
#### index.js
```js
import { drawRandom, getDeck } from './data/model/game'

const state = {
  colors: [ 'orange', 'green', 'blue', 'yellow' ],
  shapes: [ 'square', 'triangle', 'circle' ]
  cards: null,
  seed: 23
}

log(
  getDeck()
    // .chain(drawRandom)
    .evalWith(state).snd().length
)
```

[00:20] We also have this `drawRandom` function that takes a `deck` as its input and returns a state instance that uses `seed` to move a random card from the right pile to the left. When we `chain` it in, we see our right pile has been decreased by one.

```js
log(
  getDeck()
    .chain(drawRandom)
    .evalWith(state).snd().length
)
```

#### output
```txt
11
```

[00:34] To see where it's gone to, we check the left side to find it now contains this `yellow-square`. We would like to do this nine times in total. Over in our game model we create a new state transaction, called `drawNine`, that we define a little differently than our other transactions.

#### data/model/game.js
```js
// drawRandom :: Deck -> State AppState Deck
export const drawRandom = coverage(
  liftA2 (draw),
  compose(randomIndex, snd),
  liftState(identify)
)

// drawNine ::
export const drawNine
```

[00:49] It's defined as an endomorphism that takes `State AppState Deck -> State AppSTate Deck`. We'll reach for this `repeat` helper in `helpers.js` that takes `(Integer, a) -> [ a ]`. `repeat` is an endomorphism that gives us back an array populated with `num`, number of `elem`.

#### helpers.js
```js
// repeat :: (Integer, a) -> [ a ]
export const repeat = (num, elem) => 
  num === 1
    ? [ elem ]
    : repea(num - 1, elem).concat([ elem ])
```

[01:09] Which means we can give `drawNine`, `9` as it's first argument, and then satisfy the second argument with the point-free `chain` on `drawRandom` to get back an `array` of functions that match our signature. 

```js
// drawNine :: State AppState Deck -> State AppState Deck
export const drawNine =
  repeat(9, chain(drawRandom))
```

To get an inkling of what we're working with here, we'll jump back into our `index.js` file and replace our outdated `drawRandom` with this soon-to-be supercharged `drawNine`.

#### index.js
```js
import { drawNine, getDeck } from './data/model/game'

const state = {
  colors: [ 'orange', 'green', 'blue', 'yellow' ],
  shapes: [ 'square', 'triangle', 'circle' ]
  cards: null,
  seed: 23
}

log(
  drawNine
)
```

[01:29] Seeing we get back an `array` of nine functions, but not just any functions. They're all endomorphisms for `State AppState Deck`.

#### output
```js
[
  Function,
  Function,
  Function,
  Function,
  Function,
  Function,
  Function,
  Function,
  Function
]
```

[01:37] Which means we can fold them, using `mreduce` with the endomonoid, which unwrapped the resulting endomorphism, ready for use.

#### data/model/game.js
```js
// drawNine :: State AppState Deck -> State AppState Deck
export const drawNine = mreduce(
  Endo,
  repeat(9, chain(drawRandom))
```

[01:50] `drawNine` is now ours for the calling. We pass it the state instance we get back from calling `getDeck`, and then check out the resultant by calling `evalWith` with our initial `state`. Seeing our expected pair full of cards on the left, and a quick check on the length, shows our expected total of `9`.

#### index.js
```js
import { drawNine, getDeck } from './data/model/game'

const state = {
  colors: [ 'orange', 'green', 'blue', 'yellow' ],
  shapes: [ 'square', 'triangle', 'circle' ]
  cards: null,
  seed: 23
}

log(
  drawNine(getDeck())
    .evalWith(state).fst().length
)
```

[02:07] Nine random cards, while the right still houses the remaining three. Seems legit, indeed, but now we're faced with the challenge of using this function in a way consistent with the rest of our transitions. Why not capture this in a new composition, called `drawFromDeck`? While you may not see it at first, we've already created this composition.

#### data/model/game.js
```js
// drawNine :: State AppState Deck -> State AppState Deck
export const drawNine = mreduce(
  Endo,
  repeat(9, chain(drawRandom))
)

export const drawFromDeck =
```

[02:27] We just need to move this bit, `drawNine(getDeck())`, into our function and use the croks `compose` helper to get it into a more familiar style by arranging it into `compose`, so that it returns the result of `drawNine` being called with the results of `getDeck`. We now have a function that acts like all of our other state transactions.

[02:45] We can define `drawFromDeck` as a function that goes `() -> State AppState Deck`. 

```js
// drawNine :: State AppState Deck -> State AppState Deck
export const drawNine = mreduce(
  Endo,
  repeat(9, chain(drawRandom))
)

// drawFromDeck :: () -> State AppState Deck
export const drawFromDeck =
  compose(drawNine, getDeck)
```

To see our finished transition in action, over in `index.js`, we just meander on up to the top and get rid of all this noise, replacing it with `drawFromDeck`.

[02:59] Due to referential transparency, with the old swapping call, we see the same length of nine in the pairs first, corresponding to our nine drawn cards. Over in the second, we see the remaining cards, with our expected length of three.

#### index.js
```js
import { drawFromDeck } from './data/model/game'

const state = {
  colors: [ 'orange', 'green', 'blue', 'yellow' ],
  shapes: [ 'square', 'triangle', 'circle' ]
  cards: null,
  seed: 23
}

log(
  drawFromDeck()
    .evalWith(state).snd().length
)
```

[03:14] We should probably do something with these cards, so why not just add them to our state on this `cards` attribute? Popping back over to our game model, we create a transaction that we aptly name, `setCards`, taking a `deck` as it's input. We define this new `setCards` state transition as a function that takes `Deck -> State AppState ()`.

```js
// drawFromDeck :: () -> State AppState Deck
export const drawFromDeck =
  compose(drawNine, getDeck)

// setCards :: Deck -> State AppState ()
const setCards = deck =>  
```

[03:37] We implement said `cards` with our `over` helper pointed at the `cards` attribute, and load up the crocks `constant` combinator with the draw pile from the first portion of our `deck`. `setCards` takes a `Deck` as its input, while our `drawFromDeck` function returns a state instance with `Deck` in its resultant.

```js
// drawFromDeck :: () -> State AppState Deck
export const drawFromDeck =
  compose(drawNine, getDeck)

// setCards :: Deck -> State AppState ()
const setCards = deck =>  
  over('cards', constant(deck.fst()))
```

[03:54] Which means we can combine them by chaining. Let's `export` a brand new state transition that we'll name with a plural `pickCards`. We'll define this new `pickCards` transaction as a function that takes the unit from `drawFromDeck` to the `State` `AppState` of unit returned from `setCards`, `() -> State AppState ()`.

```js
// setCards :: Deck -> State AppState ()
const setCards = deck =>  
  over('cards', constant(deck.fst()))

// pickCards :: () -> State AppState ()
export const pickCards =
```

[04:12] Because we have two Kleislis, we'll use `composeK` to define a composition of `setCards` after `drawFromDeck`,

```js
// pickCards :: () -> State AppState ()
export const pickCards =
  composeK(setCards, drawFromDeck)
```

thus allowing us to randomly pick nine cards, and place them in the `cards` portion of our state, with one function call.

[04:25] Which we can now show off in our `index.js` file by pulling it in and replacing `drawFromDeck` in the log. Then we swap out a `evalWith` width with `execWith` to take a look at our newly transitioned state.

[04:37] Focusing in on the cards attribute, we find it to be populated with our expected cards, with a length of nine. We seem to be getting the same random 9 every time, due to the constant value of `23` for our `seed`. By changing it to `Date.now` we see a fresh new deck with every single save. So good.

#### index.js
```js
import { pickCards } from './data/model/game'

const state = {
  colors: [ 'orange', 'green', 'blue', 'yellow' ],
  shapes: [ 'square', 'triangle', 'circle' ]
  cards: null,
  seed: Date.now()
}

log(
  pickCards()
    .execWith(State).cards
)
```
