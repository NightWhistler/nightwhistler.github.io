---
layout: post
title:  "Making the compiler work for you"
date:   2021-09-22 00:00:00 +0100
categories: blog
tags: [development,generics,java]
---

 > Beware of bugs in the above code; I have only proved it correct, not tried it. - Donald Knuth
 
It's been a while since I last worked in Java, and now that I'm on a Java project again it's once again clear to me just how much Java grew as a language. I mean this both in the positive and in the negative sense. It's so much better than it was when I first wrote code in it, but it also shows that it's grown kind of organically. There is a lot of cruft in the language and especially in the libraries that are a legacy of the past but which are still with us today.

One of my favourite examples of this is the `equals` method: 
 
 ```java
public boolean equals(Object other);
```

So it allows you to compare any object to any other object. Sounds sensible, right? Except is it really? The very first thing any implementation of this method does is check if the other object is of the same class. If it's not, they can never be equal, so the answer would always be `false`.

This may sound reasonable at first glance, but it's also the root of a million bugs. It's pretty easy to accidentally do something like this:

 ```java
var a = getThing();
var b = getOtherThing();

if (a.getId().equals(b)) { 
 ...
}
```

Here we're trying to check the IDs, but we forgot the call to `getId()` for `b`.
IDEs like IntelliJ will of course warn you about stuff like this, but that's still just a band-aid on a deeper problem.

*EDIT: There was an incorrect example called `strictEquals` here in earlier versions, which didn't actually work. It's been removed.*

We see something similar to the `contains` method on Collections:

```java
public boolean contains(Object o);
```

This also takes an `Object` as a parameter, which dates back to the days that Java did not have generics yet. This can also easily lead to bugs by checking if things are in the collection that could never be in there.

In VAVR, the equivalent to `Collection<T>` is `Value<T>` and since VAVR didn't have to deal with legacy the types make more sense:

```java
default boolean contains(T element)
```

This means that if you know that a collection can only contain elements of `T`, you can also only ask it if a specific `T` is in it. This makes way more sense and stops a whole class of bugs in its tracks.

At university I took some classes in correctness proofs of software. It was pretty fascinating and it makes a tantalizing promise: if only you manage to express the thing you're trying to build in the (usually arcane) format of a prover tool, it then becomes possible to find inconsistencies in your logic and to give solid proof of correctness.

I have never seen these types of proofs used outside of academia since it simply isn't practical. It's insanely hard to write the proofs, and even when you do the proof is in a language that can't be compiled to code.

We do have another way to enforce correctness though: the type system.
In a sense type systems are actually very close to formal specifications. You set boundaries about what is and isn't allowed and the compiler will try to find any inconsistencies in this.

The main idea is very simple but also very powerful: _make things that should never happen not compile._

In Haskell and Scala this idea is more commonplace, since especially with pure functions the implementation will often simply follow from a robust type definition. These languages also have much more complex type systems: The Scala type system has even been shown to be [Turing Complete](https://michid.wordpress.com/2010/01/29/scala-type-level-encoding-of-the-ski-calculus/). 

Java's type system is less advanced, but since the introduction of Generics we can still come pretty close.

Some examples of things I have used in the past.

Instead of using plain `Long` or `UUID` ids, create a Wrapper class:
```java
public class ID<T> {
   private Long value;
   
   public ID(Long value) {
       this.value = value;
   }
}
```

As you can see, the `T` parameter isn't actually used inside the wrapper. The only reason it exists is so that if a method has a signature:

```java
void processCustomerRecords(ID<Customer> customerId);
```

If you try to call this method with the ID of a User instead, it won't compile.
You can take this pretty far, I created a type called `NonEmptyString` once, so that any place in the system where one of those was passed I didn't need to keep checking for `null` or emptiness. The fact that the object existed meant those checks had already been done and succeeded.

So, in closing I'd like to say this... We have a strong type system, let it work for you! Any bug that is found by the compiler is one case where you don't have to hope that unit tests will catch it. Testing is good, proof is better.
