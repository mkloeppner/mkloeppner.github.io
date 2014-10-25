---
layout: post
title:  "Swift - Memoize, make use of modern language features"
date:   2014-06-13 22:00:00
categories: Swift
---

Welcome to one more thing regarding Swift.
Apple used an great example to showcase the possibilities of Shift.

It uses higher order functions and generics to achieve something,
that Objective-C is not capable to do that easy.

This example is memoize. So what does memoize do?
To showcase what memoize does we take a deeper look into a simple implementation
of Fibonacci.

# The Fibonacci function

	f(1) = 1
	f(2) = 1
	f(n) = f(n - 1) + f(n - 2) | n > 2

So here we go. The Fibonacci function for a number n is the result of the addition
of the results from f(n - 1) and f(n - 2) as long as n is greater than 2.
The Fibonacci for 1 and 2 is defined as 1.

For the mathematicians of you: This Fibonacci is not complete and
does not work with negative numbers. I just concentrate on the simple case
in order to focus on the tutorial.

A simple implementation of Fibonacci would look like the following:

```swift
func fibonacci(n : Int) -> Int {
	if n > 2 {
		return fibonacci(n - 1) + fibonacci(n - 2)
	}
	return 1
}
```

So what does that mean for our implementation.
Lets say we calculate the Fibonacci of 12, so we need to calculate the Fibonacci of 11 and 10.
In order to calculate the Fibonacci of 11 we have to calculate the Fibonacci of 10 ..., wait, hold on.
Didn't we calculated the Fibonacci of 10 already in order to get the result for the Fibonacci of 12?

Yep we did and this is what makes this simple approach very slowly.
Instead of remember the values we already have calculated and using them
we calculate them again and again.

## Caching results

And this is what memoize solves. Memoize packs a function within itself and remembers
the values that has been calculated already. If an algorithm wants to call itself
recursively for a parameter that has been calculated already,
memoize just returns that value instead of let the algorithm do the hard work again.

But how to implement that? Here is an implementation:

```swift
var resultCache = Dictionary<Int, Int>()
func memoizedFibonacci(n : Int) -> Int {
    if !resultCache.indexForKey(n) {
        if n > 2 {
            resultCache[n] = memoizedFibonacci(n - 1) + memoizedFibonacci(n - 2)
        } else {
            resultCache[n] = 1
        }
    }
    return resultCache[n]!
}
```

Not that bad. The repeated calculations has been removed now but the code seems
complex and does not really seem to be clean.

This comes due the fact that memoizedFibonacci now mixes multiple responsibilities by:

- Running the calculations of Fibonacci itself
- Persisting the result of the Fibonacci calculations
- Determination if the cache provides a result or the calculation needs to run

So with other words there is a lot of boilerplate code around
the simple calculation that our interest spotlights of:

```swift
if n > 2 {
	result = memoizedFibonacci(n - 1) + memoizedFibonacci(n - 2)
} else {
	result = 1
}
```

Another unfortunate result from this implementation is that there is that the
dictionary that saves the result has a global scope.

And last but not least this implementation is not very reusable since the implementation
does not abstract the caching.

So far we can achieve the same results also with Objective-C. So how can Swift help?

## Swift - A modern language

Swift comes with a lot of modern language features and of course not all of them are basically new.
Other languages support those features as well. But Swift brings that features to
their full power. Lets take a deeper look into type inference for example.
Since Swift enforces type safety once a variable or constant has being assigned,
you can work with that variable and pass it around without loosing the type.
You can not imagine what i've seen in JavaScript so often:

```swift
myObject = [3, 5, 6, 8];

// A lot of code here ....

myObject = "Test";
```

myObject is changing its type during runtime and this has its dis- and advantages.
However, if some implementation depends on myObject being an Array after assigning it
to a string our runtime can end up in a unexpected state. Swift helps developers to highlight
the features that they rely on and makes this general assumptions visible. Also Swift does not
support changing the type once a type is declared or being inferred.

But lets come back to our example and Swift's support for higher order functions.

### Higher order functions

Thanks to the language feature called higher order functions we are able to hide
the complexity of caching results.

How do we do that? Its quite simple. We abstract our caching functionality into
a method called memoize. This method takes our Fibonacci
function and calls it if and only if its really necessary by the fact that
no cached value exists.

```swift
func memoize(fibonacci: (Int) -> (Int)) -> (Int) -> (Int) {
    var cache = Dictionary<Int,Int>()
    func memoized(n: Int) -> Int {
        if !cache.indexForKey(n) {
            cache[n] = fibonacci(n)
        }
        return cache[n]!
    }
    return memoized
}

func fibonacci(n : Int) -> Int {
    if (n > 2) {
        return memoized_fibonacci(n - 1) + memoized_fibonacci(n - 2)
    }
    return 1
}

var memoized_fibonacci = memoize(fibonacci)

```

