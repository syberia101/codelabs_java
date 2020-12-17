summary: Handling a Request Parameter
id: handling-a-request-parameter
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Handling a Request Parameter
<!-- ------------------------ -->
## Overview
Duration: 10

### What You'll Learn
- How to map a request parameter into a Controller
- More about conventions and configuration

<!-- ------------------------ -->

## Understand the goal

We want to pass the name of the person as a parameter.

## Amend the controller
Let's amend the controller.  Add a parameter to the method.

```Java
//code omitted
public String sayHello(@RequestParam(value = "name", defaultValue = "CM6713") String aName, Model model) {

  String timedMessage = "Hello...it's " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss a"));
//code omitted
  Greeting aGreeting = new Greeting(aName, timedMessage);
```

* The parameter is called ```aName```.
* It is annotated with the ```@RequestParam``` annotation
** This tells Spring that HTTP param called "name" should be mapped to the "aName" variable...
** and that the default value of this parameter (if it's missing) should be "CM6713".

## Test

To test this, run gradle > build > classes again and then go to http://localhost:8080/greet.

You should see the same as before because Spring will play the default value.

Now, go to http://localhost:8080/greet?name=Patrick

You should see the name change.

Try some other tests.  Maybe change the name of the request parameter?

## Review

In this tutorial, we've introduced the notion of a request parameter.

Why not add some more parameters and really explore how Spring handles it.  
Try passing numbers as parameters.  Can Spring convert the number for you?

### Other things to Try
Remove the "value=" part from the annotation and re-rest.  
Does it work?  

Change the web URL to ?aName=Patrick.  Remember conventions?

The value setting is configuration that overrides the convention that the
parameter name (in code) matches the query string parameter name.
