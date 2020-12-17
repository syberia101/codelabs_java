summary: Writing tests for the full container
id: full-container-testing
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Full Container Testing
<!-- ------------------------ -->
## Overview
Duration: 15

### What You'll Learn
- How to change the configuration so that tests use all components.

<!-- ------------------------ -->

## Understand the goal

Up to now, we've only written MockMvc tests that target the controllers.  There are times when we might want to test the system beyond the controllers where we use the actual implementations of the components rather than mocks.

## Copy the MockMvc tests

Create a new package to contain the full container tests.  Maybe change the "mvc" part of the package name to "server"?

## Change the annotations on the class

Replace the ```java @WebMvcTest``` annotation with ```java @SpringBootTest``` and remove the ```java @ContextConfiguration``` annotation.

Your class definition will look like this:

```java
@AutoConfigureMockMvc
@SpringBootTest
public class GreetingsTests {
```

## Remove the mock bean and the expectations

You can now return the tests to the way they were.  For example:

```java
@Test
 public void shouldGetInformalGreeting() throws Exception {
   this.mockMvc
           .perform(get("/greet/informal"))
           .andDo(print())
           .andExpect(status().isOk())
           .andExpect(content().string(containsString("CM6713")))
           .andExpect(content().string(containsString("Hi...")));
 }
```

## Run the tests

Again, use gradle's build task to run the tests as part of a build.

### Observe the run times

In the project explorer of IntelliJ, you can see the test report output.  You will find this at:

<project root>/build/reports/tests/test/index.html.

![Test Report Location](/assets/location-of-test-report.png)

The new tests may be faster than the mock tests.  At the moment, the full load is quick because there are few components. The mock tests are slow because the mocks are being created.  As an application grows and infrastrcture layers such as databases become involved, the SpringBootTests may slow to the point where MockMvc tests prove more valuable.

### What about unit tests?

Good question!  Now that we've separated the greeting logic into the separate component, we can test that in isolation.

Go on then - write some unit tests for the DefaultGreetingMaker class.
