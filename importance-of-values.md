---
title: The Importance of Values
publishDate: 2015-06-15
author: Sean Hagstrom
template: essay.jade
---

![](http://4.bp.blogspot.com/_vt749aV4Y7Q/TQ0RGGCoOkI/AAAAAAAAFsA/fKNZwumM1fI/s1600/palace%2Bof%2Bzinn.jpg)

#### Redo this intro

### Using Values

In an ideal system my workflow would consist of composing together functions with values, and these values would serve as the contracts between all of my code. For example, code that would normally perform operations like: 

* adding numbers
* manipulating strings
* iterating over data

can now be be modeled as functions that take in values and return a value.

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

Here we've taken our simple functions and composed into more functions.  
Composing functions is a simple technique, but the simplicity carries a great amount of power when you start using more meaningful values. For example if we were to create our own value in the system, then we can create functions that use the value as the contract between each other.

```coffeescript
person = (age, name) ->
  age: age
  name: name
  
jake = person 22, 'jake'
```

Now we have defined our own type of value, `person`, and we can created an example `person` named `jake`.  
We can now begin to create functions that rely on a `person` value being passed in, and then returning a `person` value as well.

```coffeescript
ageBy    = (y) -> (p) -> person (add y, p.age), p.name
addTitle = (t) -> (p) -> person p.age, "#{t} #{p.name}"
```

Here we've created the functions `ageBy` and `addTitle` which are functions that return functions.
This is useful since we want to create a generic functions for a `person` value, but don't want to hardcode minor details like number of years or titles. Instead the functions will take in those minor details as arguments, and then return a function that will take in a person value. This concept can be explained in more depth, but the gist here is to use the more abstract functions to build functions that will easily compose with the `person` value.

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

As you can we've built functions from functions and then went on to use those functions to compose into `person` value. In this case we've taken `jake` and made the name "Mr. Jake", and increased the age by one year.  
All we've done here so far is make small functions with a single responsibility, and compose them into larger functions that perform specific tasks. This is made possible because we've focused on creating functions that take in values and return a value.

Though what happens when we can't return a value? Does such a scenario exist?  
One particular situation comes to mind, which is Asynchronous Programming with the Callback Pattern.

### Asynchronous Programming with the Callback Pattern

Up until now we've gone over examples of how we can be composing our functions with values.  
Though all of those examples were simple, synchronous computations. What if we have a computation that should be asynchronous, like reading a file. Then we're faced with a constraint that doesn't allow us to naturally compose with the asynchronous function like we would above.

```coffeescript
readFile './file', (err, data) ->
  # logic
```

Here we've introduced a rather simple example of some asynchronous callback code.  
The `readFile` function will take in a path to a file as a string, and take in a function that acts as the correspondent of the results. The reason we pass the function in is because `readFile` doesn't return a value, so we pass in a function that will be called with the results of the read file when finished.

```coffeescript
append  = (s) -> (f) -> "#{f}#{s}"
prepend = (s) -> (f) -> "#{s}#{f}"

prependHeader = prepend 'Header'
appendFooter = append 'Footer'
```

Here we've created the functions `append` and `prepend`, that both take in a string and then file data.  
We'll also create the `prependHeader` and `appendFooter` functions in order to compose with the file data.  
Now since the `readFile` function uses the Callback Pattern we can't compose our functions like we normally would.

```coffeescript
file = readFile './file', (err, data) ->
  # logic
# file == ???

formattedFile = prependHeader appendFooter file
# formattedFile == ???
```

As we can see here `readFile` doesn't return anything.  
We solely rely on the callback function to receive the results of the async operation.  
This unfortunately means we can't compose with the values the same way we did before.  
Now we're forced to do all of our composition inside the body of the callback function.

```
readFile './file', (err, data) ->
  formattedFile = prependHeader appendFooter data
``` 

The issue that can be seen here is that now we have functions that don't return values, and we need to pass in a function just to receive the values. Because of these conditions we're not allowed to compose our functions from the return values, and have to compose through the callback chain. The downside there are two things:

1. The Callback Pattern structures our code in a way that builds up cyclomatic complexity. We are more or less trying in a situation where the more we do in each callback, the more context we need to keep track of in your head.
2. We end up making the asynchronous function responsible for its computation, and how it should be passing along the values from the asynchronous operation.

We should prefer an abstraction that allows us to return aynchronous values, and have our functions only worry about that their computation returns the correct value.

### Asynchronous Programming with Promises
Promises do exactly as we've described, they represent the pending value from the asynchronous operation.

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

### Redo Outro

Notes:
Pictures
Code Samples
Theme
___
NOTES

I think we should be trying to focus more on how we're able to compose values with functions more.
We should try to build more and more abstractions with brief explanations, and show why composing with values is very applicable for many programs. Then we should introduce how Asynchronous functions that use callbacks, contaminate that flow. Then show how promises are able to work within that flow because they're also values.
