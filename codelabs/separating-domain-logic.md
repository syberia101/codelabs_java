summary: Separating Domain Logic
id: separating-domain-logic
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Separating Domain Logic into a Component
<!-- ------------------------ -->
## Overview
Duration: 15

### What You'll Learn
- How to declare a dependency using an interface
- How to identify a domain component to Spring
- How to inject the component to meet a dependency
- The consequences of this change on our existing tests

<!-- ------------------------ -->

## Understand the goal

When we looked at the model-view-controller (MVC) pattern, we could see that each part had a distinct role.  However, the roles were still related to the user interface and we had the slightly awkward use of "model" to define two things; the data passed to the view and the internal state of the business or domain.

In this tutorial, we are going to separate the domain logic from how it is presented.  In this case, the domain logic is the determination of the greeting based on the name and the type.

Why might we want to do this? Well, just as we're implementing a web application to play the greeting, we may also want to implement a mobile app for Android to play the greeting...and then one for IoS...and then a native Windows app...and then...etc.

We would not want to replicate the determination of the greeting in every app so we separate it out, and potentially host it as an API that all apps can access.

## Make the dependency explicit

Our controller (or should that be 'controllers') are going to delegate the greeting decision making to another component.  To do that, they are going to state their dependency on an interface.

To do that, we add a private member to the controller of type ```GreetingMaker```.  

```Java
private GreetingMaker theGreetingMaker;
```

This interface provides a single method.

```Java
public interface GreetingMaker {
  Optional<Greeting> determineGreetingWithTime(String name, Optional<String> type);
}
```

The method takes a name and an optional String to indicate the type of greeting.  It returns an optional greeting - optional because the contract of the interface is to only return a greeting if the type is absent or valid.

We want Spring to provide the component that implements the interface.  To do this, we need to provide two things; an annotated class that Spring can register in its container and a mechanism for Spring to inject.

### The annotated class

We need to provide a class with either the ```@Component``` or ```@Service``` annotation - the latter is more descriptive.

```Java
@Component
public class DefaultGreetingMaker implements GreetingMaker {
\\...the implementation of the interface
```

### The mechanism

Spring supports a number of ways to inject dependencies.  Providing a constructor that takes all the dependencies as parameters is one.

This can be annotated with ```@Autowired``` (tell Spring to automatically wire in a component that matches the interface), but officially it isn't required when there is only one constructor.

```Java
@Autowired //not required when there is only one constructor - but makes the code more explicit.
 public ExampleController(GreetingMaker aGreetingMaker) {
   theGreetingMaker = aGreetingMaker;
 }
```

The project on Gitlab provides an example of the implementation of the method.  An alternative design would be to throw an exception if the type was unknown.  This is a question of design and the appropriate use of exceptions for unexceptional conditions.

## Recompile and tests

If the server is running with bootrun,

Run the gradle > build > classes to reload the classes.

Test from the browser.

### Run the build

Now, run the build.  The automated tests will fail.  This is because the current tests only load controller components so the dependency can't be wired.

You will see this indicated in the error messages from the tests.

We will fix this in a future tutorial.

## Review

In this tutorial, we have separated the domain logic into a component class, then injected it using dependency injection.  The controller now makes explicit the interfaces on which is relies and Spring injects a component that provides an implementation of the interface.

The idea of "provided" and "required" interfaces is important when considering the overall architecture of a system.
