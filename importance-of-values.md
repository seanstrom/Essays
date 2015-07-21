---
title: The Importance of Values
publishDate: 2015-06-15
author: Sean Hagstrom
template: essay.jade
---

![](http://4.bp.blogspot.com/_vt749aV4Y7Q/TQ0RGGCoOkI/AAAAAAAAFsA/fKNZwumM1fI/s1600/palace%2Bof%2Bzinn.jpg)

#### Redo this intro

### Composition

In an ideal system, our workflow would consist of composing together functions.  
For example, code that would normally perform operations like: 

* adding numbers
* manipulating strings
* iterating over data

can now be be modeled as functions.

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
This is possible because our functions are following a simple pattern, they all take in values and returning a value.  With this technique comes power, especially when we start creating our own values for the function composition.  
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

We start off by defining the functions `ageBy` and `addTitle`, which are functions that return functions (Higher Order Functions). This is useful since we want to partially apply some generic functions, and then pass in a `person` value. In this case we've created a generic aging function, and generic title function. We go on to use these functions to create functions that will easily compose with the `person` value.

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

The example above shows that we've built functions from functions with composition, and then went on to use those functions to compose with a `person` value. Which produced another `person` value whose name is "Mr. Jake", and is a year older than the `jake` value. Of course it doesn't stop here, we can still continue to compose together more functions. Though the point here is to show that: We're just building functions that take in values and return a value.

Though what happens when we can't return a value? Does such a scenario exist?  
One particular situation comes to mind, which is Asynchronous Programming with the Callback Pattern.

### Asynchronous Programming with the Callback Pattern

Up until now we've gone over examples of how we can be composing our functions with values.  
Though all of those examples were simple, synchronous operations. What if we have a operation that should be asynchronous, like reading a file. If that's the case then we're faced with a constraint that limits the ways we can compose our functions.

```coffeescript
readFile './file', (err, data) ->
  # logic
```

Here we've introduced a rather simple example of some asynchronous callback code. The `readFile` function will take in a path to a file as a string, and take in a function that acts as the correspondent of the results.  

The reason we pass the function in is because `readFile` doesn't return a value, since it's asynchronous and won't have the results to return. Instead we pass in a function that will be called with the results of the read file when finished.

```coffeescript
append  = (s) -> (f) -> "#{f}#{s}"
prepend = (s) -> (f) -> "#{s}#{f}"

appendFooter = append 'Footer'
prependHeader = prepend 'Header'
```

Here we've created the functions `append` and `prepend`, that both take in a string and then file data. We'll also create the `prependHeader` and `appendFooter` functions for convenience. Okay time to compose our functions. How do we do that? Well since the `readFile` function uses the Callback Pattern we can't compose the same way as before. Or else this happens.

```coffeescript
file = readFile './file', (err, data) ->
  # logic
# file == ???

formattedFile = prependHeader (appendFooter file)
# formattedFile == ???
```

As we've mentioned already, `readFile` doesn't return anything.  
We solely need to rely on the callback function to receive the results of the asynchronous operation. With that constraint we're forced to do all of our composition inside the body of the callback function.

```
readFile './file', (err, data) ->
  formattedFile = prependHeader (appendFooter data)
``` then return the correct value

There's a few issues that we should have with this code:

1. `readFile` doesn't return anything. So far we've derived a lot of power from composing together functions. When we have a function that doesn't return anything we essentially through a monkey wrench into environment. Now we have to tip-toe around the fact that we have special functions in the system.

2. We're conflating the way we're handling the asynchronous operation with the input of the function. Normally the arguments we would be used in context for the operation, in this case the callback is being used to deliver the results.

We should prefer an abstraction that allows us to return aynchronous values. This way our functions will worry about their computations, while returning a value that represents the operation. 

### Asynchronous Programming with Promises

When considering the use of an asynchronous abstraction, like we've, described above, there's a popular concept that comes to mind. **Promises**.
Promises will do exactly as we've described. They'll represent the pending value from an asynchronous operation.

```coffeescript
pending = readFile './file'
pending
  .then (data) -> # do something with data
  .catch (err) -> # do something with err
```

Now we've defined `readFile` to be a function that returns a `Promise`, or a pending value. So the only arguments it takes in is the file path. Once given the file path, `readfile` returns a pending value, and when finished it will contain the file data results. We then go one to use methods on the Promise value to access the results of the operation.

Now the most important part here is that we're using the Promise value in our system as the contract, between the asynchronous work and the consumer of the asynchronous work. And because Promises have a defined API and mechanics, we can assume and abstract over the way they work.
At this point we can go back to defining functions that take in values and return a value, this time with Promises.

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

Here's another version of using the promise value to chain together transformations on the file data.

```coffeescript
formatFile = (file) -> prependHeader (appendFooter file)
formatFileAsync = (promise) -> promise.then formatFile
```

Another version where we define to functions, one for unwrapping the Promise value with the `then` method, and another function that takes in the file data and performs the transformations.

```coffeescript
readFormat = (path) ->
  promise = readFile path
  formatFileAsync promise
promise = readFormat './file'
promise.then (formattedFile) -> # do something with formattedFile
```

And finally in this example we compose with the Promise value in order to create the new asynchronous function `readFormat`. All together when use Promises as the return value of from our asynchronous functions, we're allowed the same kind of composition between our functions.

### Values are Important

Now the point here isn't necessarily "Use Promises", but more so "Use Values". Promises are one way of getting the job done here. We could easily use streams or some other value to represent what we've shown. And that would be the primary point. When you're able to represent pieces of your system as values, you're able to compose with those values. In the case of asynchronous operations, we're able to represent them as Promises.

### Redo Outro
