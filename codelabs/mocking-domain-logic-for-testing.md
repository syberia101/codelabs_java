summary: Mocking Domain Logic for Testing
id: mocking-domain-logic-for-testing
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Mocking Domain Logic for testing
<!-- ------------------------ -->
## Overview
Duration: 15

### What You'll Learn
- How to keep using MockMvc tests when the domain layers aren't configured
- Using Mocks to provide suitable component implementations

<!-- ------------------------ -->

## Understand the goal

Our tests started failing when we split the code into components.  This is because MockMvc tests only configure the controller components into the container.  We want to keep using MockMvc tests because they are generally faster to run so we need a way of including the domain components (or at least their behaviour).

## Define a MockBean

Our controller needs a component that implements an interface.  In the test, let's define a MockBean whose type is that of the required interface.

```java
@AutoConfigureMockMvc
@ContextConfiguration(classes = {ExampleController.class})
@WebMvcTest
public class GreetingsTests {

  @Autowired
  private MockMvc mockMvc;

/**
New declaration below...
*/

  @MockBean
  private GreetingMaker theGreetingMaker;
```

The new annotation declares a mock bean that can be given "expectations" in the test methods.  An expectation is a definition of expected behaviour.

For example, a mock bean implementing factorials would be expected to return 6 when called with parameter 3, and 120 when called with parameter 5.  If we called the mock with parameter 2, it wouldn't know how to respond.

This allows us to configure just enough behaviour for the test that we want to conduct.  So, if we were testing that the controller returned "<div>The factorial of 5 is 120</div>" we only need the mock to handle 5.

## Define the behaviour in a test

To define an expectation, we use a when-thenReturn pattern (not a million miles from BDD).

```Java
//...
@Test
 public void shouldGetInformalGreeting() throws Exception {

   when(theGreetingMaker.determineGreetingWithTime("CM6713", Optional.of("informal")))
           .thenReturn(Optional.of(new Greeting("CM6713", "Hi... 25/12/2020 00:00:00AM")));


   this.mockMvc
           .perform(get("/greet/informal"))
           .andDo(print())
           .andExpect(status().isOk())
           .andExpect(content().string(containsString("CM6713")))
           .andExpect(content().string(containsString("Hi...")));
 }
 //...
```

In this case, we need the component to return the default name and an Optional object containing a Greeting object starting "Hi".  Note that we have to get the parameter values in the "when" correct.  If the mock doesn't get those values, it won't return the desired response.

The controller will use the mock and therefore, because we know the expected response from the mock, we can anticipate and test the output of the controller.

## Run the GreetingsTests

Use gradle > tasks > build > build to build the code (including running the tests).

* The tests should now pass.

## Review

We have learnt how to mock a dependency.  This pattern can be repeated at any layer and many times.  For example, if were to need to take credit card payments, you might want to provide your own mock implementation (or ask the service provider if they offer one).  Similarly, you might want to mock a database or a message broker to avoid costly network traffic slowing down your tests.

However, there is a case for wanting to use the real components and writing tests that load the full context will be the subject of a future tutorial.
