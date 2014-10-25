---
layout: post
title:  "Swift - The rule of safety and optionals"
date:   2014-06-07 10:49:00
categories: Swift
---

Apple send an enormous signal to the developers community by releasing
Swift.

Since Swift is extending the capabilities comparing to Objective-C its
possible to continue with the current patterns with just changing the syntax.
But doing so is like throwing the advantages of Swift away. Apple released new
features in Swift in order to reduce the amount of code to write, to highlight
intends of developers more efficiently and last but not least in order to
let llvm create much more efficiency code by understanding intends from the syntax
and optimizing the code even before llvm touches it on the low level.

So it is really important to understand the new features and there is no need
for being scared.
Every new language takes time to understand. Remember how long it took to be
fluid with Objective-C if you come from an other language like Java.

The greatest progress in learning a new language can be achieved by just playing
around with it.

This article highlights some language features and compares them with the current
 patterns from Objective-C.

## The rule of safety

Apple has done a great job of taking care about safety and making unsafe
operations visible.

The rule of safety is just that every value needs to be
initialized before working with an instance except they are marked explicitly as
optional.

So lets take a look into a class declaration:

```swift
class Person {

  let birthday :NSDate

  init() {

  } // Property 'self.birthday' not initialized

}
```

Do you wonder why the compiler is complaining? It is the rule of safety that fails.
Imagine your class person has a method that relies on birthday, say for example a method
that calculates the age. The value of that method will be invalid until
the birthday of the person will be set.

```swift
class Person {

    let birthday :NSDate

    init() {

    } // Property 'self.birthday' not initialized

    func age() -> UInt {
        // Age returns an invalid value cause birthday has never been set
        let timePassedSinceBirth = NSDate().timeIntervalSinceDate(birthday);

        ...

        return  age;
    }
    // This is a good example for calculated properties.
    // Read more about that in // TODO insert link

}
```

So what to do now? Should we set the date just to an value that does not make
 any sense? No of course not. We can pass it by the initializer as parameter or
we can mark is at optional which brings us to the next topic.

## Optionals

Optionals allow us to skip the needs for initialization for specific properties
by marking them with '?'.

This both, the rule of safety and optionals, bring Swift a step further then
Objective-C. Of course you can check if a value is set in a class method of a
Objective-C class but just by writing a bunch of code and loosing the intend.

### Compare data safety in Objective-C with Swift

Lets compare the Swift Person with an Objective-C derivat:

```objc
@interface Person

@property(strong, nonatomic) NSDate *birthday;
/**
* Calculates the age from birthday.
* Returns NSNotFound if birthday is not set
*/
@property(nonatomic, readonly) NSInteger age;

@end

@implementation Person

- (NSInteger)age {
  if (!self.birthday) {
    return NSNotFound;
  }
  return (NSInteger) [[NSDate date] timeIntervalSinceDate:self.birthday];
}

@end
```

First of all we have to clarify what do we return when self.birthday is nil.
This is not that obvious. You can use NSNotFound, return just -1 because you come
from a C background or declare your own constant.

This is what Craig Federigi meant with a language 'like Objective-C without the
baggage of C'.

On the other hand, by using NSNotFound or -1 we can't use an NSUInteger anymore
which would make more sense in this case. A Person would not exists if its age is
a negative one. So Objective-C forces us to take another data type just for declaring
that there can be a result or not.

So lets take a look how to do that in swift.
Note that I left extra space comparing to Objective-C code in order to
help readers to read. A direct comparison can be found at the end of that
section.

```swift
class Person {

    let birthday :NSDate? // By adding an question mark

    init() {

    } // not neccessary to initialize anymode

    func age() -> UInt? { // Marking also the return value as optional

        // The if statement just enters if birthday is initialized
        if let instanciatedBirthday = birthday {

          let timePassedSinceBirth = NSDate().timeIntervalSinceDate(birthday);
          ...
          return  age;

        }

        // We are just allowed to return nil cause the return value is optional
        return nil;
    }
    // This is a good example for calculated properties.
    // Mare about that later

}
```

As mentioned I left in a lot of boilerplate just to let the example be
comprehensible for beginners but lets take a look how it looks like
if we compare with the same spacing and intend rules that we used for the
Objective-C example:

```swift
class Person {
    let birthday :NSDate?

    func age() -> UInt? {
        if let instanciatedBirthday = birthday {
          let timePassedSinceBirth = NSDate().timeIntervalSinceDate(birthday);
          ...
          return  age;
        }
        return nil;
    }
}
```

So much smaller with so much intend in it.

'Ohh boooy', I can't believe that I've written this for years in the Objective-C way.
Sorry I've to mention this because an inspiring colleague of mine is saying that always
in kind of situations where he recognizes that paradigm he uses can be achieved in an easier way.

Thats what happend to me right now.

Thanks to Swift we can remove 'the need for documenting optionals' from our Coding-Guidelines.

Its not just I have to write some more code in Objective-C then in Swift.
It is that the code does not clarify my intent so I have to take care about it.
And it is about that the tools does not really help me to prevent this issue from
the beginning.

In worst case I left out the check for the birthday and the comment so that someone
using my class identifies an issue somewhere because a textfield in his application displays 0.
I think I do not have to mention that this is a common issue.

So lets wrap it up:

- Just a single sign (?) marks an attribute declaration as optional.
- In Objective-C it would be necessary to look through the implementation.
- There is no need for an extra constant that indicates the state that no birthday
is available in swift.

This is just the beginning of Swift and the features I presented right now are just the basics.
There is so much more in Swift I didn't mentioned.
