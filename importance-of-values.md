---
title: The Importance of Values
publishDate: 2015-06-15
author: Sean Hagstrom
template: essay.jade
---

![](http://4.bp.blogspot.com/_vt749aV4Y7Q/TQ0RGGCoOkI/AAAAAAAAFsA/fKNZwumM1fI/s1600/palace%2Bof%2Bzinn.jpg)

### Word of Caution

Most of what's to be covered is an opinionated stance on how I wish to write programs.  
Many of the concepts covered can be applied to many languages, but I specifically target Javascript in the discussion.  

I should also mention, that for the sake of simplicity, certain topics aren't explained in depth.  
The fact is some concepts mentioned may need their own entire essay to be explained properly.  
But I have tried to give enough of an introduction to them, while remaining focused on the core idea.  
With all that said, let's begin.

### Our System

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

This is possible because our functions are following a simple pattern.  
They all take in **values** and return a **value**. With this simple technique comes power, especially when we begin creating our own values. Then we can make functions that use those new values between one another.

```coffeescript
person = (age, name) ->
  age: age
  name: name
  
jake = person 22, 'jake'
```

Here we have defined our own value, `person`, and we created an example `person` named `jake`.  
Since the person value now exists in our system, we'll begin to create functions that rely on a `person` value being passed in, and then returning a `person` value as well.

```coffeescript
ageBy    = (y) -> (p) -> person (add y, p.age), p.name
addTitle = (t) -> (p) -> person p.age, "#{t} #{p.name}"

ageBy1       = ageBy 1
addTitleInd  = addTitle 'Ind.'
```

We start off by defining the functions `ageBy` and `addTitle`, which are functions that return functions. This is useful since we want to partially apply some generic functions, and then later pass in a `person` value. In this case we've created a generic aging function, and generic title adding function. We go on to use these functions to build even more functions, and the functions we make will directly compose with the `person` value.

```coffeescript
jake         = person 22, 'jake'
indJake      = addTitleInd jake
olderIndJake = ageBy1 indJake
```

The example above shows how we've used our new functions to compose with the `person` value.  
We produce another `person` value whose name now has the title "Ind.", and is also a year older than the original `person` value. We do this easily by combining the functions `ageBy1` and `addTitleInd`, both of which take in a `person` value and return a `person` value.

More to the point, our function composition allows us to chain many transformations into our desired result.  
Though there is still a potential flaw in our system. What would happen if some our functions would not be able to return a value?


### The Non-Value Returning Function

Up until now we've gone over examples of how we can compose our functions with values.  
Though all of those examples were simple, *synchronous* operations. What if we have a operation that should be asynchronous, like reading a file. If that's the case then we're faced with a constraint that limits the ways we can compose our functions.

```coffeescript
readFile './file', (err, data) ->
  # logic
```

Here we've introduced a rather simple, run-of-the-mill example of some asynchronous code.  
The `readFile` function will take in a path to a file as a string, and take in a function that acts as the correspondent of the results. Once the asynchronous operation is finished, the function will be called with the read file data. We then use our own logic, in the body of the passed in function, to access that data. We do this because the `readFile` function cannot return the results at the end of the function, becase the operation is asynchronous.

```coffeescript
append  = (s) -> (f) -> "#{f}#{s}"
prepend = (s) -> (f) -> "#{s}#{f}"

appendFooter  = append 'Footer'
prependHeader = prepend 'Header'
```

Above we create some functions that will be meant to compose with file data from `readFile`.  
We start by creating the functions `append` and `prepend`, which both take in a string and then file data. We'll also create the `prependHeader` and `appendFooter` functions for convenience. Now what happens when we try to compose these new functions directly with `readFile`?

```coffeescript
file = readFile './file', (err, data) ->
  # logic
# file == ???

formattedFile = prependHeader (appendFooter file)
# formattedFile == ???
```

Well as we've mentioned already, `readFile` doesn't return anything, so we can't compose with it directly.  
We solely need to rely on the passed in function to receive the results of the asynchronous operation.  
With that architecture forced upon us, we are required to do all of our composition inside the body of the passed in function.

```coffeescript
readFile './file', (err, data) ->
  formattedFile = prependHeader (appendFooter data)
``` 

Now this may not seem problematic at first, but we should consider a few things about this code.

#### 1. What does our function take?
At the moment we're conflating two things in the function arguments: the arguments needed for the computation, as well as the mechanism used for "unwrapping" the asynchronous operation. We're essentially exposing how we're handling the delivery of the asynchronous results through the function arguments instead of through the return value.

**Note**  
We use the term "unwrapping" to depict that the asynchronous operation is a package that contains the results of the operation. We "unwrap" it by waiting until the operation is finished and having the passed in function called with those results

#### 2. What does our function give?
Nothing.  
So far we've derived a lot of power from composing together functions that return values.  
When we have functions that don't return anything, we've essentially thrown a monkey wrench into our function composition.

#### Conclusion
We should prefer an abstraction that allows us to return asynchronous values.  
This way our asynchronous functions perform their computations and then return a value that we can compose our functions with.


### The Pending Value

In our case we already know of an existing abstraction that is used as the pending value, it's commonly referred to as a **Promise**. Which means we can begin to use **Promises** as a way to compose together our asynchronous operations with our synchronous ones.

```coffeescript
pending = readFile './file'
pending
  .then (data) -> # do something with data
  .catch (err) -> # do something with err
```

In the example above, we redefine `readFile` to be a function that returns a pending value, or a **Promise**.  
Which would make the file path the only argument the function needs now. Once given the file path, `readfile` will return a **Promise**, and when the operation is finished it will contain the file data.  
We then go one to use the methods of the **Promise** to access the results of the operation.   
Now that we've designed `readFile` to be a function that returns a **Promise**, we can now start defining functions that will compose with the **Promises**.

```coffeescript
promise = readFile './file'
promise
  .then (file) -> prependHeader (appendFooter file)
  .then (formattedFile) -> # do something with formattedFile
```

Above we have an example of using the `then` method to pass the results to our original `appendFooter` and `prependHeader` functions.

```coffeescript
promise
  .then appendFooter
  .then prependHeader
  .then (formattedFile) -> # use formattedFile
```

This last example uses the **Promise** `then` method to chain together transformations on the file data.

```coffeescript
formatFile = (file) -> prependHeader (appendFooter file)
formatFileAsync = (promise) -> promise.then formatFile
```

Above we define two functions. One for unwrapping the **Promise** value with the `then` method.  
And another function that takes in the file data and performs the transformations. We'll go one to use these functions in to compose with a **Promise** from `readFile`.

```coffeescript
readFormat = (path) ->
  promise = readFile path
  formatFileAsync promise
promise = readFormat './file'
promise.then (formattedFile) -> # do something with formattedFile
```

We now create a new function, `readFormat`, that abstracts over `readFile` and `formatFileAsync`.  
`readFormat` will take in the path to file and call the `readFile` function with that argument.  
We then use the `formatFileAsync` function to compose with the promise from `readFile`, and then we finally return the **Promise** of the formatted read file.

As shown above, we've successfully taken are formally *Non-Value Returning Function*, and was able to compose a transformation with the function's return value. This is thanks to the fact that we've designed our asynchronous function to return a value (**Promise**), instead of nothing.


### Summary

The message here isn't necessarily "use Promises", but more so "use Values".  
When you're able to represent pieces of your system as values, you're able to compose with those values.  
In the case of asynchronous operations, we're able to represent them as **Promises**, and with **Promises** we're able to achieve the function composition we want.
