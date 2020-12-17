summary: Adding Error Handling
id: error-handling-in-controllers
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Error Handling in Spring MVC
<!-- ------------------------ -->
## Overview
Duration: 10

### What You'll Learn
- How to handle a request for an unavailable resource using redirects.

<!-- ------------------------ -->

## Understand the goal

We want to offer different paths for different types of greeting.  
We want to support both format and informal greetings.
We'll assume that if the type isn't specified, that we want informal.
If a type is specified, it must be formal or informal.

If we go to /greet?name=Pat we'll say "hi".
If we go to /greet/formal?name=Pat we'll say "hello"
If we go to /greet/informal?name=Pat we'll say "hi"
If we go to /greet/cockney?name=Pat we'll get a 404 error as that resource (/greet/cockney) doesn't exist.

## Amend the controller
Let's amend the controller.  We'll change the logic in the controller and refactor out two private methods.

```Java
//code omitted
String prefix = "";
    if (type.isPresent()) {
      switch (type.get().toUpperCase()) {
        case "FORMAL": {
          prefix = "Hello";
          break;
        }
        case "INFORMAL": {
          prefix = "Hi";
          break;
        }
        default:
          return missingGreetingType();
      }

    } else {
      return assumeInformal();
    }
```

```Java
private String assumeInformal() {
  return "redirect:/greet/informal";
}


private String missingGreetingType() {
  return "redirect:/404";
}
```


* This method now redirects request missing a type to the informal resource
* This method now directs unknown greeting types to a 404 route.
** What else do we have to do here?

### Add the 404 page

There is a simple static 404.html file in /resource/static.

Is this enough?

### Route the 404 request

Just like we had to do with "/about-us", we need to add a route to the StaticRouter class.

```Java
registry.addViewController("/404").setViewName("forward:/404.html");
```

## Test

To test this, run gradle > build > classes again and then go to http://localhost:8080/greet.

Now, try the following:
* http://localhost:8080/greet/formal?name=Patrick
* http://localhost:8080/greet/informal?name=Patrick
* http://localhost:8080/greet?name=Patrick

This last option should now redirect to http://localhost:8080/greet/informal?name=Patrick but it doesn't as there is a bug in the code.

Negative
: Can you fix the bug?  HINT: consider adding a parameter and passing the name through.

* Try http://localhost:8080/greet/cockney?name=Patrick

You should get the a 404 error now.

## Review

In this tutorial, we've introduced the notion of error handling.
Spring Boot has a number of options for handling errors, some using exceptions and/or
a technique called Aspect-oriented Programming or AOP.  AOP is another powerful feature
of Spring that allows for the provision of common code that spans many classes.

Look up the use of a ControllerAdvice to see a good example.
