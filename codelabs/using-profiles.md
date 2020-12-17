summary: Using Spring Boot Profiles to support multiple databases
id: using-profiles
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Using Spring Boot Profiles to support multiple databases
<!-- ------------------------ -->
## Overview
Duration: 20

### What You'll Learn
- How to use Spring Boot profiles to vary the database on different environments

<!-- ------------------------ -->

## Understand the goal

We want to enable our application to communicate with different databases in different environments.

We may want to the use the repeatability and speed of H2 when developing and testing, but MariaDB when moving closer to the production environment.

This tutorial will only cover the support of different database connection properties, but profiles can vary the code as well.  This might be useful where the functionality of H2 and MariaDB vary and the application has to do more work in one environment than the other.  

## Split the application.properties

The first thing we need to do is split the properties into those which are common across all environments, and those that may be specific to each environment.

### Leave common properties in application.properties

In our case, we are going to assume that the server port is to be the same on all environments.

```
server.port=9090
```


### Put H2 connection details into application-dev.properties

We create a new file called ```application-dev.properties```.  This implicitly creates a profile called 'dev'.

We put the H2 database connection properties into this file.

```
###  H2 Settings
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=password
```

### Put MariaDB connection details into application-prod.properties

Similarly, we create a new file called ```application-prod.properties```.  This implicitly creates a profile called 'prod'.

We put the MariaDB database connection properties into this file.

```
###  MariaDB Settings

spring.datasource.url=jdbc:mariadb://localhost:3306/greetings
spring.datasource.username=root
spring.datasource.password=comsc
```

## Update build.gradle to provide tasks for each environments

We would like to be able to run a Gradle task that starts the application in a specific profile.  To do this, we add some task configuration to ```build.gradle```.

```groovy
tasks.bootRun.configure {
    systemProperty("spring.profiles.active", "dev")
}

tasks.register("bootRunProd") {
    group = "application"
    description = "Runs the Spring Boot application with the prod profile"
    doFirst {
        tasks.bootRun.configure {
            systemProperty("spring.profiles.active", "prod")
        }
    }
    finalizedBy("bootRun")
}
```

The first ```tasks``` configuration changes the default 'bootrun' task so that it uses the 'dev' profile.

The second ```tasks``` configuration registers a new Gradle task called 'bootRunProd' which will call 'bootRun', but first changes the profile to 'prod'.

We can then run the application in different profiles by running the different tasks.

## Test the application in both modes

Run the application in 'prod' mode and add some Greetings.  You will see the data go into the MariaDB table (using MySQL Workbench maybe).

Stop the application and run it in 'dev' mode.  You won't see the greetings that you've added because the development database is in-memory and refreshed on every application start.

Test the application.

Now, stop the application and restart in 'prod' profile.  Without adding new greetings, go to ```/greetings``` and observe that the previous greetings still show.

## Using profiles from the command line

You can use Spring Boot profiles from the command line as well.  If you build your application (using ```gradle build``` or ```gradle assemble```) you will get an executable JAR file in ```/build/libs```.  Let's assume this Jar file is called ```app.jar```.

Open a command line, and navigate to this folder.

Run the following command:

```
java -jar app.jar --spring.profiles.active=dev
```

or

```
java -jar app.jar --spring.profiles.active=prod
```

depending on the profile that you want to run.

Positive
: This is how I will run your assessment.  I will run from the command line, so you will need to tell me which profile to run with.

## Using Profiles to vary code

You can also use profiles to select components.

Add ```@Profile("dev")``` (at class level) to tell Spring Boot that a component should only be included when running in 'dev' profile.

For example, you may end up with something like this:

```java
@Component
@Profile("dev")
public class DefaultGreetingMaker implements GreetingMaker {
```

See [the Spring Boot documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-profiles) for more details.
