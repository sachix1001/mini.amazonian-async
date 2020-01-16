# Amazonian Async
### This was created during my time as a [Code Chrysalis](https://codechrysalis.io) Student

This activity will be a mixture of reading and solving challenges:

1. Part One - Read about Asynchronous JavaScript
1. Part Two - Solve the Amazonian Challenge using Promises & Async/Await

# Part 1 - Asynchronous JavaScript

## Lesson Objectives

By the end of this reading, you should know and understand:

- Why we need to handle asynchronous calls
- How to use callbacks
- How to use promises
- How to use async/await

## Background

We've already seen how higher-order functions can be useful to abstract behaviors while performing synchronous operations, but the most important use of higher-order functions is when doing things _asynchronously_.

### Why are Handling Asynchronous Operations Important?

JavaScript can only do one thing at a time because it is a single-threaded language. Long running operations will result in performance problems described in [Synchronous and Blocking](https://github.com/codechrysalis/students/wiki/Synchronous-and-Blocking). This can cause problems when the application is dependent on asynchronous operations.

One of the most common ways developers interact with asynchronous code in JavaScript is through APIs, which stands for Application Programming Interface. You will build your own APIs when you learn servers, and you will also learn to interact with third party APIs.

In general terms, an API is a set of clearly defined methods of communication among various components. APIs **receive requests** (e.g. "What's the weather?") and **send responses** ("22 degrees!").

Sometimes, this communication can take awhile– and that can cause problems when other code depends on the response from those requests. Because of that, JavaScript has built-in ways to handle this situation.

### How NOT to Handle Asynchronous Operations

Below, there are three examples of how to handle asynchronous code. Because these examples don't actually use an API, we are using [setTimeout](https://developer.mozilla.org/ro/docs/Web/API/window.setTimeout) to cause a delay. This method takes two arguments– a function, and a number (`n`). It will then invoke the function after `n` milliseconds.

Before we show you what to do, here's what NOT to do. Take a look at the function below. It should return a result after three seconds. What happens when you paste the code in your console and run it?

```JavaScript
function getCoffee(num) {
  setTimeout(() => {
    if (typeof num === "number") {
      return `Here are your ${num} coffees!`;
    } else {
      return `${num} is not a number!`;
    }
  }, 3000);
}

console.log(getCoffee(2));
console.log(getCoffee("butterfly"));
```

It returns undefined. The `console.log()` ran before `getCoffee()` had time to receive a response. That's a problem.

## Option 1: Callbacks

Prior to ES6, asynchronous code was handled through callbacks.

In the example below, `getCoffeeCallback` is a function that takes two arguments– the number of coffees AND a callback function. This callback function takes two arguments: `error` and `result`. Depending on the success of the request, it will return the callback function invoked with either the result or the error (and null for the other argument).

Try running the code below in your console!

```JavaScript
function getCoffeeCallback(num, func) {
  setTimeout(() => {
    if (typeof num === "number") {
      return func(`Here are your ${num} coffees!`, null);
    } else {
      return func(null, `${num} is not a number!`);
    }
  }, 3000);
}

getCoffeeCallback(2, (error, result) => console.log(error ? error : result));

getCoffeeCallback("butterfly", (error, result) => console.log(error ? error : result));
```

This time, the code doesn't return undefined. Why? The reason is because of something called the [Event Loop](https://www.youtube.com/watch?v=8aGhZQkoFbQ). Essentially, the function passed into `getCoffeeCallback` was added to a queue. JavaScript's event loop works with that queue to execute callback functions at the right time.

Let's look at one more example. Remember how we were reading files synchronously using `readFileSync`? Because reading files _can_ be a long operation, Node provides both synchronous and asynchronous implementations. The asynchronous version is called `readFile`.

```js
const fs = require("fs");

const result = fs.readFile("index.js", "utf8");
console.log(result);
```

When we use `readFile`, we get `undefined` as our result.

If we check [the docs](https://nodejs.org/api/fs.html#fs_fs_readfile_path_options_callback), we can see why: `readFile` expects a third parameter, which is a callback function that gets passed `error` and `result` parameters.

Just like our `getCoffeeCallback` function, let's give it what it wants– a callback!

```JavaScript
const fs = require('fs');

fs.readFile('index.js', 'utf8', (error, result) => console.log(error, result));
```

Yay! It worked.

## Option 2: Promises

ES6 normalized a much, much better way to do asynchronous operations– Promises.

Take a look at the refactored code below. You'll notice that the higher order function `getCoffeePromise` looks almost identical to the `getCoffeeCallback` function. This time, however, we don't pass in a callback, and we return a `Promise` (which we create with the `new` keyword). This Promise gets passed a function with two arguments: a function we call `resolve` and a function we call `reject`.

The way `getCoffeePromise` is called is a little different as well. Since we're no longer passing in a callback, we need to handle the responses another way– by chaining `.then()` and `.catch()`.

- The `.then()` is printing whatever was passed into the `resolve` function
- The `.catch()` prints whatever was passed into the `reject` function.

Try running this code in your console!

```JavaScript
function getCoffeePromise (num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (typeof num === "number") {
        resolve(`Here are your ${num} coffees!`);
      } else {
        reject(`${num} is not a number!`);
      }
    }, 3000);
  });
};

getCoffeePromise(2).then(result => console.log(result)).catch(error => console.log(error));

getCoffeePromise("butterfly")
  .then(result => console.log(result))
  .catch(error => console.log(error));
```

Note: When you call a promise, you can write it in one line like the first example or on separate lines like the second example. If you choose to put it on separate lines like the second example, just make sure you don't add spaces, comments, or semi-colons in the middle of the chain!

You can read more about [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) here.

## Option 3: Async/Await

ES7 brought another upgrade to the syntax for Promises.

Here is how it works:

1. We define `getCoffeeAsync` as an async function using the `async` keyword.
2. This allows us to call it later using the `await` keyword.
3. The next line of code after the `await` keyword _waits_ until that line is resolved.
4. We can handled the success and failure of a call using the `try` and `catch` keywords.

What makes this upgrade so great is the `await` keyword. Normally, writing `console.log()` outside of a callback or chained Promise method would return undefined. That's not a problem here, because the `await` keyword STOPS the code from running until that line of code is resolved!

Try it in your browser!

```JavaScript
const getCoffeeAsync = async function(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (typeof num === "number") {
         resolve(`Here are your ${num} coffees!`);
      } else {
        reject(`${num} is not a number!`);
      }
    }, 3000);
  });
}

const start = async function(num) {
  try {
    const result = await getCoffeeAsync(num);
    console.log(result);
  } catch (error) {
    console.error(error);
  }
}

start(2);
start("butterfly");
```

Note: You'll notice in this example that we are still returning a Promise in the getCoffeeAsync function. Unfortunately, `setTimeout` doesn't explicitly support async/await (yet!), so we are adding this wrapper to make our code still work.

You can read more about [Async/Await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) here.

## Side-by-Side Comparison

Let's take a look at how all three ways of handling asynchronous JavaScript are used to make the same request to the [Pokemon API](https://pokeapi.co/).

To make these API calls, we are using [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) (which takes a callback) or [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) (which uses Promises).

Test them out in your browser!

### 1. Callback

```JavaScript
function request(callback) {
  const xhr = new XMLHttpRequest();
  xhr.open("GET", "https://pokeapi.co/api/v2/ability/4/", true);
  xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200) {
      (function(response) {
        callback(JSON.parse(response));
      })(xhr.responseText);
    } else {
      callback(xhr.status);
    }
  };
  xhr.send();
}
request((error, result) => console.log(error ? error : result));
```

### 2. Promise

```JavaScript
fetch("https://pokeapi.co/api/v2/ability/4/")
  .then(response => response.json())
  .then(jsonResponse => console.log(jsonResponse))
  .catch(error => console.log(error));
```

### 3. Async / Await

```JavaScript
async function request() {
  const response = await fetch("https://pokeapi.co/api/v2/ability/4/");
  const jsonResponse = await response.json();
  console.log(jsonResponse);
}
request();
```

These look a little different. Why?

These calls return response that aren't immediately useable as JSON– which is common for a lot of APIs.

1. In the first example, it returns information that requires another callback to process. We handle this by passing the response into an immediately invoked anonymous function, and the calling `JSON.parse()` on the result (yikes).

2. In the other two examples, it returns another Promise! We handle this by calling a method called `.json()` on the response from the previous Promise. We then either chain another `.then()` method or use another `await` to get to our results.

You will probably need to process JSON when you work with APIs in the future, so make a mental note to remember the `.json()` method!

## Which You Should Use

Sometimes, you will have to use callbacks. However, try to use Async/Await or Promises when you can. JavaScript syntax gets upgraded for a reason!

Great job making it through this reading! Now, let's test your knowledge of what you learned.

# Part 2 - Amazonian Challenge

## Setup

- `yarn` to install independencies
- `yarn test` to run tests

## Overview

Let's test your understanding of asychronous JavaScript! For this challenge, imagine that you have been hired to work at Amazonian, an online shop where users can buy and rate products.

They store their data in relational format in `.json` files:

- products
- users
- reviews

Each file represents a database table, with an `id` and data.

Your job is to "join" these three tables into the following format:

```js
{
    productName: product.name,
    username: user.username,
    text: review.text,
    rating: review.rating
}
```

Take a look at [ReviewBuilder.js](ReviewBuilder.js). There are four methods here:

- A synchronous solution
- Three asynchronous solutions:
  1. Using callbacks
  1. Using promises (not finished)
  1. Using async & await (not finished)

The synchronous solution for buildReviewsSync is already finished for you. Note that it uses `fs.readFileSync` as its method.

The callback solution, which is also already solved, uses `fs.readFile`, which returns data data asychronously. Read about that how `fs.readFile` works [in the Node documentation](https://nodejs.org/api/fs.html#fs_fs_readfile_path_options_callback).

## Basic Requirements

1. [ ] Read through the code in `ReviewBuilder.js` and in the `helpers/index.js` file.

   - Make sure you understand what is happening in the two solutions written for you.
   - Also make sure you understand the code in the helper file. You will want to use them in the two methods you need to implement.
   - Feel free to pseudocode your understanding of what is happening on each line!

1. [ ] Implement the `buildReviewsPromises` method.
1. [ ] Implement the `buildReviewsAsyncAwait` method.