We have serval things to highlight in this code.

```swift
func memoize(fibonacci: (Int) -> (Int)) -> (Int) -> (Int) { }
```
The memoize function receives a method (indicated by () -> ()) and returns a new one
with the same parameter and return type.

```swift
func fibonacci(n : Int) -> Int {
		if (n > 2) {
				return memoized_fibonacci(n - 1) + memoized_fibonacci(n - 2)
		}
		return 1
}
```
Our fibonacci method slightly changed comparing to the simple implementation.
Instead of calling fibonacci within the fibonacci method itself we are accessing
an variable that is not defined yet.

```swift
var memoized_fibonacci = memoize(fibonacci)
```
This variable is the memoized version of fibonacci. Since memoized_fibonacci is now the function
to call in order to run the calculation, it exists when fibonacci is going to call it.

It is valid to access memoized_fibonacci before its being declared, but Swift still check
if memoized_fibonacci is declared and has the right type.

So far very great. Memoize is being abstracted and fibonacci just contains the logic that
matters for the calculation.

### Generics

But still this implementation lacks in a specific detail. It only supports functions they
return an Int and receiving one as parameter. What if a function that works with other data
types wants to memoize its results?

One solution would be to declare multiple interfaces:

```swift
func imemoizei(obj :Int) -> Int {}
func dmemoized(obj :Double) -> Double {}
func fmemoizef(obj :Float) -> Float {}
```

This solution would really end up in declaring a lot of interfaces and redundant code.
Assume that dmemoized and fmemoizef has been copied from imemoizei. If imemoizei contains
issues, dmemoized and fmemoizef inherit them.

That alone would make this approach hard to maintain but whats about
memoize still does not support functions that has different class types for the return value and the parameter.

In this case its just impossible to predeclare all combinations and even we would make use of
code generation there its only possible to support classes that are already known.

Another solution is working with AnyObject:

```swift
func memoized(obj :AnyObject) -> AnyObject() {}
```

So know the implementation would look like:

```swift
func memoize(fibonacci: (AnyObject) -> (AnyObject)) -> (AnyObject) -> (AnyObject) {
// The compiler complains that AnyObject is nor Hashable or Equatable
    var cache = Dictionary<AnyObject, AnyObject>()
    func memoized(n: AnyObject) -> AnyObject {
        if !cache.indexForKey(n) {
            cache[n] = fibonacci(n)
        }
        return cache[n]!
    }
    return memoized
}
```

The compiler won't accept that implementation because Dictionary needs an object type as key
that supports the protocols Hashable and Equatable in order to work as expected.
Of course a solution could be to subclass from AnyObject and implement those for
the key we want to use, but then again, we would create a lot of code for different types.

This is a problem that is being solved by generics. Generics allows us to apply type requirements
to a method, class, enum or struct and also to define constraints so that these types have
to support some protocols or needs to be subclasses of another class.

But how does that look like:

```swift
func memoize<T :Hashable, U>(fibonacci: (T) -> (U)) -> (T) -> (U) {
    var cache = Dictionary<T,U>()
    func memoized(n: T) -> U {
        if !cache.indexForKey(n) {
            cache[n] = fibonacci(n)
        }
        return cache[n]!
    }
    return memoized
}
```

Memoize got some constraints after its function name.

	memoize<T :Hashable, U>

Swift allows to declare a list of class types like T and U or any other symbol a developer
would like to choose and apply constraints to it.

So for our memoize function we know that we have a parameter that we want to use as key
in order to cache or find results. We choose T for that. Since Dictionary requires
keys being an Hashable to work fast and efficient we require T being an Hashable as well.

The value can consists of any type so there is no need for any requirement here.

Once we have declared our generics we can use them in the code just as we would use
class types. We can use these generic class types in order to create an Dictionary or we
can use them in order to declare our inner function memoized in order to pass type information
as well.

Lets sum that up:

- Generics allows to pass variables or constants around without loosing
type information, even through nested declarations
- Generics allows to apply requirements to class types in order to require functionality in an
explicit way

## Using modern language features

So far we have seen some of the new Swift features. Some of the problems can be solved with Objective-C as well.
Higher order functions addresses problems that C blocks also do for example BUT:

In order to achieve that the way using Objective-C is very rough and long comparing to Swift. And some
of the capabilities that Swift provides with Generics aren't possible in Objective-C at all.
There are some implementations of NSArray out there using preprocessing macros to ensure compile time type safety
but this approach is very expensive, since it generates code for every kind of type that an array uses.

And to ensure compile time type safety with NSDictionary for keys and values is just about being impossible
with preprocessing, or did you ever found an implementation of that on Github?

So please stay tuned and make use of the modern language features in order to really benefit from it.
Play around with Swift and experience the new possibilities.
