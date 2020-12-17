summary: Adding templates and a controller to a Spring Boot Application
id: adding-templates-and-controller-to-Spring-Boot
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Adding a Template and Controller to a Spring Boot Application
<!-- ------------------------ -->
## Overview
Duration: 10

### What You'll Learn
- Adding a controller to handle the request
- Populating a "model" to pass to the view
- Adding a template to Spring Boot
- Injecting model data into the view

<!-- ------------------------ -->
## Adding a controller
The first thing we want to do is add a class that can respond to web requests.
Spring's support for this is via the Spring MVC framework.
MVC stands for "Model-View-Controller" which is a well-known design pattern for separating
responsibility in an application.  It originated with the Smalltalk langauge in the 1970s
and has been adopted and adapted by many frameworks across many platforms.

* Create a class called "ExampleController" in the "web" package (assuming you've followed the previous tutorials)
* Add this code: (add your own package line at the top)

```Java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class ExampleController {

  @GetMapping("greet")
  public String sayHello(Model model) {

    Greeting aGreeting = new Greeting("CM6213", "Hello");
    model.addAttribute("greeting", aGreeting);

    return "hello";
  }
}
```
Let's note a few things about this.

### The class annotation
The class is annotated with the ```@Controller``` annotation.  This tells Spring
that this class provides methods to handle web requests.  At this point in
time, we won't need to know any more.

### The method annotation
The ```sayHello``` method is annotated with the ```@GetMapping``` annotation.
This tells Spring that GET requests to the address "/greet" should be directed to
this method to handle.

### The model parameter
At the moment, we're not passing any parameters over the request, so the only parameter
is a Model.  This class is provided by Spring and is injected automatically for us.

The Model class allows the controller to bundle a set of objects that can be
passed to the view.

### The method code
The code is quite simple.  It constructs a new ```Greeting``` object which is
a Lombok-ed class containing two fields.  (You need to code this yourselves).

We then add this object to the model using the ```addAttribute``` method.  This
takes a key and a value.  The view will get the object according to the key.

### Specifying the next View
The final line of the method tells us the name of the view that should be
shown next.  In this case, "hello".  Spring will look for a template file called
"hello.html".  Again, it will do this by convention, so it will look in a known place.

## Adding the template
That known place is the ```resources/templates``` folder.

In here, we place a hello.html file.

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>

<div th:text="${greeting.message}"></div>
<div th:text="${greeting.name}"></div>

</body>
</html>
```

This is fairly standard HTML, except for the ```th:text``` attributes.
This is the Thymeleaf template engine.  It will replace the content of the DIV elements
with the values of the fields in the object that it references with the "greeting" key.

Thymeleaf has the advantage that you populate the content in the template and preview it
as a static page with dummy content.

Positive
: What do you think is the advantage of this?

## Test

To test this, run gradle > build > classes again and then go to http://localhost:8080/greet.

## Review

In this tutorial, we've introduced all of the basic parts of the MVC framework.

* The model: the Greeting object is added to the Model object.
* The view: the hello.html file provides the view and injects data from the Model
* The controller: the ExampleController class handles the request, decides what to do, populates the model and decides which view comes next.

Positive
: In this example, the controller has done all the work itself.  We would prefer the controller to stick to a discrete set of responsibilities.  It should delegate to other objects for the core processing.
