---
title: Learning to Read F# Documentation
subtitle:
author: Samir Parikh
date: 20 March 2026
updated:
tags: [F#, documentation]
---

I've come across a lot of good resources in the form of books and blog posts in my journey to learn F# but one resource I'm spending more time with is the official Microsoft [.Net Documentation](https://learn.microsoft.com/en-us/dotnet/) and more specifically, the [F# Language Reference](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/).  The language reference material in particular has a lot of good information including code examples and links to the [F# Core Library Documentation](https://fsharp.github.io/fsharp-core-docs/).

However, as I spend more time scouring and reading through all of this, I'm finding it challenging navigating all of the various sites and understanding it.  Much like reading code is an acquired skill, so too is learning to read documentation.  There are conventions, idioms and syntax you need to be familiar with to get the most out of the documentation.  It can be a bit like "Catch-22":  To better understand F#, you need to read the documentation but to understand the documentation, you need to be familiar with the language.  To be clear, this isn't an F#-only paradox - it's common in almost every programming language.

That's why I thought it would be good for me to document what I'm learning as I read through the F# documentation, not only about the language itself, but also on how to read the docs.

One recent instance of this came when I was learning more about the `DateTime` struct to access various time and date properties and methods.  Let's assume for a moment that we don't have access to any search engines, LLMs, coding agents or AI.  The first question is how do I even find the documentation so I can read it?

A good first place to start is the official Microsoft [F# Documentation](https://learn.microsoft.com/en-us/dotnet/fsharp/) page - I have this one bookmarked in my browser!  From here, you can find lots of good links to the language guide and library reference I mentioned above.  Just as important, you can also find a link to the [.Net library reference](https://learn.microsoft.com/en-us/dotnet/api/) and API browser which is what we want.  From there, we can browse what is contained in the [`System`](https://learn.microsoft.com/en-us/dotnet/api/system?view=net-10.0) namespace which is where we will find our [`DateTime`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime?view=net-10.0) struct.  (I know there was a lot of hand-waving in that paragraph, especially for anyone who is brand new to .NET and/or F# but unfortunately, I'm no expert either!)

On the documenation page for `DateTime`, we can see it's type definition along with what interfaces it implements.  (**Note:** Interfaces are definitely something completely over my head right now and on my list to eventually learn so I'll refrain from commenting on them here with respect to how to read the documentation.)  More importantly for me, this page also lists the various contructors, properties and methods available for this type.
