---
title: Discriminated Unions and Pattern Matching in F#
subtitle:
author: Samir Parikh
date: 28 March 2026
updated:
tags: [F#]
---

I'm starting to kick the tires on [discriminated unions](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions) and pattern matching as I continue to learn about F#.  While not exactly the same thing, they are kind of like C#'s [enumeration types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum) where a type can have only possible value.  That value is called a union case.  Using playing cards as an example, we can define the type `Suit`:
```
> type Suit =
-     | Heart
-     | Diamond
-     | Spade
-     | Club;;
type Suit =
  | Heart
  | Diamond
  | Spade
  | Club

> Diamond;;
val it: Suit = Diamond
```
Here, we can see that `Diamond` is of type `Suit`.

Discriminated unions are used to define what is sometimes known as an "or" relationship which, in our example, means that the suit of a playing card can be a Heart *or* a Diamond *or* a Spade *or* a Club.  This is in contrast to F# Records which help define an "and" relationship.

We can then define the type `PlayingCard` as being either an Ace, a face/court card (King, Queen or Jack) or a numeral card (2 through 10, inclusive) using another discriminated union:
```
> type PlayingCard =
-     | Ace of Suit
-     | King of Suit
-     | Queen of Suit
-     | Jack of Suit
-     | ValueCard of int * Suit;;
type PlayingCard =
  | Ace of Suit
  | King of Suit
  | Queen of Suit
  | Jack of Suit
  | ValueCard of int * Suit

> Ace;;
val it: Item: Suit -> PlayingCard

> ValueCard;;
val it: Item1: int * Item2: Suit -> PlayingCard
```
In defining the type `PlayingCard` we have chosen to associate data with each union case.  This is optional.  The data we associate with the union case can be a simple type (e.g. `int`, `string`), a tuple, a record or even another discriminated union.  In our case, we've associated the discriminated union `Suit` to the union cases `Ace`, `King`, `Queen` and `Jack`.  For the union case `ValueCard`, we've associcated a tuple consisting of an `int` and a `Suit`.

Note how F# treats these union cases as a function when we examine their type signatures.  `Ace`, when provided a parameter of type `Suit`, will return a `PlayingCard`.  `ValueCard` is also a function that returns a `PlayingCard` but it takes a tuple made up of an `int` and `Suit` for its parameter.  This is very important because understanding the type signature not only helps us in constructing a `card` but, as we'll see later, also guides us in defining the correct patterns when it comes to matching a `card`. Knowing this, we can define a few individual cards:
```
> let card0 = Ace Spade;;
val card0: PlayingCard = Ace Spade

> let card1 = King Spade;;
val card1: PlayingCard = King Spade

> let card4 = ValueCard (2, Spade);;
val card4: PlayingCard = ValueCard (2, Spade)

> let card18 = ValueCard (3, Club);; 
val card18: PlayingCard = ValueCard (3, Club)
```
(Note that when defining a card that belongs to the `ValueCard` union case, parentheses are required because we have to pass a tuple.  The parentheses are optional if we are defining a card with the union case `Ace`, `King`, `Queen` or `Jack`.  We can include parenetheses for readability of we desire like in the example below.)

Armed with these discriminated unions, we can now create a `deckOfCards`:
```
> let deckOfCards =
-     [
-         for suit in [ Spade; Club; Heart; Diamond ] do
-             yield Ace(suit)
-             yield King(suit)
-             yield Queen(suit)
-             yield Jack(suit)
-             for value in 2 .. 10 do
-                 yield ValueCard(value, suit)
-     ];;
val deckOfCards: PlayingCard list =
  [Ace Spade; King Spade; Queen Spade; Jack Spade; ValueCard (2, Spade);
   ValueCard (3, Spade); ValueCard (4, Spade); ValueCard (5, Spade);
   ValueCard (6, Spade); ValueCard (7, Spade); ValueCard (8, Spade);
   ValueCard (9, Spade); ValueCard (10, Spade); Ace Club; King Club;
   Queen Club; Jack Club; ValueCard (2, Club); ValueCard (3, Club);
   ValueCard (4, Club); ValueCard (5, Club); ValueCard (6, Club);
   ValueCard (7, Club); ValueCard (8, Club); ValueCard (9, Club);
   ValueCard (10, Club); Ace Heart; King Heart; Queen Heart; Jack Heart;
   ValueCard (2, Heart); ValueCard (3, Heart); ValueCard (4, Heart);
   ValueCard (5, Heart); ValueCard (6, Heart); ValueCard (7, Heart);
   ValueCard (8, Heart); ValueCard (9, Heart); ValueCard (10, Heart);
   Ace Diamond; King Diamond; Queen Diamond; Jack Diamond;
   ValueCard (2, Diamond); ValueCard (3, Diamond); ValueCard (4, Diamond);
   ValueCard (5, Diamond); ValueCard (6, Diamond); ValueCard (7, Diamond);
   ValueCard (8, Diamond); ValueCard (9, Diamond); ValueCard (10, Diamond)]
```
Now that we have our `deckOfCards`, we can see how discriminated unions and pattern matching can work together by creating a `describe` function which will take a parameter `card` of type `PlayingCard` and return a `string` that tells us what type of card we have.  Let's take a minute to quickly revisit the type signatures of the union cases that make up the `PlayingCard` type:
```
> Ace;;
val it: Item: Suit -> PlayingCard

> ValueCard;;
val it: Item1: int * Item2: Suit -> PlayingCard
```
Using `Ace` as an example, we can mentally think of how we used its signature to create individual cards above:
```
Ace : Suit -> PlayingCard
```
Using the same logic, we can "deconstruct" the card in a way to be used in pattern matching.  Let's start with just one specific rule and one wildcard (to silence distracting compiler warnings):
```
> let describe card =
-     match card with
-     | Ace suit -> $"Ace of %A{suit}"  // note the "Ace suit" pattern
-     | _ -> "another card";;
val describe: card: PlayingCard -> string

> describe deckOfCards[0];;           
val it: string = "Ace of Spade"

> describe deckOfCards[1];;
val it: string = "another card"
```
Using the same format, we can build up our `describe` function to include the face cards:
```
> let describe card =
-     match card with
-     | Ace suit -> $"Ace of %A{suit}"
-     | King suit -> $"King of %A{suit}"
-     | Queen suit -> $"Queen of %A{suit}"
-     | Jack suit -> $"Jack of %A{suit}"
-     | _ -> "another card";;
val describe: card: PlayingCard -> string

> describe deckOfCards[3];;
val it: string = "Jack of Spade"
```
We can now match for `ValueCard`s but with a slightly different pattern.  Again, going back to the `ValueCard` type signature, we see that it takes a tuple of an `int` and `Suit`.  Another way to think about this is:
```
ValueCard : (int * Suit) -> PlayingCard
```
Therefore, we can augment our `describe` function with a pattern to map to `ValueCard`s:
```
> let describe card =
-     match card with
-     | Ace suit -> $"Ace of %A{suit}"
-     | King suit -> $"King of %A{suit}"
-     | Queen suit -> $"Queen of %A{suit}"
-     | Jack suit -> $"Jack of %A{suit}"
-     | ValueCard (value, suit) -> $"{value} of %A{suit}";;
val describe: card: PlayingCard -> string

> describe deckOfCards[4];;
val it: string = "2 of Spade"

> describe deckOfCards[18];;
val it: string = "3 of Club"
```
Note that because we have now accounted for all possible union cases, we no longer require the wildcard pattern.

Like all of my prior posts on F#, this is in now way meant to be an exhaustive or comprehensive treatise on discriminated unions or pattern matching.  Rather, it's a brief summary of my limited understanding of how they work to hopefully help improve my comprehension.  As usual, comments, suggestions or feedback are greatly appreciated!
