---
title: Pipelines
date: 2023-01-23 15:02:00 -0800
categories: [Snippets, Design]
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

## Naive Solution

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

Though it technically works, it doesn't solve the problem at all: the second task doesn't _run_ two seconds after the first one, it _just so happens to_. If I wanted to make `secondTask` run three seconds after `firstTask` instead of two, I'd have to do this:

```js
setTimeout(firstTask, 2000)
setTimeout(secondTask, 5000) // modified
setTimeout(thirdTask, 7000) // ALSO modified!
```

So, no good. What I really needed to do was make each task run _after_ the previous one.

## Callback Chaining

Okay, `secondTask` runs after `firstTask` right?

```js
function firstTask() {
	console.log("First task done!")
	setTimeout(secondTask, 2000)
}
```

Wait.

If I just wanted to run `firstTask`, it would also run `secondTask`. I need the sequencing to happen *outside* of either task.

```js
setTimeout(() => {
	firstTask()
	setTimeout(secondTask, 2000)
}, 2000)
```

Adding `thirdTask` in...

```js
setTimeout(() => {
	firstTask()
	setTimeout(() => {
		secondTask()
		setTimeout(thirdTask, 2000)
	}, 2000)
}, 2000)
```

Now, when I change `secondTask` to three seconds I only do this:

```js
setTimeout(() => {
	firstTask()
	setTimeout(() => {
		secondTask()
		setTimeout(thirdTask, 2000)
	}, 3000) // modified
}, 2000)
```

...And it works!

---
This solved my previous problem, but I still wasn't completely happy with it.

For one, it's hard to read. Every other line is stuffed with boilerplate. It's clearer when you extend the pattern out like this:

```js
setTimeout(() => { // boilerplate
	firstTask()
	setTimeout(() => { // boilerplate
		secondTask()
		setTimeout(() => { // boilerplate
			thirdTask()
		}, 2000)
	}, 2000)
}, 2000) 
```
<small>[This is hell.](http://callbackhell.com/)</small>

From here, it's easy to see the problem. I've got some repetitive code using `setTimeout` that I have to write before every task.

Contrast that with the naive solution from earlier:

```js
setTimeout(firstTask, 2000)
setTimeout(secondTask, 4000)
setTimeout(thirdTask, 6000)
```

It'd be nice if I could chain these tasks together without all the boilerplate.

<small>But, that's not possible, is it?</small>

## Pipelines

> Introducing the JS [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) -- your one-stop shop to call-back for all your callback chaining needs! Call now, and it'll call back as many times as you need!

Allow me to demonstrate.

---
### Recipe: How to Cook a Call-Forward

**Ingredients**
- A stack of boilerplates (covered in callbacks)
- A properly prepared promise

**Steps**

1\. Set your boilerplates down on the table

```js
setTimeout(() => { // boilerplate
	firstTask()
	setTimeout(() => { // boilerplate
		secondTask()
		setTimeout(() => { // boilerplate
			thirdTask()
		}, 2000)
	}, 2000)
}, 2000) 
```

2\. Flatten them out, one by one

```js
setTimeout(() => { // boilerplate
	firstTask()
}, 2000)
setTimeout(() => { // boilerplate
	secondTask()
}, 2000)
setTimeout(() => { // boilerplate
	thirdTask()
}, 2000)
```

3\. Promise each one you'll have the resolve to call-back

```js
new Promise((resolve, reject) => {
	setTimeout(() => { // boilerplate
		resolve(firstTask())
	}, 2000)
})

new Promise((resolve, reject) => {
	setTimeout(() => { // boilerplate
		resolve(secondTask())
	}, 2000)
})

new Promise((resolve, reject) => {
	setTimeout(() => { // boilerplate
		resolve(thirdTask())
	}, 2000)
})
```

4\. Then enjoy your newly cooked call-forward

```js
new Promise((resolve, reject) => {
	setTimeout(() => { // boilerplate
		resolve(firstTask())
	}, 2000)
})
.then(() =>
	new Promise((resolve, reject) => {
		setTimeout(() => { // boilerplate
			resolve(secondTask())
		}, 2000)
	})
)
.then(() =>
	new Promise((resolve, reject) => {
		setTimeout(() => { // boilerplate
			resolve(thirdTask())
		}, 2000)
	})
)
```

---
### Refactoring

Jokes aside, it's now much easier to fix the boilerplate. Instead of writing each promise by hand, let's put it in a function!

```js
function setTimeoutPromise(func, ms) {
	return new Promise((resolve, reject) => {
		setTimeout(() => { // boilerplate (now in one place)
			resolve(func())
		}, ms)
	})
}
```

Now my call-forward <small>\*cough\* pipeline \*cough\*</small> looks like this:

```js
setTimeoutPromise(firstTask, 2000)
.then(() => setTimeoutPromise(secondTask, 2000))
.then(() => setTimeoutPromise(thirdTask, 2000))
```

Modifying `secondTask` is easy:
```js
setTimeoutPromise(firstTask, 2000)
.then(() => setTimeoutPromise(secondTask, 3000)) // modified
.then(() => setTimeoutPromise(thirdTask, 2000))
```

And adding `fourthTask` is too:
```js
setTimeoutPromise(firstTask, 2000)
.then(() => setTimeoutPromise(secondTask, 3000))
.then(() => setTimeoutPromise(thirdTask, 2000))
.then(() => setTimeoutPromise(fourthTask, 2000)) // modified
```

## The Solution

```js
function firstTask() {
	console.log("Completed the first task")
}
function secondTask() {
	console.log("Completed the second task")
}
function thirdTask() {
	console.log("Completed the third task")
}

function setTimeoutPromise(func, ms) {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve(func())
		}, ms)
	})
}

setTimeoutPromise(firstTask, 2000)
.then(() => setTimeoutPromise(secondTask, 2000))
.then(() => setTimeoutPromise(thirdTask, 2000))
```
<small>My final solution. Beatiful, isn't it?</small>

---
So, yeah. Pipelines. Use them when you need to chain anything.

...

Wait, you still don't understand Promises?

A Promise is just a monoid in the category of callbacks, what's the problem?