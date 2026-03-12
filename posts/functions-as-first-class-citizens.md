One of the things about F# that I am still trying to better understand is the concept of treating functions as "first class citizens", especially when it comes to passing functions to other functions or having functions return another function.  Functions that can take another function as an argument or return a function as a result are called [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function).

Coming from more mainstream programming languages, this idea really makes my head explode!

Here's a simple example.  Suppose we define a function `applyFunction` as follows:
```
> let applyFunction funct a b = funct a b;;
val applyFunction: funct: ('a -> 'b -> 'c) -> a: 'a -> b: 'b -> 'c
```
In this example, `applyFunction` takes three arguments:

1. A function `funct` that takes two generic arguments and returns a generic value.

2. A generic argument `'a`.

3. A generic argument `'b`.

It then applies the function `funct` to `'a` and `'b`, returning the generic value `'c`.

Let's see how we can use this in practice.  First, let's define a function that takes two arguments and returns a value:
```
> let add x y = x + y;;
val add: x: int -> y: int -> int
```
The type signature of `add` matches that of `funct` above which means we can now pass `add`, along with two other arguments, to `applyFunction`:
```
> applyFunction add 4 5;;    
val it: int = 9
```
`applyFunction` accepts the function `add` and the two integers `4` and `5` and applies `add` to them returning `9`.

Like we did in the [previous post](/2026/03/functions-in-f.html), we can also use partial application on `applyFunction` to define a new function:
```
> let newAdd = applyFunction add;;
val newAdd: (int -> int -> int)

> newAdd 4 5;;
val it: int = 9
```
`newAdd` now takes two integers and returns another.  In practice, is this how we would define `newAdd`?  Probably not since we could have more easily done:
```
> let newAdd = add;;
val newAdd: (int -> int -> int)
```
However, I think this line of application does make sense when we're working with [anonymous functions](https://en.wikipedia.org/wiki/Anonymous_function), or functions that have no name (also referred to as lamdas).  For example:
```
> let newAdd = applyFunction (fun x y -> x + y);;
val newAdd: (int -> int -> int)

> newAdd 3 4;;
val it: int = 7
```
Here, we're combining the concepts of anonymous functions with partial applcation.  Instead of passing a defined function, such as `add` to `applyFunction`, we're passing anonymous function `fun x y -> x + y`.  This works because the anonymous function has the same type signature expected by `applyFunction`: `'a -> 'b -> 'c`.

For a more practical example of how all of this comes together, let's take a look at Exercise 7.1 in "[F# in Action](https://www.manning.com/books/f-sharp-in-action)" which has us build a simple calculator.

Let's define a new function, `calculate`, similar to `applyFunction`:
```
> let calculate operation x y = operation x y;;
val calculate: operation: ('a -> 'b -> 'c) -> x: 'a -> y: 'b -> 'c
```

Because `calculate` is a higher-order function, we can pass to it various other anonymous functions that represent basic mathematical operations:
```
> calculate (fun a b -> a + b) 2 3;;
val it: int = 5

> calculate (fun a b -> a - b) 10 8;;
val it: int = 2
```
We can get a little bit fancier by passing both an operation as well as a description of the action we are taking as a tuple so that we can return a string that describes what we are doing:
```
> let calculate (operation, action) x y =                   
-     let answer = operation x y                            
-     sprintf $"%i{x} %s{action} %i{y} equals %i{answer}";; 
val calculate:
  operation: (int -> int -> int) * action: string ->
    x: int -> y: int -> string
```
We can then call `calculate` as follows:
```
> let result = calculate (( fun x y -> x / y ), "divided by") 10 2;;
val result: string = "10 divided by 2 equals 5"
```
Putting this all together, we can create a simple `calculate.fsx` script:
```
let calculate (operation, action) x y =
    let answer = operation x y
    sprintf $"%i{x} {action} %i{y} equals %i{answer}"

printfn "Enter the first number: "
let x = System.Console.ReadLine() |> int
printfn "Enter the second number: "
let y = System.Console.ReadLine() |> int

printfn "Enter an operation:"
printfn "    1. Add"
printfn "    2. Multiply"
printfn "    3. Subtract"
printfn "    4. Divide"
printfn "Enter your choice: "
let choice = System.Console.ReadLine()

let operation =
    match choice with
    | "1" -> ((fun a b -> a + b), "plus")
    | "2" -> ((fun a b -> a * b), "multiplied by")
    | "3" -> ((fun a b -> a - b), "minus")
    | "4" -> ((fun a b -> a / b), "divided by")
    | _ -> failwith "Invalid choice"

let result = calculate operation x y
printfn "%s" result
```
which we can run via `dotnet fsi calculate.fsx`:
```
Enter the first number: 
10
Enter the second number: 
5
Enter an operation:
    1. Add
    2. Multiply
    3. Subtract
    4. Divide
Enter your choice: 
2
10 multiplied by 5 equals 50
```
Hopefully this example helps explain a bit more about what higher-order and anonymous functions are in F# and how they can be used in a simple example.
