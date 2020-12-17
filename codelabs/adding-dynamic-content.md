summary: Adding dynamic content to a Spring Boot Application
id: adding-dynamic-content-to-Spring-Boot
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Adding Dynamic Content
<!-- ------------------------ -->
## Overview
Duration: 5

### What You'll Learn
- Passing dynamic content

<!-- ------------------------ -->
## Amend the controller
All we're going to do in this tutorial is add the ability to show dynamic content.

Let's amend the controller.

```Java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class ExampleController {

  @GetMapping("greet")
  public String sayHello(Model model) {

        String timedMessage = "Hello...it's " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss a"));

        Greeting aGreeting = new Greeting("CM6213", timedMessage);
        model.addAttribute("greeting", aGreeting);

    return "hello";
  }
}
```


### Set the greeting to include the current time.
Rather than pass a static greeting, we're going to provide a timestamp with it.

Nothing else has changed.  The view will still show what it is passed because there
is no change to the objects being passed via the model.

## Test

To test this, run gradle > build > classes again and then go to http://localhost:8080/greet.

Do this a number of times to see the time change.

## Review

In this tutorial, we've introduced the notion of dynamic content.

Each request can now return dynamic data which moves us from web sites to web applications.

At the moment, we are generated the view on the server-side and only asking the browser to render the HTML.

### APIs

Spring Boot is moving more and more towards being a framework for APIs where the HTTP response is not HTML, but is typically JSON.
We will look at this later, but it may be worth you starting to look at web frameworks such as React.js that consume REST APIs.
