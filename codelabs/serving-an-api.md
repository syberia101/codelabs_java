summary: Serving a REST API
id: serving-REST-API
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Serving a RESTful API
<!-- ------------------------ -->
## Overview
Duration: 10

### What You'll Learn
- How to write an API using the RestController

<!-- ------------------------ -->

## Understand the goal

We want to explore another role of an application server and that is to serve API requests.

An API serves data rather than content.  The API can be consumed by another process and the data used directly to fulfil another requirement and/or to populate a user interface.  

APIs can service many different clients though sometimes they are better optimised to a certain client.

### Context

[The Back-end for Front-end pattern](https://samnewman.io/patterns/architectural/bff/)

Watch [The Browser is Dead](https://www.youtube.com/watch?v=Bw1dgUS27uE)

## Add the controller

### Create a package

First of all, we're going to create a new package for our API handlers.

* Create a package "api" that is parallel with the package where your previous controllers reside.
 * e.g. if the previous controllers are in a.b.c.web, create a package a.b.c.api.

### Create a class

Create a class called ```ExampleApi```

## Annotate the classes

Add two annotations; ```@RestController``` and ```@RequestMapping("api")```

```Java
//...
@RestController
@RequestMapping("api")
public class ExampleApi {
//...
}
```

These annotations tell Spring that this component is a ```@RestController```.  

A ```@RestController``` returns its results directly on the response rather than forwarding to a view.

The ```@RequestMapping``` annotation tells Spring that all routes to the mapped methods in this class should start with "api".

e.g. ```localhost:8080/api/~~~~```

## Add a method and map it

Now we add a method to handle a request for a default greeting.  

The body of the response should be a Greeting object in JSON format, but we also need to return the HTTP status code.  

So, we use a ```ResponseEntity``` object and using generics, we specify the type of the object that will form the body.

The full method looks like:

```Java
@GetMapping("greet")
 public ResponseEntity<GreetingApiData> getGreeting() {
   GreetingApiData greeting = new GreetingApiData("Marvin", "Hello");
   return ResponseEntity.ok(greeting);
}
```

## Serve the change

As before, if you have your server running with bootrun, you can just run the gradle > build > classes task to reload the server.

## Test the APIs

Go to a browser and browse to ```localhost:8080/api/greet```.

You should see

```JSON
{"name":"Marvin","message":"Hello"}
```

## Write an automated tests

The MockMvc approach works equally well for REST responses.

The assertions now check for JSON data rather than HTML elements.

The HTTP status code is more important given their significance in REST.

Spring Boot integrates a number of options for writing assertions on JSON data.

## Review

In this tutorial, we have created a very simple RESTful API.  

The features of controllers for handling query strings and path variables are all useable with REST.

It is important that you read up on the rules of REST APIs and in particular the use of HTTP verbs and HTTP status code.

### Context

[Roy Fielding's dissertation]([https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)

[Mulesoft Guide to REST APIs]([https://www.mulesoft.com/resources/api/what-is-rest-api-design)

[Red Hat on REST APIs](https://www.redhat.com/en/topics/api/what-is-a-rest-api)
