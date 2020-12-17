summary: Consuming a REST API with JQuery
id: consuming-REST-API-with-JQuery
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Consuming a RESTful API with JQuery
<!-- ------------------------ -->
## Overview
Duration: 10

### What You'll Learn
- Using JQuery's AJAX support for access a REST APIs
- Injecting the data into a web page

<!-- ------------------------ -->

## Understand the goal

We want to explore different architectures which we vary by altering where the user interface is generated.

With Thymeleaf, we generate the HTML on the server.  When using the server to host APIs, the user interface is generated on the client.

This can reduce the number of round-trips across the network and can lead to a more responsive user interface.  Most modern web applications use client-side frameworks such as React.js, Vue.js, Angular.js or Svelte.  These frameworks add MVC capabilities to the client whereby the user interface reacts to changes in the client-side model.

However, there is also a potential downside as reduced round-trips can allow the client's model to lag behind the server model.  Think about applications like theatre or flight booking systems where events update the shared model (e.g. seats available for a flight).

### Context

If not already watched, take time to view [The Browser is Dead](https://www.youtube.com/watch?v=Bw1dgUS27uE)

## Add a web page

We need a web page that will provide formatting and structure to our content.  This can be a static file as we are not generating any content on the server.

There is nothing to stop us mixing and matching server-side and client-side approaches, so this could be a template.

Be careful with routing.  In the gitlab example, we have added a routing line to the ```StaticRouter.java``` file.

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Greeting with JQuery</title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
    <script src="/js/GreetingApiClient.js"></script>
</head>
<body>
<div>
    <span id="greeting-message"></span>

    <span id="greeting-name"></span>
</div>
</body>
</html>
```

NOTE: There are two spans identified with id attributes that contain no content.  You could put dummy content in the page.

Positive
: What would that allow you to do with the roles in the team?

## Code the Javascript

In the HTML, we included a Javascript file.  Let's add that (to the /js folder under static content).

```Javascript
$(document).ready(function () {
    $.ajax({
        url: "http://localhost:8080/api/greet"
    }).then(function (greeting) {
        $('#greeting-name').append(greeting.name);
        $('#greeting-message').append(greeting.message);
    });
});
```

This JQuery runs when the page is ready.

It calls the API and when the response is received, passes the data received to a function that appends the received data to the spans.  We could replace rather than append.

## Testing

Testing is now different because the processing is distributed between server and client.  The page isn't rendered until the JQuery completes so the server response isn't sufficient to check.

There are other testing tools for this.  These sit at the top of the testing pyramid.  Examples include Selenium and Playwright.

## Review

In this tutorial, we have written JQuery to consume an API and use the received data to populate a HTML page.

While still useful, JQuery is falling out of favour and alternative client-side frameworks are gaining market share.  
