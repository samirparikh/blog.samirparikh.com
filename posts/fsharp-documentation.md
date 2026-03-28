---
title: Learning to Read the F# Documentation
subtitle:
author: Samir Parikh
date: 20 March 2026
updated:
tags: [F#, documentation]
---

I've come across a lot of good resources in the form of books and blog posts in my journey to learn F# but one resource I'm spending more time with is the official Microsoft [.Net Documentation](https://learn.microsoft.com/en-us/dotnet/) and more specifically, the [F# Language Reference](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/).  The language reference material in particular has a lot of good information including code examples and links to the [F# Core Library Documentation](https://fsharp.github.io/fsharp-core-docs/).

However, as I spend more time scouring and reading through all of this, I'm finding it challenging navigating all of the various sites and understanding it.  Much like reading code is an acquired skill, so too is learning to read documentation.  There are conventions, idioms and syntax you need to be familiar with to get the most out of the documentation.  It can be a bit like "Catch-22":  To better understand F#, you need to read the documentation but to understand the documentation, you need to be familiar with the language.  To be clear, this isn't an F#-only paradox - it's common in almost every programming language.

But I've yet to find any resource that helps walk you through the documentation, how it's organized or how to read it.  That's why I thought it would be good for me to document what I'm learning as I read through the F# documentation, not only about the language itself, but also on how to read the docs.

One recent instance of this came when I was learning more about the `DateTime` struct to access various time and date properties and methods.  Let's assume for a moment that we don't have access to any search engines, LLMs, coding agents or AI.  The first question is how do I even find the documentation so I can read it?

A good first place to start is the official Microsoft [F# Documentation](https://learn.microsoft.com/en-us/dotnet/fsharp/) page - I have this one bookmarked in my browser!  From here, you can find lots of good links to the language guide and library reference I mentioned above.  Just as important, you can also find a link to the [.Net library reference](https://learn.microsoft.com/en-us/dotnet/api/) and API browser which is what we want.  From there, we can browse what is contained in the [`System`](https://learn.microsoft.com/en-us/dotnet/api/system?view=net-10.0) namespace which is where we will find our [`DateTime`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime?view=net-10.0) struct.  (I know there was a lot of hand-waving in that paragraph, especially for anyone who is brand new to .NET and/or F# but unfortunately, I'm no expert either!)

On the documenation page for `DateTime`, we can see it's type definition along with what interfaces it implements.  (**Note:** The concept and application of interfaces are completely over my head right now and on my list to eventually learn so I'll refrain from commenting on them here.)  More importantly for me, this page also lists the various contructors, properties and methods available for this type.

On the [`DateTime` Constructors](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.-ctor?view=net-10.0) page, we can see all of the Overloads available to construct a `DateTime` object.  Taking the simpler example of [`DateTime(Int32, Int32, Int32)`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.-ctor?view=net-10.0#system-datetime-ctor(system-int32-system-int32-system-int32)), we can see the type signature of this function:
```
new DateTime : int * int * int -> DateTime
```
Just below this, the documentation helpfully indicates what each parameter is:

> **Parameters**
>
> year `Int32`
>
> The year (1 through 9999).
> 
> month `Int32`
>
> The month (1 through 12).
> 
> day `Int32`
>
> The day (1 through the number of days in month).

This allows us to create an instance of `DateTime` as follows:
```
> open System;;
> let dt = DateTime(2026, 3, 21);;
val dt: DateTime = 3/21/2026 12:00:00 AM
```

Going back to the [definition](https://learn.microsoft.com/en-us/dotnet/api/system.datetime?view=net-10.0) of `DateTime`, we can also examine the type's various [properties](https://learn.microsoft.com/en-us/dotnet/api/system.datetime?view=net-10.0#properties).  Looking at the page for the [`DateTime.DayOfWeek` Property](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.dayofweek?view=net-10.0#system-datetime-dayofweek), we can see that this is an instance property represented by the type signature:
```
member this.DayOfWeek : DayOfWeek
```
where `DayOfWeek` is an [enumerated constant](https://learn.microsoft.com/en-us/dotnet/api/system.dayofweek?view=net-10.0).  Instance properties return the property for a specific instance of the type.  To use it, replace `this` with our object:
```
> dt.DayOfWeek;;
val it: DayOfWeek = Saturday
```
There are also static properties, such as [`DateTime.Today`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.today?view=net-10.0#system-datetime-today) and [`DateTime.Now`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.now?view=net-10.0#system-datetime-now) that return `DateTime` objects based on the current date and time.  Note the `static` keyword in their signatures:
```
static member Today : DateTime
static member Now : DateTime
```
Static properties apply to the type in general, `DateTime` in this case.  They can be used as follows:
```
> let today = DateTime.Today;;
val today: DateTime = 3/20/2026 12:00:00 AM

> let now = DateTime.Now;;    
val now: DateTime = 3/20/2026 5:48:14 PM
```
Returning again to the `DateTime` definition page, we can also see the long list of [methods](https://learn.microsoft.com/en-us/dotnet/api/system.datetime?view=net-10.0#methods) that are available to us.  Like the properties we reviewed, methods come in two flavors:  instance methods, such as [`DateTime.AddDays`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.adddays?view=net-10.0#system-datetime-adddays(system-double)) and static methods such as [`DateTime.IsLeapYear`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime.isleapyear?view=net-10.0#system-datetime-isleapyear(system-int32)).  Let's examine both:

Instance methods, such as `DateTime.AddDays`, will have some reference of returning a "value of this instance" in the method definition.  In addition, we can again look at the method's type signature:
```
member this.AddDays : double -> DateTime
```
Note the `this`.  Like the instance property, instance methods apply to a specific instance of the type.  Reading the signature, we know that we have to pass a `double` to the method to obtain another `DateTime` object like so:
```
> dt.AddDays 2.0;;
val it: DateTime = 3/23/2026 12:00:00 AM {Date = 3/23/2026 12:00:00 AM;
                                          Day = 23;
                                          DayOfWeek = Monday;
                                          DayOfYear = 82;
                                          Hour = 0;
                                          Kind = Unspecified;
                                          Microsecond = 0;
                                          Millisecond = 0;
                                          Minute = 0;
                                          Month = 3;
                                          Nanosecond = 0;
                                          Second = 0;
                                          Ticks = 639098208000000000L;
                                          TimeOfDay = 00:00:00;
                                          Year = 2026;}
```
We can also use the pipe operator (`2.0 |> dt.AddDays`) to achieve the same result.

For static methods, such as `DateTime.IsLeapYear`, the method works on the type in general.  Looking at the type signature, we see:
```
static member IsLeapYear : int -> bool
```
which means that we can use it like:
```
> DateTime.IsLeapYear 2026;;
val it: bool = false
```
or via the pipe operator (`2026 |> DateTime.IsLeapYear`).

This is by no means an exhaustive review of how to use all of the Microsofot .NET or F# documentation but just a reminder to myself that while complicated, there is a ton of good information on the language and associated APIs.  It takes effort to properly understand how it is all organized and how to use it but I think the the investment is worth it.
