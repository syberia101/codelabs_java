summary: Handling Form Submission
id: form-submission
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Handling Form Submission
<!-- ------------------------ -->
## Overview
Duration: 30

### What You'll Learn
- How to consume a form posted from a browser to the server
- How Spring Boot maps the form elements to an object
- How Spring Boot can validate the input
- How Spring Boot captures errors


<!-- ------------------------ -->

## Understand the goal

Handling user input is a common requirement and handling a form submission is a common way of implementing user input.  In this tutorial, we will look at different ways in which this can be implemented (from the simple to the more complicated).

## Submit a plain HTML Form

First of all, we are going to handle form submission from a plain HTML form.  This code can be found at tag ```plain-HTML-form``` in the Spring Boot examples.

## Write a plain HTML form

In the static folder, create a new HTML file called ```greeting-form.html```.  The body element should contain this form element.

```HTML
<form action="/greetingRequest" method="post">
    <label for="name">Name:</label>
    <input id="name" name="name" type="text"/><br>
    <label for="formal">Formal greeting?</label>
    <input checked id="formal" name="type" type="radio" value="formal">
    <br>
    <label for="informal">Informal greeting?</label>
    <input id="informal" name="type" type="radio" value="informal"/>
    <br>
    <input id="submit" name="submit" type="submit" value="Submit"/>
</form>
```

There are no Thymeleaf attributes in this form.

## Write a controller method to handle the POST

Let's write the simplest possible controller method.  We'll receive the form, and redirect to another request passing the parameters as path variables and/or query parameters.

```Java
@PostMapping("greetingRequest")
 public String handleGreetingForm(GreetingForm aForm, Model model) {
   return "redirect:greet/" + aForm.getType() + "?name=" + aForm.getName();
 }
```

### Things to note  

We are using a new annotation here; ```java @PostMapping```.

The parameters on the method are a ```GreetingForm``` object - the class of which we need to write - and the model.  Like other controllers, the string that we pass back is the name of the template or a web directive (e.g. a redirect).

The ```GreetingForm``` class looks like this:

```java
//...
@Data
public class GreetingForm {
  private String name;
  private String type;
}
//...
```

What's important to notice here is that the HTML form element names match the field names on the form class.  Again, this is convention-over-configuration.  Spring Boot will map the HTML request into a form object so when the start writing code, we already have the data in an object form.

### Add a route

To complete this exercise and test it, you need to add a route to the router.  Map "greeting-form" to "greeting-form.html".

```Java
registry.addViewController("/greeting-form").setViewName("forward:/greeting-form.html");

```

## Run and test the code

Run the server and open the new form in a browser (i.e. go to localhost:8080/greeting-form).

Enter some data and observe the behaviour.  Try changing field names and observe what happens.

## Add validation

We now want to validate the input data.  This code can be found at tag ```validate-HTML-form-validation``` in the Spring Boot examples.

### Add validation annotation to the GreetingForm

We want to prevent the entry of a blank name into the form.

Amend the GreetingForm class...

```Java
import javax.validation.constraints.NotBlank;

@Data
public class GreetingForm {

  @NotBlank
  private String name;
  private String type;
}
```

Notice the new annotation.  You will also need to add the validation dependency to build.gradle.

In the dependencies block, add:

```groovy
implementation 'org.springframework.boot:spring-boot-starter-validation'

```

### Enable validation in the controllers

We can tell Spring Boot to validate the GreetingForm by adding the ```java @Valid``` annotation to the parameter.  We also add the ```BindingResult``` object as a parameter.

```java
@PostMapping("greetingRequest")
  public String handleGreetingForm(@Valid GreetingForm aForm, BindingResult bindings, Model model) {

    if (bindings.hasErrors()) {
      System.out.println("Errors:" + bindings.getFieldErrorCount());
      for (ObjectError oe : bindings.getAllErrors()) {
        System.out.println(oe);
      }
      return "redirect:greeting-form";
    } else {
      return "redirect:greet/" + aForm.getType() + "?name=" + aForm.getName();
    }
  }
```

We can now examine the bindings object to see if there were any errors.  If there are errors, we can iterate through them and print them to the console, then redirect back to the form.

### Run and test

Recompile your code (gradle > tasks > build > classes) to initiate a server reload, then test with a browser.

Leave the name field empty and observe the console output.

You should see output similar to:

```
Errors:1
Field error in object 'greetingForm' on field 'name': rejected value []; codes [NotBlank.greetingForm.name,NotBlank.name,NotBlank.java.lang.String,NotBlank]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [greetingForm.name,name]; arguments []; default message [name]]; default message [must not be blank]
```

## Review

So far, we've processed a standard web form containing no Thymeleaf attributes.  We've validated the form data and logged the error.  Next, we want to show the errors to the user so we need to convert our static HTML form to a Thymeleaf template.

There are a number of parts to this change, so we'll put them in a separate tutorial.
