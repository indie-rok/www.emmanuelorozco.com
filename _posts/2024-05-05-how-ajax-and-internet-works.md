---
layout: post
title: How AJAX and internet works?
date: 2024-05-05 18:30:00 +0000
tags:
  - javascript
  - teaching
---

![AJAX and Internet](./assets/blog/blog-ajax.jpg)

Imagine you're going to send a letter to your crush. You're going to confess your love. What do you do?

1.  You write the letter.
2.  You put it in the mailbox.

And then you wait for a response.

**You don’t know if that response will ever arrive.**

Are you going to just sit there waiting, doing absolutely nothing?

Synchronous programming usually does exactly that.

It executes commands and doesn’t move on to the next command unless the current one has completely finished executing.

```js
writeLetter();
sendMail();
waitForResponse();
// you are blocked here
// go to school waits for waitForResponse to finish
gotoSchool();
buyIceCream();
```

In this case, we would have to wait until our crush feels like responding, whether positively or negatively.

Asynchronous programming DOESN'T work like that.

Instructions are executed, and more instructions continue to run, regardless of whether the current instruction has finished or not.

In the case of our crush, we send the letter and proceed to do other activities. Whenever they feel like responding, they will. We’re not just waiting for a response without doing anything else.

> AJAX (**A**synchronous **J**avascript **a**nd **X**ML) is the implementation of this asynchronous programming technique in Javascript.

Imagine you're writing a comment on Facebook and you hit send.

With a traditional synchronous model, as soon as you hit enter, the request is sent, but the ENTIRE content of the browser updates (you can't do anything until the page refreshes).

With an asynchronous model, as soon as you hit enter, the request is sent, but the entire page does NOT refresh. You could still receive notifications, give likes, etc., regardless of whether the comment is successfully posted or not.

In Javascript, there are many ways to handle asynchronous programming, such as [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) and [Async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function). However, in this article, I will show you a simple one: AJAX.

AJAX is helpful to talk to other servers (triggered by an user action like a click of a button) while mantaining the UI free to do other things. i.e (clicking a like button while scrolling a news feed on Instagram)

We will need the `XMLHttpRequest` Object.

As you may recall, all [objects](https://emmanuelorozco.com/blog/object-oriented-programing-for-objects) are initialized with the reserved word `new`.

```js
const ajax = new XMLHttpRequest();
```

Objects have methods, and two methods of the `XMLHttpRequest` object are `open()` and `send()`.

This means we can do this:

```js
ajax.open("POST", "https://apiserver.com/like");
ajax.send();
```

**`.open()`** prepares the request. In this case, the first parameter is the HTTP verb, and the second is the location where the verb will be executed.

**`.send()`** handles sending the request.

...

The benefing of this approach is that after I send a like I can do other things:

```js
const ajax = new XMLHttpRequest();
ajax.open("POST", "https://apiserver.com/like");
ajax.send();
// async programing
// scroll with not wait to ajax send to finish
scroll();
```

---

In conclusion, With async programming, your code becomes more efficient and responsive, allowing users to interact with the application while background tasks are being processed.
