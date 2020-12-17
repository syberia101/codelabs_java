summary: Introduction to JPA (Java Persistence Architecture)
id: intro-to-jpa
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Introduction to Java Persistence Architecture (Spring Data JPA)
<!-- ------------------------ -->
## Overview
Duration: 30

### What You'll Learn
- How to use JPA to manage the mapping of objects to a relational database.  
- How to annotate a class for JPA
- How to specify the primary key generation strategy
- How to start modelling relationships
- How to adapt existing interfaces to use jpa

<!-- ------------------------ -->

## Understand the goal

With JDBC, we have to write the SQL ourselves and we need to write code to convert the returned data into objects.  We have to manage the primary and foreign keys and the object graphs ourselves.  This provides a deep level of control over the database interaction and enables us to make full use of the facilities of the database, tuning queries and data sizes to the needs of the request.

However, there is another point of view that would say that there are a number of common patterns in the generation of the SQL and that those patterns can be abstracted into a framework that can generate the SQL, handling key generation, joins and offer concepts such as eager vs lazy loading (lazy => only getting data from the database when required).

This latter concern has been addressed for many years.  The dominate ORM (object-relational mapping) framework in Java is [Hibernate](https://www.hibernate.org/orm) which is used by Spring Data JPA (Java Persistence Architecture).

In this tutorial, we are going to introduce Spring Data JPA.  It is only intended to provide an overview.  JPA is a large framework with many features and options and is well beyond the scope of a single tutorial.

## Basics

In Spring Data JPA, we need to do the following:

* Annotate our domain classes so that they are mapped to tables (these domain classes then become known as 'Entity' classes)
* Annotate the fields in our domain classes so that they map to columns in tables
* Annotate certain fields in our domain classes so that we know which relate to a primary keys
* Annotate certain fields in our domain classes so that we know which relate to object-to-object relationships and how they relate to foreign keys
* Create Repository components that allow to persist Entity objects
* Write methods on our Repository components that define queries

Positive
: In the accompanying video, we show how to adapt the calls to the new JPA repositories so that we can leave our existing controllers and services untouched.  This won't be covered in this tutorial.

## Adding the JPA dependency

As always, we need to add the Spring Data JPA dependency to our project.  We do this by adding this line to ```build.gradle``` in the ```dependencies``` block.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

## Converting a domain class to an Entity

To delegate persistence to JPA, we need to convert our domain classes into entities.

Positive
: We could keep our domain classes as POJOs (plain old Java objects) if we wished.  We would then need to marshall the data between our domain objects and our entities.

### Add the @Entity annotation

We need to identify the class as an entity, so we add the ```@Entity``` annotation to the class.

```Java
@NoArgsConstructor
@Entity
public class Greeting {
//...
```

We do the same for the GreetingType class.
```Java
@NoArgsConstructor
@Entity
public class GreetingType {
//..

```

The import package for JPA is ```javax.persistence.* ```.

Positive
: We also have to include a no-args constructor on any entity class.  This is related to proxies which will be explained later.  Luckily, Lombok provides a convenient annotation for this.

### Identify the primary key

To tell JPA which field in the class relates to the primary key in the table, we need to apply the ```@Id``` annotation.

For Greeting:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
Long id;
```

In the ```Greeting``` case, the primary key is a new ```Long``` field (it must be an object type) and we want the database to create the primary key when we save a new object to the database.  JPA allows us to specify a number of different generation strategies.  Here we are choosing one that is compatible with our database's ```AUTO-INCREMENT``` setting.

For GreetingType:

```java

@Id
 private String type;

```

In the ```GreetingType``` class, we have a natural primary key and we manage new rows in this table via direct SQL, so we don't specify a generation strategy.

### Naming standards

We are now in a position where we could save and retrieve objects from tables.  Spring Data JPA applies convention-over-configuration again and assumes that a Greeting class object will be saved in a 'greeting' table and that a GreetingType class object will be saved into a 'greeting_type'.  That is, it applies a naming strategy to convert standard camel case Java code into standard underscored table names.

It applies the same logic to fields and columns.

If the mappings differ, JPA supplies a ```@Table``` annotation (at class level) and a ```@Column``` annotation (at field level).

For more info, see [this](https://www.baeldung.com/hibernate-field-naming-spring-boot).

### Annotating relationships

Every Greeting object is linked to a GreetingType object, and each GreetingType could be related to many (lots) of Greeting objects.

In this example, we want to be able to ask every Greeting object for its GreetingType, but we don't want to be able to ask every GreetingType object for Greetings of that type (as that could return a big list).

In essence, we have a one-to-many relationship between GreetingType and Greeting, which implies a many-to-one in the opposite direction.

The annotations are (in Greeting.java):

```Java

@ManyToOne
@JoinColumn(name = "type")
private GreetingType type;

```


Let's walk through these annotations.

The annotation ```@ManyToOne``` is self explanatory.  We are in the "many" side of the relationship and we're pointing at the "one" side.


The annotation ``` @JoinColumn``` tells us the column name in the table that will be used to handle the relationship in the database.  In the ```greeting``` table, we have a column "type" of type "VARCHAR".  We will put the foreign key in this column.  

However, when we retrieve the data, we will want to get a ```Greeting``` object and be able to call ```getType()``` to get a ```GreetingType``` object back.

### What SQL is generated

You may already be thinking what SQL is generated.  There are two options (at least).

When we get a ```Greeting``` object (or set of such), we will select from the ```greeting``` table.  This will give us the foreign key to ```GreetingType```.  When ```getType()``` is called, we would need to generate another select on the ```greeting_type``` table to get the other fields and populate an object.  This is known as lazy loading because we only pull data from the database when it is required.

Alternatively, we could join the table together, so that every row contains all the required fields to create all objects.  This pulls back more data (maybe some is not required), but reduces the number of queries made.  This is known as eager loading.

Positive
: Some of you may be wondering how the 2nd SQL query gets generated because our ```Greeting``` class doesn't contain any SQL-generating code.  The truth is that you don't get back instances of your ```Greeting``` class.  Rather you get back a *proxy* to your class.  This proxy is generated by JPA and it acts as an intermediary.  It sits between the calling code and your POJO and intercepts the call, making the SQL call as required.

## The JPA Repository

Making a JPA repository component requires you to extend a provided generic interface.

Create an interface called ```GreetingRepoJPA```.  

Import ```org.springframework.data.jpa.repository.JpaRepository```.

Your interface can then be made to extend ```JpaRepository```.

```Java

public interface GreetingRepoJPA extends JpaRepository<Greeting, Long> {
//..
}
```

This is an interface extending an interface.  The ```JpaRepository``` is provided with two types - the type of entity that the repository is managing (in this case, 'Greeting') and the type of the key of that entity.

: Yes, it does seem odd that the type of the key needs to be provided, when Spring does everything else by reflection!

### Standard methods

At this point, there are already a number of methods available.  You could inject this component (no need to annotate with ```@Repository```) into another component and see the methods.  (Try it and let IntelliJ show you the available methods)

## Adding methods

We now want to add methods to the interface that meet our needs.  Firstly, let's add a method to get a list of ```Greeting``` objects by name.

In our new interface source file, add the following method declaration.

```java
List<Greeting> findByName(String aName);
```

Believe it or not, you've just implemented a method that will generate the SQL required to get the right rows from the database and construct the required objects.

Spring Data JPA can parse the method name.  Each part instructs the framework to generate the required code.

'find' => a select is required
'By' => a where will be required
'Name' => implies the column 'name' will be in the "where" clause.

Let's add a more complicated method.  Let's get the 5 most recent greetings.

```java
List<Greeting> findTop5ByOrderByTimestampDesc();
```

'find' - as above
'top' => a 'LIMIT' (or similar) will be required on the SQL
'5' => the number to limit to
'By' => since it's followed by 'Order' it means no "where" clause
'OrderBy' => implies we'll have an order by on the SQL
'Timestamp' => implies the field to sort by
'Desc' => implies the direction of the sort


We now have a functioning repository for retrieving ```Greeting``` objects.  Let's add the same for ```GreetingType``` objects.

This is all we need:

```java
public interface GreetingTypeRepoJPA extends JpaRepository<GreetingType, String> {
}
```

This will give us a ```findById()``` method and that's all we need.

## Using the new JPA repository

In the [Gitlab project](https://git.cardiff.ac.uk/ase-2020/cm6213/springbootexamples/-/tree/convert-to-jpa), we show how to write adaptors that enable the service components to work unchanged.  Here we'll highlight some of the code.

In our adaptor, we inject the new repository and refer to it with field ```greetingRepoJPA```.

We can then write the following methods:

```java
public void saveGreeting(Greeting aGreeting) {
   greetingRepoJPA.save(aGreeting);
 }

 @Override
 public void updateGreeting(Greeting aGreeting) {
   greetingRepoJPA.save(aGreeting);
 }
```

Things to note include:

The ```saveGreeting``` and ```updateGreeting``` methods are exactly as before (with JDBC).  Now, we delegate the call to the ```save()``` method on the new repository.  We didn't write this ```save``` method.  We didn't even specify it, but it's there for free because we extended ```JpaRepository```.

Also, note that both methods use ```save()```.  JPA can work out if the object is new (and therefore needs an 'insert') or existing (and therefore needs an 'update').

## Summary

This has been a fly-past of JPA.  It can do so much more and you can control the SQL so much more.
