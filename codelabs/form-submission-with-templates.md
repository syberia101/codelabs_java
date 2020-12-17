summary: Handling Form Submission With a Template
id: form-submission-with-templates
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Handling Form Submission with a Thymeleaf Template
<!-- ------------------------ -->
## Overview
Duration: 30

### What You'll Learn
- How Thymeleaf can play the captured errors
- How to pass data into a template


<!-- ------------------------ -->

## Understand the goal

We want our forms to be intelligent and provide useful feedback.  We also want to be able to use them to handle updates so we need to pass data into them.

## Move the HTML form to templates

The first thing we need to do is move the form into templates because we want to add Thymeleaf attributes and these will only be acted on if Spring Boot thinks this is a template.

## Convert to Thymeleaf.

There are a number of changes to go through.  The source code can be found on the ```form-processing-with-thymeleaf``` tag.

The HTML file body will end up looking like this:

```HTML
<form action="#" method="post" th:action="@{/greeting-form}" th:object="${greeting}">
    <label th:for="name">Name:</label>
    <input th:field="*{name}" type="text"></input>

    <span th:errors="*{name}" th:if="${#fields.hasErrors('name')}">Name error</span>

    <br>

    <input th:field="*{type}" th:value="formal" type="radio">
    <label th:for="${#ids.prev('type')}">Formal greeting?</label>

    <br>

    <input th:field="*{type}" th:value="informal" type="radio"/>
    <label th:for="${#ids.prev('type')}">Informal greeting?</label>

    <br>

    <input id="submit" name="submit" type="submit" value="Submit"/>

</form>
```

We'll go through these changes separately.

## Change the form element

In the form element, we introduce the ```th:action```  attribute and the ```th:object``` attribute.

```HTML
<form action="#" method="post" th:action="@{/greeting-form}" th:object="${greeting}">
```

Let's examine those two attributes:

*  ```th:action``` overrides the ```action``` attribute and it uses the [Thymeleaf link format](https://www.thymeleaf.org/doc/articles/standardurlsyntax.html).
* ```th:object``` binds an object to the form.  We can then refer to the object's fields in the form.  Note: we have to pass a suitable object into the template.  For a "create" form ,this will typically be an empty object (i.e. created with the default constructor).  The value is the name of the object on the model with '${}'.

## Change the text element

The next step is to change the text element, making sure we refer to the name field of the form object.

```html
<label th:for="name">Name:</label>
<input th:field="*{name}" type="text"></input>
```

In the input field, we now use a ```th:field``` attribute.  Note, that we prefix the field value with "*" rather than "$".  This is because we want to refer to a field within the object.  This notation is basically saying "map the field called 'name' within the object defined at the form level to this input element."

We also need to replace the ```for``` attribute with ```th:for```.

We only have ```th:field```.  This will expand to cover id and name attributes, and set the value to the value of the passed-in field.

## Change the radio button elements

Radio buttons are slightly more tricky because of the way they need to be linked (so that only one is checked at a time).

```HTML
<input th:field="*{type}" th:value="formal" type="radio">
<label th:for="${#ids.prev('type')}">Formal greeting?</label>


<input th:field="*{type}" th:value="informal" type="radio"/>
<label th:for="${#ids.prev('type')}">Informal greeting?</label>
```

The ```th:field``` attribute links to the ```type``` field in the form object and it will generate different ids for the two fields.  In the ```label``` field we need to use the ```#ids.prev``` expression to get the generated id for the previous field.

If we want to set one of these to checked by default, we need the form object to be set to the value that matches that in the ```th:value``` attribute.

## The submit button

The submit button doesn't need to change, but you can research how you might go about changing the button's label using Thymeleaf.

## Add the error handling

We will come back to the HTML to add the error handling.

## Change the controller

The controller needs to change in two places.

### Route to the form

Firstly, we need to handle the route to the form.  We used to do this in the router, but now we need to add an empty object to the model before we route to the form template.

```java
@GetMapping("greeting-form")
  public String serveGreetingForm(Model model) {
    GreetingForm greetingForm = new GreetingForm("", "formal");
    model.addAttribute("greeting", greetingForm);
    return "greeting-form";
  }
```

In this method, we create a new ```GreetingForm``` object with an empty name and the greeting type set to "formal".  We then add it to the model and direct to the template.

### Process the form submission

Now we need to change the method that handles the post.

```java
@PostMapping("greeting-form")
public String handleGreetingForm(@Valid @ModelAttribute("greeting") GreetingForm aForm, BindingResult bindings, Model model) {

  if (bindings.hasErrors()) {
    System.out.println("Errors:" + bindings.getFieldErrorCount());
    for (ObjectError oe : bindings.getAllErrors()) {
      System.out.println(oe);
    }
    return "greeting-form";
  } else {
    return "redirect:greet/" + aForm.getType() + "?name=" + aForm.getName();
  }
}
```

The ```@ModelAttribute``` tells Spring Boot to expect this parameter to be on the model.  A good explanation can be found [here](https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)

The rest of the code is pretty similar to before.  Notice that we don't need to explicitly pass the errors to the view.  We simply return (not redirect) to the form template name.  The errors are passed in the response along with the model.

## Show the errors in the templates

Now we return to the template and look at the line which shows the error.

```HTML
<span th:errors="*{name}" th:if="${#fields.hasErrors('name')}">Name error</span>
```

Now we use the ```th:errors``` attribute and set it to name of the desired field (here 'name').  This will generate the span tag containing an error associated with the named field.  

We then use ```th:if``` and the ```#fields.hasErrors``` function to determine if this field had an error, and if so, include the span element or not.

There are other ways of handling errors.  You can choose to just show all errors in one block rather than the inline example above.

## Recompile and tests

You can now try out the code.  Leave the name field blank to see the error message.

BONUS: Look up examples of how to change the [style of the error](https://www.thymeleaf.org/doc/tutorials/2.1/thymeleafspring.html#field-errors) as well!

## Review

We have linked the form processing to a backing object and enabled inline error reporting.

When testing this, note how the state of the form persists until you explictly reload the page (e.g. go the to address bar and hit 'return').
