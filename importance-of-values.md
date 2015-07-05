---
title: The Importance of Values
publishDate: 2015-06-15
author: Sean Hagstrom
template: essay.jade
---

![](http://4.bp.blogspot.com/_vt749aV4Y7Q/TQ0RGGCoOkI/AAAAAAAAFsA/fKNZwumM1fI/s1600/palace%2Bof%2Bzinn.jpg)

#### Redo this intro

## Using Values

In an ideal system, my workflow would consist of using values between my functions or objects, and these values would serve as the contracts between all of my code. For example, code that would normally perform operations like: 

* iterating through data
* adding numbers
* manipulating strings

can now be be modeled as functions or object methods that take in values and return a value.

```coffeescript
add       = (a, b) -> a + b
first     = (a) -> a[0]
lowercase = (a) -> a.toLowerCase()
```

The `add`, `first`, and `lowercase` functions are very simple, they all take in the values they need and return a value. Since these functions are that simple it's easy to compose them to build more functions.

```coffeescript
shrinkFirst = (a) -> lowercase (first a)
abbreviate  = (a, b) -> add (shrinkFirst a), (shrinkFirst b)
```

Composing functions with values is a very simple technique, but the simplicity carries a great amount of power when you start using more meaningful values. For example if we were to create our own value in the system, we can create functions that use that value as the contract between all the functions we want to compose.

```coffeescript
person = (age, name) ->
  age: age
  name: name
  
jake = person 22, 'jake'
```

Now we have defined our own type of value, which is a `person`, and we can created an example person `jake`.
Let's start making some functions that can rely on a `person` value being passed in, and then returning a `person` value as well.

```coffeescript
ageBy    = (y) -> (p) -> person (add y, p.age), p.name
addTitle = (t) -> (p) -> person p.age, "#{t} #{p.name}"
```

Here we've created the functions `addTitle` and `ageBy` which are functions that return other functions.  
This is useful for when I want to set defaults for the composing functions that take in a `person` value.
For example, if I wanted to age a `person` value by one year, and didn't want to hardcode into my function the number `1`. The same notion applies to the `addTitle` function. The concept can be explained in more depth, but the gist here is to use the more abstract functions to build simpler, composable functions.

```coffeescript
addMr  = addTitle 'Mr.'
ageBy1 = ageBy 1
```

Here we have the functions `addMr` and `ageBy1`, which are simple functions that will compose the `person` value in the system.

```coffeescript
jake = person 22, 'jake'

# with variables
mrJake = addMr jake
olderMrJake = ageBy1 mrJake

# or inlined
ageBy1 (addMr jake)

# or inlined this way
addMr (ageBy1 jake)
```

Now we've successfully built functions from functions, and then went on to use those functions to compose a `person` value. In this case we've taken the `person` value `jake` and made them "Mr. Jake", who's now also one year older.
And this process can be build upon over and over again because we're simply using values to compose with our functions.

Using this technique we're able to build many modular functions that are able to easily compose with one another, which in my opinion leads to more expressive and maintainable code. Though in many of the programs we write, we run into a situation where we no longer can compose our functions in the same way. That situation would be asynchronous programming.

## Async Functions

So far I've gone over many examples of how and why we should be composing our values with functions, but all of those examples were just simple computations that didn't need to be asynchronous. When we have computations that should be asynchronous, like reading a file, then we're faced with a constraint that doesn't allow us to naturally compose with the asynchronous operation like we would above.

```coffeescript
readFile './file', (err, data) ->
  # do something with err and/or data
```

Here we've introduced a rather simple example of some asynchronous code. The `readFile` function will take in a path to a file as a string, and take in a function that acts as the correspondent of the results. The reason we pass the function in is because readFile doesn't return a value that pertains to the operation it's performing. So we pass in a function that will be called with the results of reading file when finished. This pattern is commonly referred to as the Callback Pattern. To drive the point even more home lets show an example of trying to compose with the value of `readFile`.

```coffeescript
append  = (s) -> (f) -> "#{f}#{s}"
prepend = (s) -> (f) -> "#{s}#{f}"
```

Now we have created the functions `append` and `prepend`, that both take in a string and then file data.
Let's try to use these with `readFile`

```coffeescript
file = readFile './file', (err, data) ->
  # Do something err and data
formattedFile = prepend 'Header', (append 'Footer', file)
```

As we can see `readFile` doesn't return anything useful for our functions to compose with. Since the function is asynchronous it doesn't have the file data results yet and it needs a way for us to give it instructions for when the asynchronous task is finished. In this case we have to pass a callback function just to get the results back, which means we can't compose with the values the same way we did before. We're now forced to do all of our composing inside the body of the callback function. This is where I think Promises solve this problem fairly well. Instead of passing in all the arguments with a callback function, we just pass in all the arguments needed to perform the asynchronous operation.

```coffeescript
pending = readFile './file'
pending
  .then (data) -> # do something with data
  .catch (err) -> # do something with err
```

In this scenario we now just pass the path string to the `readFile` function.  
`readFile` will return a Promise because it's performing an asynchronous task.  
We then use the `.then` and `.catch` methods to access the Promises pending value when it is finished reading file.   Now the most important take away here is that we're using the Promise type in our system as the value, or contract, between the asynchronous work and the consumer of the asynchronous work. And because Promises have a defined API and mechanics, we can assume and abstract over the way they work.

We now can keep composing our functions, just like we would with synchronous code, but with asynchronous functions that rely on the Promise type. Being able to compose my functions this way is very important to me.  
I feel that having this high compose-ability leads to more modular code, that's easily kept decoupled by the boundaries of your returning values. Unfortunately you won't be able to achieve the same amount of power with using callbacks as your primary pattern. Callbacks code is limited by the fact that they don't have a way to state that the asynchronous functions are doing work that is pending.

In order to convey this idea of pending work through the function we would need to introduce the concept of the asynchronous function, that does pending work, to return a pending value that corresponds to that pending work.

```coffeescript
pendingValue = readFile './file'
pendingValue.done (err, data) ->
  # do something with err and data
```

Which hypothetically will work, but all I've done in this case is decided to use a different type to represent the value, rather than a Promise. Which I think is totally fine, in fact there are other constructs that you can use to do this, Streams for example.

```coffeescript
file = readFile './file'
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
___
NOTES

I think we should be trying to focus more on how we're able to compose values with functions more.
We should try to build more and more abstractions with brief explanations, and show why composing with values is very applicable for many programs. Then we should introduce how Asynchronous functions that use callbacks, contaminate that flow. Then show how promises are able to work within that flow because they're also values.
