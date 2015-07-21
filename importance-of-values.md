---
title: The Importance of Values
publishDate: 2015-06-15
author: Sean Hagstrom
template: essay.jade
---

![](http://4.bp.blogspot.com/_vt749aV4Y7Q/TQ0RGGCoOkI/AAAAAAAAFsA/fKNZwumM1fI/s1600/palace%2Bof%2Bzinn.jpg)

### The Grid

In an ideal system, our workflow would consist of composing together functions.  
For example, code that would normally perform operations like: 

* adding numbers
* manipulating strings
* iterating over data

would now be be modeled as functions.

```coffeescript
add       = (a, b) -> a + b
first     = (a) -> a[0]
lowercase = (a) -> a.toLowerCase()
```

The `add`, `first`, and `lowercase` functions are simple, they take in their arguments and return a value.  
Because of this simplicity it's easy to compose them into more functions.

```coffeescript
shrinkFirst = (a) -> lowercase (first a)
abbreviate  = (a, b) -> add (shrinkFirst a), (shrinkFirst b)
```
This is possible because our functions are following a simple pattern, they all take in values and returning a value.  
With this technique comes power, especially when we start creating our own values for the function composition.  
For example, if we were to create our own value in the system, then we can create functions that use the value between each other.

```coffeescript
person = (age, name) ->
  age: age
  name: name
  
jake = person 22, 'jake'
```

Now we have defined our own value, `person`, and we can create an example `person` named `jake`.  
Since the person value now exists in our system, we'll begin to create functions that rely on a `person` value being passed in, and then returning a `person` value as well.

```coffeescript
ageBy    = (y) -> (p) -> person (add y, p.age), p.name
addTitle = (t) -> (p) -> person p.age, "#{t} #{p.name}"

ageBy1 = ageBy 1
addTitleInd  = addTitle 'Ind.'
```

We start off by defining the functions `ageBy` and `addTitle`, which are functions that return functions (Higher Order Functions). This is useful since we want to partially apply some generic functions, and then pass in a `person` value. In this case we've created a generic aging function, and generic title adding function. We go on to use these functions to build even more functions, and these functions will directly compose with the `person` value.

```coffeescript
jake = person 22, 'jake'

# with variables
indJake = addTitleInd jake
olderIndJake = ageBy1 indJake

# or inlined
ageBy1 (addTitleInd jake)

# or inlined this way
addTitleInd (ageBy1 jake)
```

The example above shows how we use function composition with the `person` value.  
We produce another `person` value whose name is "Mr. Jake", and is a year older than the `jake` value.  
And we do this easily by combining the functions that take in a `person` value and return a `person` value.  

Of course the composition doesn't stop there, for brevity's sake we'll stop here, but we can still continue to compose together more functions if we wished to. But we get a good amount of mileage through just this, and it's all thanks to being able to return values from our functions.

Though there exists a few problems in our system...

___

### The Non-Value Returning Function

Up until now we've gone over examples of how we can compose our functions with values.  
Though all of those examples were simple, synchronous operations. What if we have a operation that should be asynchronous, like reading a file. If that's the case then we're faced with a constraint that limits the ways we can compose our functions.

```coffeescript
readFile './file', (err, data) ->
  # logic
```

Here we've introduced a rather simple, run-of-the-mill example of some asynchronous code.  
The `readFile` function will take in a path to a file as a string, and take in a function that acts as the correspondent of the results. Once the asynchronous operation is finished, the function will be called with the read file data. We then use our own logic, in the body of the passed in function, to access that data. We do this because the `readFile` function cannot return the results at the end of the function, becase the operation is asynchronous.

```coffeescript
append  = (s) -> (f) -> "#{f}#{s}"
prepend = (s) -> (f) -> "#{s}#{f}"

appendFooter = append 'Footer'
prependHeader = prepend 'Header'
```

Before we use the `readFile` function, we'll create some functions that will be meant to compose with file data from `readFile`. We start by creating the functions `append` and `prepend`, which both take in a string and then file data. We'll also create the `prependHeader` and `appendFooter` functions for convenience.  
Now if try to compose these new functions with `readFile`, like we did with our other functions, we'll get errors.

```coffeescript
file = readFile './file', (err, data) ->
  # logic
# file == ???

formattedFile = prependHeader (appendFooter file)
# formattedFile == ???
```

As we've mentioned already, `readFile` doesn't return anything.  
We solely need to rely on the passed in function to receive the results of the asynchronous operation.  
With that architecture forced on us, we're forced to do all of our composition inside the body of the callback function.

```coffeescript
readFile './file', (err, data) ->
  formattedFile = prependHeader (appendFooter data)
``` 

Now this may not like a bad thing at first, but here's a few things we should consider about this code.

#### What does our function take?
We're conflating two things in the function arguments, which are the arguments needed for the computation, as well as the mechanism used for "unwrapping" the asynchronous operation. We're essentially exposing how we're handling the delivery of the asynchronous results through the function arguments instead of through the return value.

**Note**  
We use the term "unwrapping" to depict that the asynchronous operation is a package that contains the results of operation, and we "unwrap" it by waiting until the operation is finished and having the passed in function called with those results

#### What does our function give?
So far we've derived a lot of power from composing together functions that return values.  
When we have functions that don't return anything, we've essentially thrown a monkey wrench into our function composition. And then we're in a situation where we have two kinds of functions and have to tip-toe around the asynchronous functions, which we should avoid.

#### Conclusion
We should prefer an abstraction that allows us to return aynchronous values. This way our asynchronous functions perform their computations and just return a value that represents the pending operation.

___

### The Pending Value

In our case we already know of an existing abstraction that is used as the pending value, it's commonly referred to as a **Promise**. Which means we can begin to use the Promise value as a way to compose together our asynchronous operations with our synchronous ones.

```coffeescript
pending = readFile './file'
pending
  .then (data) -> # do something with data
  .catch (err) -> # do something with err
```

First we'll redefine `readFile` to be a function that returns a pending value, or a **Promise**. Which makes the only arguments it takes in the file path. Once given the file path, `readfile` will return a **Promise**. Then when the operation is finished it will contain the file data. We then go one to use methods on the Promise value to access the results of the operation.

Now the important part here is that we're using **Promises** as the contract between the asynchronous work and the consumer of the asynchronous work. At this point we can go back to defining functions that take in values and return a value, this time with **Promises**.

```coffeescript
promise = readFile './file'
promise
  .then (file) -> prependHeader (appendFooter file)
  .then (formattedFile) -> # do something with formattedFile
```

Here's one example of just using the `then` method to pass the results to our original `appendFooter` and `prependHeader` functions.

```coffeescript
promise
  .then appendFooter
  .then prependHeader
```

And another version of using the **Promise** value to chain together transformations on the file data.

```coffeescript
formatFile = (file) -> prependHeader (appendFooter file)
formatFileAsync = (promise) -> promise.then formatFile
```

Another version where we define two functions. One for unwrapping the **Promise** value with the `then` method, and another function that takes in the file data and performs the transformations.

```coffeescript
readFormat = (path) ->
  promise = readFile path
  formatFileAsync promise
promise = readFormat './file'
promise.then (formattedFile) -> # do something with formattedFile
```

And finally in this example we compose with the **Promise** value in order to create the new asynchronous function `readFormat`. `readFormat` takes in a path and calls `readFile` with that path. We go on to compose with the **Promise** returned from `readFile` with the function `formatFileAsync`.

___

### The Message

The message here isn't necessarily "Use Promises", but more so "Use Values".  
When you're able to represent pieces of your system as values, you're able to compose with those values.  
In the case of asynchronous operations, we're able to represent them as **Promises**.  
With **Promises** we're able to achieve the same kind of composition between our functions.
