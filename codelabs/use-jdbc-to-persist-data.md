summary: Use JDBC to save data to a relational database
id: use-jdbc-to-persist-data
categories: Week 4
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Use JDBC to persist data to a relational database
<!-- ------------------------ -->
## Overview
Duration: 180

### What You'll Learn
- How to configure Spring Boot to use a H2 database
- How to set up a test database
- How to create a Repository component
- How to inject the JdbcTemplate into the Repository component
- How to inject the Repository component into the application
- How to change the architecture so that the core defines the abstractions

Positive
: This is a long tutorial.  Take your time and take breaks to allow the new information to embed.


<!-- ------------------------ -->
## The Goal of this tutorials

In this tutorial, we are going to complete our first end-to-end slice.  Up to now, everything has been done using hard-coded data.  We have introduced dependency injection and Spring's Autowiring approach, and we are going to continue with that pattern as we introduce a new layer to our application.

This new layer is the data access layer and will consist of a component called a repository.  Unsurprisingly, this component will be annotated with the ```@Repository``` annotation.  From that point, all the usual rules apply.  The component can be injected into other components if the interface that it implements matches that required by the dependent component.

### H2

For this tutorial, we are going to use the H2 database.  This is a relational database that runs in-memory.  Spring Boot can be configured to start a H2 database when it starts (or restarts) the application.  This means that data doesn't persist between restarts which makes it ideal for testing as each new start puts the database into a clean state as required for testing.

As you will see later, changing to MySQL, Maria, Oracle or any other relational database is (mostly) a matter of changing the dependencies and the application properties.  However, some databases offer specific SQL extensions and using those may require change to the SQL queries that are written.  

Positive
:Choosing to put the responsibility into a specific database would be an architectural choice.

## Start from a working application

Make sure you have a working "Greeting" application before starting this tutorial.  It doesn't need to have the full feature set, just enough to play a greeting as a response.

## Add the required dependencies to the projects

Currently, our project has no database access dependency and Spring's tools don't support the addition of dependencies via the wizard.

You can always create a new project that contains the dependencies and then copy and paste the required lines across, or you can search the Maven repository for the dependencies.

