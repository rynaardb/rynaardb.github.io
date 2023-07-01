---
layout: post
title: "Functional Programming in Swift"
date: 2019-04-02
categories: swift
tags: swift functional-programming
image:
    path: /assets/images/functional-programming-swift.webp
---

[Functional programming](https://en.wikipedia.org/wiki/Functional_programming) is a programming paradigm and perhaps even more so a mindset which helps you structure your programs into separate functions. These functions take some input and return the computed output based on the given input. The most important aspects of functional programming are to prevent side effects and to avoid mutating global state.

Once you start applying functional programming techniques you will find that it makes your code more predictable, safer, easier to debug, test, and maintain. 

This post covers functional programming at a high level, so it’s helpful to consider the concepts in real-world problems.

## Imperative vs Functional

With an imperative programming style, you write sequential code, one line after the other mutating and maintaining state along the way, something we should all be very familiar with.

It looks something like this:

```swift
var character = "Superman"
var skillLevel = 500

print("\(character) with skill level: \(skillLevel)") // Superman with skill level: 500
```

Although technically correct, a more functional way to re-write the above example would look like this:

```swift
func superHero(character: String, skillLevel: Int) -> String {
    return "\(character) with skill level: \(skillLevel)"
}

print(superHero(character: "Superman", skillLevel: 500)) // Superman with skill level: 500
```

The two examples above produce the same output but with the functional approach, you express your intentions with a function that makes it easier to read and to understand what it is supposed to do. The function only acts on the input and in turn produces and returns an output. 

## Side effects and immutability

It is important to remember that with functional programming you are trying to avoid side effects. A side effect is anything a function might do which isn't computing the output from the input given and return that output. In the case of the example below we are mutating the global state from within the function:

```swift
var dcCharacters = [["Superman", 500], ["Batman", 400], ["Joker", 300]]

print(dcCharacters) // [["Superman", 500], ["Batman", 400], ["Joker", 300]]

func increaseSkillLevelOfCharacter(index: Int, increaseBy: Int) {
    if let currentSkillLevel = dcCharacters[0][1] as? Int {
        dcCharacters[index][1] = currentSkillLevel + increaseBy
    }
}

increaseSkillLevelOfCharacter(index: 0, increaseBy: 100)

print(dcCharacters) // [["Superman", 600], ["Batman", 400], ["Joker", 300]]
```

Mutating and maintaining the global state using this approach will cause some undesired side effects as you constantly need to keep track of when and where you have changed the global state. In multi-threaded applications, this can lead to race conditions, deadlocks any other strange behavior which is often hard to debug.

These side effects can be prevented by adhering to a few simple rules: 

* Your functions should only rely on its own input
* Your functions should **not** mutate or change elements outside of themselves
* Your functions should always return some output

## Pure functions

It all boils down to keeping your functions "pure". A pure function is the opposite of side effects. It only depends on the input given, compute on that, and returns the output, nothing else. A function is considered to be pure if the output returned is always the same given the same input, making it predictable.

As shown in a previous example, the following function is a pure function:

```swift
func superHero(character: String, skillLevel: Int) -> String {
    return "\(character) with skill level: \(skillLevel)"
}

print(superHero(character: "Superman", skillLevel: 500)) // This will always produce the same result given the same input
```

Here the `superHero` function is only concerned about its own input which in turn computes and return the output based on the input given. The output of this function will always be the same given the same input, again making it predictable.

## Dangers of in-place mutation

As we've seen, in-place mutation is never a good idea and leads to all sorts of unpredictable behavior which is hard to debug. More often than not you make assumptions of what the data should look like but somewhere in your code you mutated a value and forgotten about it. 

Ever found yourself trying to debug an issue which simply doesn't make any sense and often hard to replicate? Chances are that some in-place mutation for example on a background queue causes a race condition, a common issue which might not always be that obvious.

A common workaround is to make mutable copies of your data structures instead. Going back to one of the earlier examples we could create a mutable copy of our array and mutate that instead:

```swift
let dcCharacters = [["Superman", 500], ["Batman", 400], ["Joker", 300]]

var dcCharactersCopy = dcCharacters

dcCharactersCopy[0][1] = 1000
```

### Persistent data structures for efficient immutability

Our `dcCharacters` array in the above example is now a constant making it immutable and we now have copy we can work with. But not only is this inefficient in terms of memory allocation it also introduces a new set of challenges having to maintain the state of the new mutable copy throughout the rest of the application. This can quickly escalate into a state management nightmare and can get messy rather quickly.

Enter [persistent data structures](https://en.wikipedia.org/wiki/Persistent_data_structure). A persistent data structure is a data structure allowing you to always preserves the previous version of itself when it is modified. Using the correct persistent data structure, you no longer have to create an entirely new copy of the data structure just to mutate for example one value in an array.

The usage of persistent data structures is beyond the scope of this article, but I would suggest diving deeper into [hash trees](https://en.wikipedia.org/wiki/Hash_tree_persistent_data_structure) and [linked lists](https://en.wikipedia.org/wiki/Linked_list).

## First-class and higher-order functions

A higher-order function is a function that has as its input and/or output, other functions. Functions in Swift are first-class values, i.e. functions may be passed as arguments to other functions, and functions may return new functions, also known as closures. This makes Swift a great language choice for functional programming.

## Avoid iteration and loops

In functional programming, we'd like to avoid manual iteration and looping over items whenever possible. The most common way to iterate and loop over a collection would look something like this:

```swift
var onlyDcSuperheros: [SuperHero] = [SuperHero]()

for superhero in superheros {
    if superhero.world == .dc {
        onlyDcSuperheros.append(superhero)
    }
}
```

Swift provides several built-in higher-order functions but the most common are `filter`, `map`, and `reduce` which does all the heavy lifting for you.

### filter

As the name suggests, filter will iterate over a collection and filter out only those items passed as a parameter in the closure, for example, filter out only dc superhero characters in the `superheros` array:

```swift
let superman = SuperHero(name: "Superman", skillLevel: 500, world: .dc)
let batman = SuperHero(name: "Batman", skillLevel: 400, world: .dc)
let joker = SuperHero(name: "Joker", skillLevel: 300, world: .dc)
let ironman = SuperHero(name: "Iron Man", skillLevel: 450, world: .marvel)
let captainamerica = SuperHero(name: "Captain America", skillLevel: 550, world: .marvel)
let thor = SuperHero(name: "Thor", skillLevel: 350, world: .marvel)

let superheros = [superman, batman, joker, ironman, captainamerica, thor]

let dcSuperheros = superheros.filter { $0.world == .dc }
```

### map

Applies transformations to each item in the collection. Here we increase the `skillLevel` by `100` for each of the superhero characters:

```swift
let improvedSuperHeros = superheros.map { $0.skillLevel + 100 }
```

### reduce

Reduces a collection into a single result, in the example below calculating `combinedSkillLevel` for all the superhero characters in our array:

```swift
let combinedSkillLevel = dcSuperheros.reduce(0) { (result, superhero) in
    result + superhero.skillLevel
}
```

## Summary

As with any other programming paradigm, functional programming is not a "one size fits all" solution, and depending on the problem you are trying to solve it might not always the best choice. If however, you need to solve complicated problems at scale where the safety, predictability, and ease of maintenance is key then you might want to consider functional programming.

## Additional Resources

[Swift and the Legacy of Functional Programming by Rob Napier](https://academy.realm.io/posts/tryswift-rob-napier-swift-legacy-functional-programming/)

[Persist data structure](https://en.wikipedia.org/wiki/Persistent_data_structure)
