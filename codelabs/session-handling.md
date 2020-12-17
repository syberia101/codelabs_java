summary: Using Spring Boot Profiles to manage sessions
id: session-handling
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Using Spring Boot Profiles to manage server sessions
<!-- ------------------------ -->
## Overview
Duration: 20

### What You'll Learn
- How Spring declares parts of the MVC model as being session attributes, and how state can be retained between requests.
- How to use the POST-REDIRECT-GET pattern and how Spring supports the retention of session state.

<!-- ------------------------ -->

## Understand the goal

In web applications, the server needs to retain state between requests.  Since HTTP is inherently stateless, this can be a problem.  Sessions provides a logical way for a server to retain state relevant only to the specific client-server interaction.

There are problems with sessions.  As the server-side of an application scales, the requests from a client may need to be served by different servers.  To maintain the session, it needs to remain 'sticky' to one server or the session needs to be available across servers.

The other problem is that they can get big if too much data is stored in the session.  As they stick around until the session times out (~ 30 minutes) the sessions will consume memory which can then cause performance issues or worse.

Ideally, try to minimise the amount of work done on a session and, if you can, clean up the session when safe to do so.

## Populating the model

We've already seen that objects can be added to the model using:

```java
model.addAttribute("key", anObject);
```

We can also tell Spring to ensure that an object is on the model for every request.

This is done by using the ```@ModelAttribute``` annotation on a method within a controller component.

In our example, we are going to add a List of String objects to our model.  We'll record the names that we see in this list and we'll react different to new names than we do to names that we've seen before.  If we see a new name, we'll log "You're very welcome" and if we've seen the name before, we'll log "Welcome back".

Add this method to your controller:

```java
@ModelAttribute("names")
public List<String> names() {
  return new ArrayList<String>();
}
```

With this method in place, every request will now have a "names" list in place.  If it doesn't exist, it will be created using this method.

## Make sure the new model attribute is available in the method

We need to ask Spring to provide the attributes that we need as parameters on the method.  

Positive
: NOTE: the controller doesn't define a field to hold the list of names.  That's because, by default, controllers are singletons.  There is only one instance per server and therefore, we don't want the list of names shared across sessions.

Change the ```GetMapping``` method to:

```java
@GetMapping("post-greeting/{type}")
  public String sayHello(@PathVariable(name = "type", required = false) Optional<String> type,
                         @RequestParam(value = "name", defaultValue = "CM6713") String aName,
                         @ModelAttribute("names") List<String> names,
                         Model model) {


    Optional<Greeting> aGreeting = theGreetingMaker.makeGreeting(aName, type);

    if (names.contains(aName)) {
      System.out.println("Welcome back");
    } else {
      System.out.println("You're very welcome.");
      names.add(aName);
    }

    System.out.println("List of members: " + names);


    if (aGreeting.isPresent()) {
      Greeting theGreeting = aGreeting.get();
      GreetingValue greetingValue = new GreetingValue(theGreeting.getName(), theGreeting.getMessage());

      model.addAttribute("greeting", greetingValue);
    } else {
      return missingGreetingType();
    }

    return "hello";
  }
```

We have a ```@ModelAttribute``` parameter in the list of parameters.  Spring will provide this.  We then use ```names``` to check whether the session has seen the name before.

## Tell Spring to put this attribute on the sessions

The last thing we need to do is to tell Spring to store this attribute on the session.

We do this by annotating the class and specifying which atttributes should go onto the session.

```java
@Controller
@SessionAttributes(names = {"names"})
public class ExampleController {
```

The ```@SessionAttributes``` annotation takes an array of Strings that match to the keys in the model.

With this in place, Spring now knows that, when the method is called it should check the session for the "names" parameter.  It is exists it will provide it.  If not, it will use the ```@ModelAttribute``` method to initialise the list.

## Test the GET request

At this point, we can test the code using ```post-greeting/informal?name=carl``` and similar patterns.

Try repeating the name and then changing it.  You should see different messages in the console.

Try using more than browser and use different names.  What do you see in the console?  Is the list different for each browser?

## Add POST request supports

To make sessions work with POST requests, we need to introduce the concept of 'RedirectAttributes' because we are going to use the POST-REDIRECT-GET pattern.

### POST-REDIRECT-GET pattern

Rather than return a template directly from a POST request, we prefer to redirect the browser to a GET request.  This is reduce the risk of a repeated submission of a POST.  This is risky (think duplicate order or duplicate payment).

The pattern is nicely illustrated at [Wikipedia](https://en.wikipedia.org/wiki/Post/Redirect/Get).

### Add redirect attributes to the controller

```java
//..
@PostMapping("get-greeting")
public String handleGreetingForm(@Valid @ModelAttribute("greeting") GreetingForm aForm,
                                 @ModelAttribute("names") List<String> names,
                                 RedirectAttributes attributes,
                                 BindingResult bindings,
                                 Model model)
//..
```

Notice the new parameter ```attributes```.

Spring will provide this for us.

Just before we do the redirect (which we do on success), we want to put the name into the RedirectAttributes so that the next method receives it when the GET is processed.

```java
//..
else {
      attributes.addFlashAttribute("names", names);
      return "redirect:post-greeting/" + aForm.getType().get() + "?name=" + aForm.getName();
    }
//..
```

```java
attributes.addFlashAttribute("names", names)
``` is the line that adds the item to the list of attributes.

## Test the form

Now test using the form that can be found at ```get-greeting```.  Again, the session should be populated.

Try the same tests as before and observe the behaviour.

## Experiments

Try commenting out the ```@SessionAttributes``` annotation and re-run the server.  Observe the behaviour.

Try adding some ```System.out.println``` lines to the code to see what gets called where.

## Summary

Spring supports sessions in a number of ways.  @SessionAttributes is a declarative way of identifying parts of the model that need to be retained between requests.

## To think about

* Look up how to explicitly flush the session
* Some web journeys need sessions, some don't.  Is this a good reason for having separate controller for different groups of routes?
