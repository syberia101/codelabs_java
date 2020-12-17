summary: Configure the application to use MariaDB
id: using-mariadb
categories: Sample
tags: medium
status: Published
authors: Carl Jones
Feedback Link:

# Configure the application to use MariaDB
<!-- ------------------------ -->
## Overview
Duration: 10

### What You'll Learn
- How to change the application configuration to use MariaDB as the database

<!-- ------------------------ -->

## Understand the goal

We want to change the application to use MariaDB.  Importantly, we want to do this with only configuration.

## Add MariaDB support to the application

* Add MariaDB to ```build.gradle```

```groovy
runtime 'org.mariadb.jdbc:mariadb-java-client:2.7.0'
```


## Change the database properties to match database

* Add the database properties.  The properties are the URL to the database (including the schema name), and the credentials.  For production databases, you would want to consider other ways of managing the credentials.

```
spring.datasource.url=jdbc:mariadb://localhost:3306/greetings
spring.datasource.username=root
spring.datasource.password=comsc
```

In this example, the database is on the same machine as the Spring Boot server, listening on the default port (3306) and the schema name is 'greetings'.

## Database initialisation

The database initialisation scripts will not be run, so you need to create the database and initialise it externally.

You can alter this behaviour using properties. See [Spring Boot documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-initialize-a-database-using-spring-jdbc) for details. 
