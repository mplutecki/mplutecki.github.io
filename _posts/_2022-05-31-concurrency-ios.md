---
layout: post
title: "Concurrency on iOS"
date: 2022-05-18 10:13:17 +0200
categories: jekyll update
---

# Overview

On iOS we have several options to execute a code concurrently.
Each one comes with it's own pros and cons.
The introduction of the Swift Concurrency is a ground breaking change.
However old tools will still be used for a long time.
It's important to understand each of them.

# Options

1. Swift concurrency
1. GCD (Grand Central Dispatch)
1. `Operation`
1. `Thread` (very low level, rarely used)

# Swift concurrency

It's the newest options available for apps targeting iOS 15.

<!-- It is relient on the new operating system features called non-blocking threads. -->

It's an improvement over GCD and provides more performance and better reliablity.
An example: thread explosion problem.

## Migrating to Swift Concurrency

Swift compiler already bridges the current `completionHanler` based system APIs to Swift Concurrency automatically.
This pattern is called 'Continuation'.
You can use `withCheckedContinuation`.

# Operation

`Operation` (or `NSOperation`) is an object-oriented model commonly used in Cocoa.

# Grand Central Dispatch

Grand Central Dispatch is a lower-level API based on the concept of.

# A few words on custom threads

Apple encouraged [moving away from using threads][move-away-from-threads] a long time ago.
Working with threads is difficult to get right.
For most cases it's not feasible.
However, there are still niche cases where threads might be useful.
This includes apps with real time data progessing.
In this case make sure you use a minimal number of therads.
Checkout the [thread programming guide][thread-programming-guide].

[thread-programming-guide]: https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i
[move-away-from-threads]: https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ConcurrencyandApplicationDesign/ConcurrencyandApplicationDesign.html#//apple_ref/doc/uid/TP40008091-CH100-SW8
