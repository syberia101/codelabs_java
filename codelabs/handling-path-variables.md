summary: Handling Path Variables
id: handling-path-variables
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Handling Path Variables
<!-- ------------------------ -->
## Overview
Duration: 10

### What You'll Learn
- How to extract variables from the URL
- How to handle variations in the path

<!-- ------------------------ -->

## Understand the goal

We want to offer different paths for different types of greeting.  We want to support
both format and informal greetings.

If we go to /greet?name=Pat we'll just see the time.
If we go to /greet/formal?name=Pat we'll say "hello"
If we go to /greet/informal?name=Pat we'll say "hi"

## Amend the controller
Let's amend the controller.  Add a Path Variable parameter and some logic.

```Java
//code omitted
@GetMapping({"greet", "greet/{type}"})
public String sayHello(@PathVariable(name = "type", required = false) Optional<String> type,
              @RequestParam(value = "name", defaultValue = "CM6713") String aName, Model model) {

    String prefix = "";

    if (type.isPresent()) {
      if (type.get().equals("formal")) {
        prefix = "Hello";
      }
      if (type.get().equals("informal")) {
        prefix = "Hi";
      }
    }
```

* This method now support two URKL patterns.
* The type parameter is extracted from the URL.  The name maps to the field between curly braces in the URL.
* We use an Optional<> to handle the potential that the path might be missing from the URL.
* We then have some logic that determines the prefix based on presence and (if present) value.

## Test

To test this, run gradle > build > classes again and then go to http://localhost:8080/greet.

Now, try the following:
* http://localhost:8080/greet/formal?name=Patrick
* http://localhost:8080/greet/informal?name=Patrick
* http://localhost:8080/greet?name=Patrick

Notice how the message changes.

* Try http://localhost:8080/greet/cockney?name=Patrick

You should get the default.  Maybe we'd be better off returning a 404 here because that page (resource) doesn't exist?

## Review

In this tutorial, we've introduced the notion of a path variable and how to deal with variations in the URL.