* Go to [Maven Repository](https://mvnrepository.com/)
* Search for "Spring Boot JDBC"
* Select "Spring Boot Starter JDBC"
* Select the latest release
* Click the "Gradle" tab
* Copy and paste the code provided into the dependencies section of build.gradle.

```groovy
// https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-jdbc
compile group: 'org.springframework.boot', name: 'spring-boot-starter-jdbc', version: '2.3.4.RELEASE'
```

In the provided example, I've converted this to the more modern "implementation" approach and combined the group and name into one string using the ":" to separate group name from artifact name.

```groovy
dependencies {
    //...
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
```

### Repeat for H2

The gradle from the Maven Repository is:

```groovy
// https://mvnrepository.com/artifact/com.h2database/h2
testCompile group: 'com.h2database', name: 'h2', version: '1.4.200'
```

Sometimes you need to think about how you're using the dependency.  In our case, we want the H2 database to be available when we run our application, not just when we compile tests (as implied by Maven Repository).  Therefore, in our example, we will replace ```testCompile``` with ```runtime```.  Our build.gradle file's dependencies section will look like this:

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    runtime 'com.h2database:h2'

    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

Our new dependencies are the two lines in the middle.

## Configure the databases

We now need to configure the database so that the application can access it.

We are going to edit the file at ```/src/main/resources/application.properties```.

We need to set the driver, URL, user id and password for the database.

Add the following lines:

```
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=password
```

### Run the application and access the console

* Run the application.
* Go to [http://localhost:8080/h2-console](http://localhost:8080/h2-console).

You will see this login page.

![h2-console](/assets/h2-console-login.png)

Click ```Connect``` and you will a simple DB console application.  You won't have any tables to access yet.  We'll do that next.

## Adding test data

At the moment, our database is empty, but we can ask Spring Boot to run DB scripts at startup, just by providing them in a standard location (more convention over configuration).

* Add a file called ```schema.sql``` to ```/src/main/resources```.
* Add this DDL to the file.

```sql
SET MODE MySQL;
SET IGNORECASE=TRUE;

CREATE TABLE IF NOT EXISTS `greeting` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  `message` VARCHAR(100) NOT NULL,
  `timestamp` TIMESTAMP NOT NULL,
  PRIMARY KEY (`id`))
ENGINE = InnoDB;
```

* This sets a couple of properties; we want to use MySQL SQL dialect and we want to ignore the case in queries.
* We then create a table.

### The Greeting table

The table contains 4 fields.

* The ```id``` field is a number and is the primary key of the table.  We want the database to allocate a new value to this for new rows.
* The ```name``` and ```message``` fields are character-based and will map to the fields of the same name in the ```Greeting``` class.
* The ```timestamp``` field is added by the application so that we have the timestamp is a data type that can do date functions (e.g. order by)

### Run the application

If you run the application, and access the h2-console again, you will see the table (which will be empty).

### Adding test data

So far, we've created a schema.  We can also add test data by providing a file called ```test.sql``` in the same folder as the ```schema.sql``` file.  This file should contain SQL to insert/update data.  We don't need it in our example, so don't include the file (Spring Boot will complain if it sees an empty file called ```data.sql```.)

## Add a Repository component

We're now going to add a component.  The role of this component is to extract data from the database and present it to us as objects.  It will offer our application an interface to retrieve and store data from the database in a variety of ways.

### What drives the design of the repository interface?

An aside from the coding.  Now we have to decide what the interface should look like.  We can drive this in at least two ways; inside-out or bottom-up.

* Inside-out: the design is driven by what the core of our application requires
* Bottom-up: the design is driven by what our repository wishes to provided

We are going to take an inside-out approach in this example.

### Add an interface in the domain package

Since we are taking an inside-out approach, the core of our application is going to state what it requires.  This requires us to revisit the code that we already have.

The core of our application is exposed via the GreetingMaker interface.  It offers a method that accepts two strings as parameters and returns an Optional ```Greeting``` object as the return.  We want this ```Greeting``` class to be part of our domain so we create a new class in the ```domain``` package.

The class should look something like this:

```java
@Data
@AllArgsConstructor
public class Greeting {
  Long id;
  private String name;
  private String message;
  private LocalDateTime timestamp;

  public Greeting(String aName, String aMessage) {
    this(null, aName, aMessage, LocalDateTime.now());
  }
}
```

Negative
:This will cause other parts of the code to break because you probably have a ```Greeting``` class in another package (e.g. 'web').  Don't worry, we're going to fix that in a minute.

Therefore, our ```GreetingMaker``` interface (which is part of the domain) is returning a class from within the domain.

Equally, we want to express the requirement on the repository in terms of a domain class.  So, let's define that interface and call it ```GreetingAuditor```.

The interface should be in the ```domain``` package and look like this:

```java
public interface GreetingAuditor {
  public void saveGreeting(Greeting aGreeting);
}
```

It contains one method and we can see that its purpose is to save a domain ```Greeting``` object.  Note: we don't know how the object will be saved.  The interface reveals nothing about the type of database being used.

### Add the implementation

Now we need a class that implements the interface.  At this point, we are going to need JDBC (Java DataBase Connectivity) to access the database.

* Create a class called "GreetingAuditorJDBC" and add this code:

```Java
//...add your package here remember
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.stereotype.Repository;
import uk.ac.cf.cm6213.springbootexamples.domain.Greeting;
import uk.ac.cf.cm6213.springbootexamples.domain.GreetingAuditor;

import java.sql.*;
import java.util.List;
import java.util.Map;

@Repository
public class GreetingAuditorJDBC implements GreetingAuditor {

  @Autowired
  private JdbcTemplate jdbc;

  public void saveGreeting2(Greeting aGreeting) {

    GeneratedKeyHolder holder = new GeneratedKeyHolder();

    jdbc.update(
            new PreparedStatementCreator() {
              @Override
              public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
                PreparedStatement ps =
                        connection.prepareStatement("insert into greeting(name, message, timestamp) values(?,?,?)", Statement.RETURN_GENERATED_KEYS);

                ps.setString(1, aGreeting.getName());
                ps.setString(2, aGreeting.getMessage());
                ps.setTimestamp(3, Timestamp.valueOf(aGreeting.getTimestamp()));
                return ps;
              }
            },
            holder);

    aGreeting.setId(holder.getKey().longValue());
    System.out.println(aGreeting);

  }


  @Override
  public void saveGreeting(Greeting aGreeting) {

    Map<String, Object> greetingMap = Map.of("name", aGreeting.getName(), "message", aGreeting.getMessage(), "timestamp", aGreeting.getTimestamp());

    SimpleJdbcInsert simpleJdbcInsert = new SimpleJdbcInsert(jdbc)
            .withTableName("greeting")
            .usingGeneratedKeyColumns("id");
    Long key = simpleJdbcInsert.executeAndReturnKey(greetingMap).longValue();
    aGreeting.setId(key);
    System.out.println(aGreeting);

  }
  //...
}
```

In this example, there are two implementations of the same method.  I've just renamed one to ```saveGreeting2``` to avoid a clash.

Let's go over some of the key points.

#### The annotation

The class is annotated with the ```@Repository``` interface.  This identifies it as a Spring component that can be loaded into the container and also tells Spring that this is a component that might be a candidate for loading when we do DB testing.

#### The JdbcTemplate

We need to use JDBC to access the database.  JDBC code is typically very similar every time you use it and there are common rules to follow such as making sure that the connection is closed each time.  Spring provides the ```JdbcTemplate``` interface (and an implementation) that seeks to reduce the amount of repeated code.  It allows the application developer to focus on:

* The SQL to send to the database
* The parameters that may need to be passed into the SQL query
* Mapping the rows into objects for return to the calling code

We add the JdbcTemplate to the repository by using the ```@Autowire``` annotation.

```Java
//...
public class GreetingAuditorJDBC implements GreetingAuditor {

  @Autowired
  private JdbcTemplate jdbc;
  //...
```

As you might expect now, Spring will provide the implementation of the template for us.  It is a standard component that Spring knows it will have to provide as soon as put the JDBC dependency in the ```build.gradle``` file.

We can now use the ```jdbc``` variable in our methods.

#### Using the templates

Let's look at the shorter method which uses the ```SimpleJdbcInsert``` class.

```Java
//...
@Override
  public void saveGreeting(Greeting aGreeting) {

    Map<String, Object> greetingMap = Map.of("name", aGreeting.getName(), "message", aGreeting.getMessage(), "timestamp", aGreeting.getTimestamp());

    SimpleJdbcInsert simpleJdbcInsert = new SimpleJdbcInsert(jdbc)
            .withTableName("greeting")
            .usingGeneratedKeyColumns("id");
    Long key = simpleJdbcInsert.executeAndReturnKey(greetingMap).longValue();
    aGreeting.setId(key);
    System.out.println(aGreeting);
  }
  //...
```

Let's examine this code line-by-line.

```Java
//...
    Map<String, Object> greetingMap = Map.of("name", aGreeting.getName(), "message", aGreeting.getMessage(), "timestamp", aGreeting.getTimestamp());
  //...
```

The ```SimpleJdbcInsert``` class requires the data to be saved to be supplied in the form of a ```Map```.  The key of each entry in the map should match a column name in the table and the value should be the value to be saved.  Note: we don't need to set the value of the ```id``` column because we set up the database to set this automatically on insert.  So here, we use the ```Map.of``` method to create a map of the data that we wish to insert.

```Java
//...
    SimpleJdbcInsert simpleJdbcInsert = new SimpleJdbcInsert(jdbc)
            .withTableName("greeting")
            .usingGeneratedKeyColumns("id");
  //...
```

Next, we create a ```SimpleJdbcInsert``` object by constructing it and passing our ```JdbcTemplate``` to it.  The ```SimpleJdbcInsert``` object wraps the template and uses it to call the database.

The ```.withTableName``` method allows us to say which table we want to insert into and ```usingGeneratedKeyColumns``` allows us to say what the primary key columns are.

```Java
//...
    Long key = simpleJdbcInsert.executeAndReturnKey(greetingMap).longValue();
    aGreeting.setId(key);
    System.out.println(aGreeting);
  }
  //...
```

These last lines show us how to get the generated key back from the call to the database.  ```executeAndReturnKey``` passes in the ```Map``` of our object and returns the value of the new key (i.e. the value given to the ```id``` column in the new row).

We can then set the id of the Greeting object and print it to the console.

We'll come back to the other implementation, but first let's call this method from another service so that we can see data going into the database.

## Calling the Repository

As is the way with Spring, we will inject the repository into another component.

### Add the GreetingAuditor dependency

Adjust the code (you should be able to work out how to do this)...

```Java
@Component
public class DefaultGreetingMaker implements GreetingMaker {

  private GreetingAuditor greetingAuditor;

  @Autowired
  public DefaultGreetingMaker(GreetingAuditor aGreetingAuditor) {
    greetingAuditor = aGreetingAuditor;
  }
```

* Spring will inject the repository component that implements the required interface.

### Use the GreetingAuditor interface

Positive
:Examine the code on Gitlab before proceeding

There are some other subtle changes in the code.  The ```Greeting``` class has been moved into the domain package and the ```GreetingMaker``` and ```GreetingAuditor``` use this class.

This has been done to make the interfaces cohesive.  Those interfaces offered or required by the domain should be stated in terms of domain objects.

This will have knock-on effects in terms of change, but will make our architecture cleaner.  The components in the ```web``` package now have to make or receive domain objects doing whatever transformations they need to do, but the ```domain``` package has no dependency on the ```web``` package which is what we want to achieve.

#### Add a method to call the interface

Add this method...

```Java
private void auditGreeting(Greeting aGreeting) {

  uk.ac.cf.cm6213.springbootexamples.domain.Greeting domainGreeting = new uk.ac.cf.cm6213.springbootexamples.domain.Greeting(aGreeting.getName(), aGreeting.getMessage());

  greetingAuditor.saveGreeting(domainGreeting);
}
```

#### Adjust the ```determineGreetingWithTime``` method to call this method...

The new line is highlighted with a comment below.

```java
Optional<Greeting> returnGreeting = Optional.of(new Greeting(name, timedMessage));

   if (returnGreeting.isPresent()) {
     auditGreeting(returnGreeting.get()); //new line
   }

   return returnGreeting;
```

## Recap

We now have an end to end application.

The controller requires a service (```GreetingMaker```) which is implemented by the ```DefaultGreetingMaker``` and in turn, this service requires a repository (```GreetingAuditor```) which is implemented by the ```GreetingAuditorJDBC``` class.

Both the service and the repository classes are annotated with ```@Service``` and ```@Repository``` respectively.  These annotations tell Spring to treat these classes as components and make them candidates for injection.

Using reflection, Spring can determine which constructors or fields have ```@Autowired``` requirements and which interfaces are required.  It can then look to see which components implement the required interfaces.  Spring will, by default create one single instance of each component, and can then pass that instance into the constructors as parameters.  This enables it to 'wire' an application together from components.

## Run the application

Run the application.

* Go to [http://localhost:8080/greeting-form](http://localhost:8080/greeting-form).

* Enter a name, pick a greeting type and click 'Submit'.

* In a separate tab, open the [h2-console](http://localhost:8080/h2-console).

* Login (remember the credentials are in ```application.properties```)

* Click on ```GREETING``` in the left-hand pane.
* You will see ```SELECT * FROM GREETING ``` appear in the SQL pane.
* Click on the ```Run``` tab in the SQL pane.
* You should see a row appear in the table.
* Repeat with different tests.  Only successful greetings will be audited.  
 * You don't have to use the form.  Try the direct link...e.g [http://localhost:9090/greet/formal?name=Clare](http://localhost:9090/greet/formal?name=Clare)

Positive
: Notice that the ```id``` is automatically generated for us as we asked the database to generate this field for us.  Different databases do this in their own way.  For example, Oracle requires you to set up sequence objects in the database.

At some point, we will almost certainly need to access the primary key that has just been generated.  For example, if were saving a graph of related objects, we were need to form the joins between the tables that are storing our objects.  We would need to get the newly-generated primary key from one object so that we can set it as the foreign key on the other.

Some developers welcome this level of control over the SQL that is sent to the database, but it can have the effect that the objects become nothing more than data records and the full object-oriented model (where state and behaviour are combined to provide full abstractions) is not fully utilised.

This is known as the 'object-relational mismatch' and there are a number of different views and solutions to this.

Spring offers at least two alternatives; Spring Data JPA and Spring Data JDBC.

* Spring Data JPA - JPA stands for Java Persistence Architecture and provides an 'object-relational mapping (ORM)' solution where the developer can avoid writing the SQL themselves.  The idea is that by taking the responsibility for SQL writing away from the developer, that they will be encouraged to follow better OO principles, be willing to use OO modelling techniques such as inheritance and, as a by product, get database portability because the framework will handle the differences for you.
* Spring Data JDBC - this is a relatively new approach and follow a design approach known as Domain-driven Design (DDD).  In DDD, the problem domain is split into a number of distinct domains (e.g. Customer order handling, dispatch, warehousing, etc) and each sub-domain is modelled separately.  DDD is beyond the scope of this module, but it is a highly recommended technique as it dovetails nicely with microservices and can enable developers and architects to split a large domain into areas that 'fit in your head'.

## Our current architecture

<div hidden>
```
@startuml assets/firstDiagram

class GreetingController
class DefaultGreetingMaker
class GreetingAuditorJDBC
interface GreetingMaker
interface GreetingAuditor

GreetingMaker <|--  DefaultGreetingMaker
note on link : Core service offers an interface
GreetingAuditor <|--  GreetingAuditorJDBC

GreetingController --> GreetingMaker

DefaultGreetingMaker --> GreetingAuditor
note on link : core service requires an interface

@enduml
```
</div>

![](/assets/firstDiagram.png)

### Balance

Good design will allow us to maintain productivity over time.  However, a principle of agile development is YAGNI (You ain't gonna need it) - implying that we shouldn't design for things that may never happen.

Therefore, if we know that we'll never need to change databases, or we'll never need to support a different UI should we go to these lengths?

A core principle of design is to separate the solution into modules that are cohesive and have a low coupling to other components.  That is, they have a clear responsibility within the application.  If you apply this principle and want clear boundaries between components and responsibilities, you will tend towards interfaces and representations that are contextual (that is, they make sense within the scope of the component or module).

## Reading data from the Database

Now that we have inserted data into the database, we want to try and get it back out.  We'll work from the bottom up, adding the functionality to the repository, and finally exposing the data via a controller.

### Write the SQL and map to objects

Add this method to ```GreetingAuditorJDBC``` (you will need to add the method signature to the interface as well)

```Java
public List<Greeting> allGreetingsByMostRecent() {
  return jdbc.query(
      "select * from greeting order by timestamp desc",
      new Object[]{},
      (rs, i) -> new Greeting(rs.getLong("id"),
          rs.getString("name"),
          rs.getString("message"),
          rs.getTimestamp("timestamp").toLocalDateTime()
      )
 );
}
```

Here we are using the JDBC template to send a SQL query to the database, and map the results into a List of  ```Greeting``` objects.

You should note that using the JDBC template removes any concerns about connection handling.  You (as the developer) can focus on the interaction with the database (which is specific to the application in hand).

In this case, the SQL is a select against the ```GREETING``` table to get all rows ordered by the timestamp (so that see them in reverse order).

The query has no parameters so we don't need to pass any parameters.  We'll do that soon, but for now, the 2nd parameter to the ```query``` method is ```new Object[]{}``` (an empty array of objects).

In normal JDBC, the query will return a ```ResultSet``` and the developer needs to iterate through that.  The template knows this, and so merely needs a function to map each row in the ResultSet into an object.  The template will collect the mapped objects together and return as a list.

Therefore the 3rd parameter is a function that receives the current row ```ResultSet``` (rs) and current index (i) and returns a new ```Greeting``` object by passing the fields in the row to the constructor.  Note: we need to transform the timestamp between the database's type and the Java type.

### Call the method from the service

Add a method to the service to call the new method in the repository.  Again, you will need to add the signature to the appropriate interface.

```Java
public List<Greeting> allGreetingsByMostRecent() {
    return greetingAuditor.allGreetingsByMostRecent();
  }
```

### Call the service method from the controllers

Now we call the service method from the controller and put the list of ```Greeting``` objects on the model.

```Java
@GetMapping("all-greetings")
  public String allGreetings(Model model) {

    //should check for the empty list scenario, but will it out for brevity

    model.addAttribute("greetings", theGreetingMaker.allGreetingsByMostRecent());
    return "greeting-list";
  }
```

### Create a template to show the allGreetings

We're passing the model to the "greeting-list" template, so we need to write that (in /resources/templates/greeting-list.html).

Showing the body only for brevity...

Positive
: This also shows how to render a table in Thymeleaf.  The ```th:each``` attribute is on the ```<tr>``` element as it is the row that will be iterated.  The ```<td>``` tags show the data for each greeting.

```html
<body>
<h1>Greeting List</h1>

<table>
    <thead>
    <tr>
        <td>Id</td>
        <td>Name</td>
        <td>Message</td>
        <td>Time</td>
    </tr>
    </thead>

    <tbody>
    <tr th:each="greeting : ${greetings}">
        <td th:text="${greeting.id}"></td>
        <td th:text="${greeting.name}"></td>
        <td th:text="${greeting.message}"></td>
        <td th:text="${greeting.timestamp}"></td>
    </tr>
    </tbody>
</table>
</body>
```

### Run the application

* Enter some data as before using the form or the direct URLs.
* Browse to [http://localhost:8080/all-greetings](http://localhost:8080/all-greetings)

You should see all of the greetings that you've entered in reverse order.

## Revisit the alterative way to insert data

In the ```GreetingAuditorJDBC``` class, there is an alternative method of inserting data.

```Java
public void saveGreeting2(Greeting aGreeting) {

    GeneratedKeyHolder holder = new GeneratedKeyHolder();

    jdbc.update(
            new PreparedStatementCreator() {
              @Override
              public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
                PreparedStatement ps =
                        connection.prepareStatement("insert into greeting(name, message, timestamp) values(?,?,?)", Statement.RETURN_GENERATED_KEYS);

                ps.setString(1, aGreeting.getName());
                ps.setString(2, aGreeting.getMessage());
                ps.setTimestamp(3, Timestamp.valueOf(aGreeting.getTimestamp()));
                return ps;
              }
            },
            holder);

    aGreeting.setId(holder.getKey().longValue());
```

This method is more verbose, but is more consistent with the approach used to select data as the SQL query and the mapping of parameters is more explicit.

In JDBC, we should prefer to use PreparedStatements as they help us to avoid SQL injection attacks.  So in this example, we create a ```PreparedStatement``` that takes the required fields as parameters (indicated by '?'s).  We then map by index each of our object's fields into the query. For example, the 1st '?' should get the name field, so we set the 1st parameter to the name on the greeting object.

The wrapper code is a bit verbose, but once done once, it can be copied and pasted as a recurring snippet.

The ```SimpleJDBCInsert``` method is a way of avoiding this repeated code.  

It should be noted that there is no equicalent for "SimpleJdbcUpdate".  A web search will show that this has been looked at, but that development was stopped in 2019.

Therefore, the ```PreparedStatement``` approach is required for updates and deletes.

## Extensions

There are a number of extra activities that you can try with this.

* Add a feature where the list of greetings can be filtered by name

* Add the ability to store the type of greeting.

* Add a feature whereby a specific greeting can be toggled between informal and formal.  Update the message and the timestamp accordingly.

* Add a feature to delete a specific greeting.  Provide the id as a parameter.  e.g. "delete greeting 5"

* (More complicated) Add a feature that puts the greeting types into their own table and stores the message pattern alongside it.  For example, the table would contain "formal","hello {name} it's {time}" as a row.  The application would need to check that the URL submitted matches a greeting type in the database.  The service will then need to retrieve the message format from the database and use it to create the message.
