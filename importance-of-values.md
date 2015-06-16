—
title: The Importance of Values
publishDate: 2015-06-15
author: Sean Hagstrom
template: essay.jade
—

In the world of Javascript a lot has happened over the last decade around asynchronous programming and all the different patterns used to program asynchronously. During that time I started programming, and in the last half and I’ve been programming in Javascript. Most of that time I wasn’t very conscious of the discussions being had and decisions being made around Javascript and around asynchronous programming, so when I became more aware of the topics I began to backfill the information. Of course one of the most popular discussions would be the debate around Promises, with or without the A+ Spec, and the traditional approach of CPS with callback functions. Through all of that I of course have picked I side that I prefer, and for a while I was pretty dogmatic about which one everyone should prefer. But know I feel that I’ve reached a point in my career where I realize the true strength in what I want from Promises and how I want to shape all of my code. I know feel that Promises, more specifically A+ Promises, are a means to an end to achieve the design that I want in my system. The design I’m speaking will uses values heavily in between functions and objects as the contract or API between one another. Normally I just have functions that perform operations like transforming data, adding numbers, or manipulating strings, which are easy to model in a Value Driven approach. For example:

```
add = (a, b) -> a + b
n = add 1, 2 # 3
```

The `add` function here is very simple. It takes in two values, adds them together, and then returns the result of that expression. You could even begin to point out that this type of operation would be considered synchronous, meaning it doesn’t do anything asynchronous. Well let’s see some typical asynchronous code.

```
readFile ‘./file’, (err, data) ->
  # do something with err and data
```

This is pretty generic example of some callback code.
We have a `readFile` function that takes in a path to a file as a string, and a function that will be corresponding the results of reading the file. As you can see the function we pass in is responsible for taking the `err` value and the `data` value. Now this may seem completely normal, but let’s ask the question of:
What should we expect the `readFile` function to return?
Now technically we don’t know what it should return, there’s not really a standard on what it should return since what can it return? It’s not going to have the values we care about so what would we expect from it, and if it did return something should we even care about it? My point ultimately here is that we’ve now stopped programming with functions always returning values because we started doing asynchronous work. Now the solution here isn’t to start programming everything with synchronous operations, the asynchronous part here is good, we just want the asynchronous function to return a useful value. This is where I think Promises solve this problem fairly well. Instead of passing in all the arguments with a callback function, we just pass in all the arguments needed to perform the asynchronous operation.

```
file = readFile ‘./file’
file
  .then (data) -> # do something with data
  .catch (err) -> # do something with err
```

In this scenario we now just pass the path string to the `readFile` function. `readFile` will return a Promise because it’s performing an asynchronous task. We then use the `.then` and `.catch` methods to access the Promises pending value when it is finished reading file. Now the most important take away here is that we’re using the Promise type in our system as the value, or contract, between the asynchronous work and the consumer of the asynchronous work. And because Promises have a defined API and mechanics, we can assume and abstract over the way they work. We now can keep composing our functions, just like we would with synchronous code, but with asynchronous functions that rely on the Promise type. Being able to compose my functions this way is very important to me. I feel that having this high compose-ability leads to more modular code, that’s easily kept decoupled by the boundaries of your returning values. Unfortunately you won’t be able to achieve the same amount of power with using callbacks as your primary pattern.
Callbacks code is limited by the fact that they don’t have a way to state that the asynchronous functions are doing work that is pending. In order to convey this idea of pending work through the function we would need to introduce the concept of the asynchronous function, that does pending work, to return a pending value that corresponds to that pending work.

```
pendingValue = readFile ‘./file’
pendingValue.done (err, data) ->
  # do something with err and data
```

Which hypothetically will work, but all I’ve done in this case is decided to use a different type to represent the value, rather than a Promise. Which I think is totally fine, in fact there are other constructs that you can use to do this, Streams for example.	

```
file = readFile ‘./file’
file.map (data) -> # do something with data
file.errors (err, push) -> # do something with err
```
** Stream API courtesy of Highland.js

In this scenario the `readFile` function returns a stream, we then use that the Stream value as the contract between me and the function, and then I use the API defined by the Stream type to access the values. Which still fits within the constraint of having my functions always returning values, and when I want to compose my asynchronous functions, I can use the Stream type as the contract between all my functions as well.

Now at this point my main philosophy, as you can tell, is to encourage myself and others to really focus on using values as the contract between my functions, with that we achieve a greater amount of compose-ability and in IMO easier to maintain and test code.

Thanks.

Notes:
Pictures
Code Samples
Theme