---
layout: post
title: Object Oriented Programing for Objects
date: 2024-06-06 18:30:00 +0000
tags:
  - javascript
  - teaching
---

Hello, object.

Imagine we're going to tell a computer about you.

1. What’s your name?
2. How old are you?
3. What is your favorite ice cream?

In a programing world with out objects, you only need to create variables and assign values.

```js
let name = "Emmanuel";
let age = 24;
let favoriteIceCream = "vanilla";
```

But, there is a problem when it comes to scalability. What if we want to create many people? **We would need to create more variables**

```js
let name1 = "Emmanuel";
let age1 = 24;
let favoriteIceCream1 = "vanilla";

let name2 = "Peter";
let age2 = 25;
let favoriteIceCream2 = "strawberry";
```

Which, in scalability terms, is a disaster. (An app can have millions of users)

> What if there was a better way to structure our code?

## What is Object-Oriented Programming?

To answer that, we need to ask a more concrete question.

**What is an object?**

Concretely: **An object is a collection of properties and values.**

You are an object, the person next to you is an object. We’re all objects!

You, as an object (**an object called 'person'**), have different properties, such as a _name_, an _age_, and a _favoriteIceCream_.

These same properties can **apply to another person** (a different person who also has a _name_, _age_, and _favoriteIceCream_).

That “mold” of properties (the same properties that can apply to different people) is called a "**class**," and in Javascript, it is defined like this:

```js
function Person(name, age, favoriteIceCream) {}
```

As you can see, defining a class is the same as defining a function since it has input parameters. (Remember that for naming classes, we use the [CamelCase](https://en.wikipedia.org/wiki/CamelCase) format).

As mentioned earlier, people also have characteristics we share with other people (_name, age, favoriteIceCream_), these characteristics are called **properties**, and in Javascript, we define them like this:

```js
function Person(name, age, favoriteIceCream) {
  this.name = name;
  this.age = age;
  this.favoriteIceCream = favoriteIceCream;
}
```

By using the reserved word **this**, we refer to the current object, so the input parameters **will be assigned to that Person object.**

The power of using objects is that we can create new people with a single line of code using the reserved word: **new.**

```js
var Emmanuel = new Person("Emmanuel", 24, "vanilla");
```

And not just that, you can create people without creating so many isolated variables. (I didn’t need 9 variables to store 3 different properties of 3 people, just 3 variables for 3 people).

```js
var Emmanuel = new Person("Emmanuel", 24, "vanilla");
var Peter = new Person("Peter", 27, "lemon");
var Andrea = new Person("Andrea", 18, "peach");
```

Just like we share properties, we also share actions; these actions are called: **methods.** For example, all people in the world can greet each other.

```js
function Person(name, age, favoriteIceCream) {
  this.name = name;
  this.age = age;
  this.favoriteIceCream = favoriteIceCream;
  this.canVote = function () {
    return age >= 18;
  };
}
```

As we can see, now we've added the `canVote` property, but this time it’s not a numeric variable or a string; it’s **a function**, which will execute every time we call that method. We can do it like this:

```js
var Emmanuel = new Person("Emmanuel", 25, "vanilla");
Emmanuel.canVote();
// true
var Andres = new Person("Andres", 17, "Strawberry");
Andres.canVote();
// false
```

We’re using two variables: Emmanuel (_var Emmanuel_) and Andrés (_var Andres_), to store **two different people** who share the same class (they share the `Person` class). That “variable” is what’s called an **object**, which is an **instance of a class.**

If we wanted to access a property of a class, we could do it like this:

```js
var Emmanuel = new Person("Emmanuel", 25, "vanilla");
Emmanuel.age;
// 25
```

Notice the difference between calling a property and calling a method:

```js
var Emmanuel = new Person("Emmanuel", 25, "vanilla");
Emmanuel.age;
Emmanuel.canVote();
```

To call the method we use parentheses, but for the property, we don't.

Remember that a method stores functions, and a property stores numbers, strings, booleans, etc. (but not functions).

---

In conclusion, to avoid creating a million variables in your code. Create Objects instead.

Happy coding, and may your objects always be as delightful as your favorite scoop of ice cream!
