---
title: Using F# Sequences to Count Instances of the 13th of the Month
subtitle:
author: Samir Parikh
date: 15 March 2026
updated:
tags: [F#]
---

Shortly after publishing yesterday's post on the [lazy evaluation of F# sequences](https://blog.samirparikh.com/2026/03/lazy-evaluation-of-sequence-in-f.html), I came across this [post by Dr. Drang](https://leancrew.com/all-this/2026/03/scientific-american-and-friday-the-13th/) about calculating the frequency by day of week of occurences of the 13th day of the month.  His post was inspired by a *Scientfic American* [article](https://www.scientificamerican.com/article/why-friday-the-13th-is-a-mathematical-inevitability/) about why the 13th of the month falls on a Friday more than any other day of the week (like two days ago from when this post is published).  Spoiler:  while true, it's not by much.

In his post, Dr. Drang offered up a short Python script that calculates the number of times the 13th falls on each day of the week across a [400-year span](https://en.wikipedia.org/wiki/Gregorian_calendar#Description).  He uses a simple imperative script consisting of nested `for` loops to iterate through the years and months and counting up the number of occurences by day in an array:
```
f13s = [0]*7
for y in range(1800, 2200):
  for m in range(1, 13):
    wd = date(y, m, 13).weekday()
    f13s[wd] += 1
```
Given the timeliness of his post as well as my recent forays into functional programming and the use of collections in F#, I thought it would be fun to see what I could come up with:
```
open System

let allDates (startDate: DateTime) (endDate: DateTime) =
    seq {
        let mutable theDate = startDate.Date
        while theDate <= endDate.Date do
            yield theDate
            theDate <- theDate.AddDays 1
    }

let startDate = DateTime(2000, 1, 1)
let endDate = DateTime(2399, 12, 31)

let myDates = allDates startDate endDate

myDates
|> Seq.filter (fun date -> date.Day = 13)
|> Seq.countBy (fun date -> date.DayOfWeek)
|> Seq.sort
|> Seq.iter (fun (dayOfWeek, count) -> printfn $"{dayOfWeek}: {count}")
```
While not as concise as the Python implementation, I thought I did ok.

I started with the now familiar `allDates` function from yesterday's post to define a sequence consisting of all dates within the range we are interested in.  Like the *Scientific American* article, I went for years 2000 through 2399.  Once we have our dates, I apply a filter to find all occurrences of the 13th of the month and then count them by day of week.  After sorting them by day of week, we print our results:
```
Sunday: 687
Monday: 685
Tuesday: 685
Wednesday: 687
Thursday: 684
Friday: 688
Saturday: 684
```
which match both the authors of the article and Dr. Drang![^*]

While this past Friday was the second Friday the 13th of year, we'll have [one more in November](https://www.nytimes.com/2026/03/13/us/friday-13th-three-times.html).  To learn more about why, look no further than [the good Doctor](https://leancrew.com/all-this/2012/07/friday-the-13th-frequency/)!

[^*]: The only difference is in the order of the days.  Whereas Python [`date.weekday()`](https://docs.python.org/3/library/datetime.html#datetime.date.weekday) uses `0` for Monday and `6` for Sunday, .NET [`DayOfWeek`](https://learn.microsoft.com/en-us/dotnet/api/system.dayofweek?view=net-10.0) uses `0` for Sunday and `6` for Saturday.
