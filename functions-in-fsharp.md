In learning about F#, one of the first things I had to come to terms with is that all F# functions have only one input and only one output.

This is a strange thing to understand when you're allowed to write things like:
```
> let add x y = x + y;;
```
Even though it's clear we're returning one integer, it sure looks like we're passing two arguments to `add`.

In reality, what's happening becomes a bit clearer if we examine the function's type signature:
```
val add: x: int -> y: int -> int
```
What this is saying is that `add` is a function that takes an `int x` and returns a function that takes `int y` and returns an `int`.  Think of each arrow (`->`) as a function that takes the parameter from the left and returns the value to the right.  This is called [Currying](https://en.wikipedia.org/wiki/Currying), which is apparently named after the guy who invented Haskell.

Therefore, we can pass an integer, say `5`, to the function `add`, and it will return another function which will take an interger `y`, add it to `5`, and then return the result.  If we name that new function `add5`, we can define it as follows:
```
> let add5 = add 5;;
```
which gives us the type signature of
```
val add5: (int -> int)
```
Calling add5 with another integer gives us the sum:
```
> add5 4;;
val it: int = 9

> add5 -7;;
val it: int = -2
```
We can also partially define other such functions by just passing the first argument to `add`:
```
> let add7 = add 7;;
val add7: (int -> int)

> let add10 = add 10;;
val add10: (int -> int)

> add7 4;;
val it: int = 11

> add10 20;;
val it: int = 30
```
Unsurprisingly, this concept is called [Partial Application](https://en.wikipedia.org/wiki/Partial_application).

**Update on 08 March 2026**:

I didn't want my post to imply that you can't pass two or more arguments to a function.  You certainly can:
```
> let add a b = a + b;;
val add: a: int -> b: int -> int

> add 4 9;;
val it: int = 13
```
In my very limited experience with F# so far, I don't worry too much about currying; when I see a type signature that looks like `val add: a: int -> b: int -> int`, I'm still thinking about it as a function that takes two integers and returns one integer.