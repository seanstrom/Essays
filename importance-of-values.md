---
title: The Importance of Values
publishDate: 2015-06-15
author: Sean Hagstrom
template: essay.jade
---

![](http://4.bp.blogspot.com/_vt749aV4Y7Q/TQ0RGGCoOkI/AAAAAAAAFsA/fKNZwumM1fI/s1600/palace%2Bof%2Bzinn.jpg)

### Prelude

Most of what's to be covered is an opinionated stance on how I write programs.  
The concepts that will be covered can be applied to other languages, but we specifically target Javascript in the discussion. Also, for the sake of simplicity, certain topics are not explained in depth.  
The fact is some concepts mentioned may need their own entire essay to be explained properly.  
But I have tried to give enough of an introduction to them, while remaining focused on the core idea.  
With all that said, let's begin.  

### Our System

In an ideal system, our workflow would consist of composing together functions.  
For example, code that would normally perform operations like: 

* adding numbers
* manipulating strings
* iterating over data

would now be modeled as functions.

```coffeescript
add         = (a, b) -> a + b
first       =    (a) -> a[0]
lowercase   =    (a) -> a.toLowerCase()
shrinkFirst =    (a) -> lowercase (first a)
abbreviate  = (a, b) -> add (shrinkFirst a), (shrinkFirst b)
```

The functions defined above all follow a simple pattern. They all take in **values** and return a **value**.  
When using that pattern, we're able to use our generic functions to build up abstractions.  
This is especially true when we start introducing our own values into the system.

```coffeescript
# Creating our own value

person = (age, name) ->
  age: age
  name: name
  
jake = person 22, 'jake'

# Creating our functions

ageBy    = (y) -> (p) -> person (add y, p.age), p.name
addTitle = (t) -> (p) -> person p.age, "#{t} #{p.name}"

ageBy1       = ageBy 1
addTitleInd  = addTitle 'Ind.'

# Composing them together

formatPerson  = (p) -> ageBy1 (addTitleInd jake)
formattedJake = formatPerson jake
```

First we begin by defining our own value, `person`, and we create an example `person` named `jake`.  
We continue by defining the functions `ageBy` and `addTitle`, which are functions that return functions.  
We then partially apply these functions in order to create the functions `ageBy1` and `addTitleInd`.  
These will be the functions that will directly compose with the `person` value.

Next we show how to use our new functions in composition with the `person` value.  
We start by creating the function `formatPerson`, which takes in a `person` value and composes it with our functions. Finally we call `formatPerson` with the `person` value `jake`, which was defined in the first step.  
The results are that we've produced another `person` value, one with the name "Ind. Jake", and the age of `23`.  
And we did this easily by combining the functions `ageBy1` and `addTitleInd` inside of `formatPerson`.  

All together, function composition has allowed us to chain together these transformations into our desired result.  
Though there is still a potential flaw in our system's design.  
What would happen if some our functions were not able to return a value?


### The Non-Value Returning Function

So far, we've gone over several examples to show how we can compose our functions with values.  
Though all of those examples were made under the assumption that we can return values from our functions.  
What would happen if we couldn't use our simple pattern in certain places of the program?  
And more importantly does such a scenario exist?

As of matter of fact, that scenario does exist.  
It is likely that there are several variations of this issue, but we're only going to be focusing on one.  
And that variation would be asynchronous programming.

Up until now, we've only showed code samples that were *synchronous* operations.  
What if we have an operation that should be asynchronous, like reading a file?  
If that's the case then we're faced with a constraint that limits the ways we can compose our functions.

```coffeescript
readFile './file', (err, data) ->
  # logic
```

Here we've introduced a rather simple, run-of-the-mill example of the use of some asynchronous code.  
We have the `readFile` function, which will take in a path to a file as a string.  
It will also be passed in a function that will act as the correspondent of the results.  

Once the asynchronous operation is finished, the function will be called with the read file data.  
We then use our own logic, in the body of the passed in function, to access that data.  
We do this because the `readFile` function cannot return the results at the end of the function.  
This is because the operation is asynchronous.

```coffeescript
append  = (s) -> (f) -> "#{f}#{s}"
prepend = (s) -> (f) -> "#{s}#{f}"

appendFooter  = append 'Footer'
prependHeader = prepend 'Header'
```

Above, we begin by creating the functions `append` and `prepend`.  
Both of these functions take in a string, and then return a function that takes in file data.  
Using more partial application, we'll also create the `prependHeader` and `appendFooter` functions.   
These functions are intended to be the ones that will be composed directly with the file data.  
Now that we have defined our functions, we can see what will happen when we compose them with `readFile`.

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

Now this may not seem problematic at first, but there are a few things to consider about this code.

#### 1. What does our function take?

At the moment we're conflating two things in the function arguments: the arguments needed for the computation, as well as the mechanism used for "unwrapping" the asynchronous operation. We're essentially exposing how we're handling the delivery of the asynchronous results through the function arguments instead of through the return value.

**Note**  
We use the term "unwrapping" to depict that the asynchronous operation is a package that contains the results of the operation. We "unwrap" it by waiting until the operation is finished and having the passed in function called with those results

#### 2. What does our function give?

Nothing.  
So far, we've derived a lot of power from composing together functions that return values.  
When we have functions that don't return anything, we've essentially thrown a monkey wrench into our function composition.

#### Conclusion

We should prefer an abstraction that allows us to return asynchronous values.  
This way our asynchronous functions perform their computations and then return a value that we can compose our functions with.


### The Pending Value

In our case we already know of an existing abstraction that is used as the pending value, it's commonly referred to as a **Promise**. Which means we can begin to use **Promises** as a way to compose together our asynchronous operations with our synchronous ones.

```coffeescript
# Calling readFile

promise = readFile './file'

# Composing the Promise with our Functions

promise
  .then (fileData) -> # logic with fileData
  .catch (err) -> # error handling

# or

promise = readFile './file'
promise
  .then (fileData) -> prependHeader (appendFooter fileData)
  .then (formattedFile) -> # logic with formattedFile
  .catch (err) -> # error handling

# or

promise
  .then appendFooter
  .then prependHeader
  .then (formattedFile) -> # logic with formattedFile
  .catch (err) -> # error handling
```

In the first step we redefine `readFile` to be a function that returns a pending value, or a **Promise**.  
That now makes the file path the only argument the function needs now. Once given the file path, `readfile` will return a **Promise**, and when the operation is finished it will contain the file data.  

We then go one to use the methods of the **Promise** to access the results of the operation.   
Above we show how we use these methods to pass the results into our `appendFooter` and `prependHeader` functions.
Now that we've designed `readFile` to be a function that returns a **Promise**, we can now start defining functions that will compose with the **Promises**.

```coffeescript
formatFile = (file) -> prependHeader (appendFooter file)
formatFileAsync = (promise) -> promise.then formatFile

readFormat = (path) -> formatFileAsync (readFile path)

promise = readFormat './file'
promise.then (formattedFile) -> # do something with formattedFile
```

Above we've defined two functions.  
`formatFileAsync` for unwrapping the **Promise** value with the `then` method.  
And `formatFile` which takes in the file data and performs the transformations.  

We then create a new function, `readFormat`, that abstracts over `readFile` and `formatFileAsync`.  
`readFormat` will take in the path to file and call the `readFile` function with that argument.  
We then use the `formatFileAsync` function to compose with the promise from `readFile`.  
Finally we return the **Promise** of the formatted read file.

Now let's compare this to what we originally had.

#### 1. What does our function take?

Our function only receives the arguments needed to perform the operation.  
We're no longer conflating the "unwrappping" mechanism with the function's arguments.

#### 2. What does our function give?

A **Value**. More specifically a **Promise**.  
As we said before, we derive power from being able to compose our functions.  
Before we were limited because we didn't return a value, but now we're able to return a pending value.  
We've gone and removed the monkey wrench that was previously casted into our system.  

**Note**  
The "unwrapping" mechanism is now the **Promise** itself. We've standardized on a way for us to represent pending values as **Promises**, which means the **Promise** can/will understand how to "unwrap" itself.

#### Conclusion

We've successfully taken are formally *Non-Value Returning Function*, and modified it to be a value returning function. Which makes us able to compose more functions with that value. 

### Summary

To be clear, the message here isn't necessarily "use Promises", but more so "use Values".  
When you're able to represent pieces of your system as values, you're able to compose with those values.  
In the case of asynchronous operations, we're able to represent them as **Promises**, and with **Promises** we're able to achieve the function composition we want.
