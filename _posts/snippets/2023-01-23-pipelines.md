---
title: Pipelines
date: 2023-01-23 15:02:00 -0800
categories: [Snippets, Design, Test]
tags: [patterns, problem-solving]
---

## The Problem
```
Write JavaScript code using the setTimeout function to print 3 lines asynchronously.

The output should do the following:
  1. Wait 2 seconds
  2. Print out “First task done!”
  3. Wait another 2 seconds
  4. Print out “Second task done!”
  5. Wait another 2 seconds
  6. Print out “Third task done!”
```
---
So, I was given this problem a little while ago. It's a deceptively simple problem (for beginners), and I think it gets at core of a common design problem.

In my head, [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) worked like this: Give it some code and a time to wait, and it'll run the code after said time has passed.

After thinking it over for a bit, this was the first thing that popped in my head:
```js
// code to run
function firstTask() {
	console.log("First task done!")
}
function secondTask() {
	console.log("Second task done!")
}
function thirdTask() {
	console.log("Third task done!")
}

// time to wait
setTimeout(firstTask, 2000)
setTimeout(secondTask, 4000)
setTimeout(thirdTask, 6000)
```

Since `secondTask` runs two seconds after `firstTask`, that means it occurs *four* seconds after running the program!!! :DDD

> Yeah, no.

Though it technically works, it doesn't solve the problem at all:



```js
function setTimeoutPromise(func, ms) {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve(func())
		}, ms)
	})
}

function firstTask() {
	console.log("Completed the first task")
}
function secondTask() {
	console.log("Completed the second task")
}
function thirdTask() {
	console.log("Completed the third task")
}

setTimeoutPromise(firstTask, 2000)
.then(() => setTimeoutPromise(secondTask, 2000))
.then(() => setTimeoutPromise(thirdTask, 2000))
```