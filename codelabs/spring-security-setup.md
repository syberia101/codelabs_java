summary: Enable Spring Security in an existing project
id: spring-security-setup
categories: cyber-security
tags: spring-security, setup
status: Published
authors: Phil Smart


# Spring Security Basic Setup
Duration: 0:20:00

<!-- ------------------------ -->
## Overview
Duration: 1

### What You’ll Learn

- How to add Spring Security to an existing Spring Boot project
- How to add the Thymeleaf Spring Security extras to an existing Spring Boot Project


<!-- ------------------------ -->
## Enabling Spring Security
Duration: 10

Positive
: Reference: https://spring.io/guides/gs/securing-web/

- Add the following Spring Security starter dependency to your `build.gradle` file

```xml
dependencies {
	...
	implementation 'org.springframework.boot:spring-boot-starter-security'
}
```

- Restart your application. You will see ALL HTTP endpoints are protected with a basic login page by default.
- The default username is `user` and the default password is printed to the console e.g.

```java
2020-11-18 21:26:09.144  INFO 33929 --- [main] .s.s.UserDetailsServiceAutoConfiguration :  Using generated security password: 9d3593b1-ce46-4de5-895f-abce86bb878c
```

<!-- ------------------------ -->
## Add the Thymeleaf Spring Security Dialect
Duration: 5

Positive
: Reference: https://github.com/thymeleaf/thymeleaf-extras-springsecurity

- Integrate Thymeleaf and Spring Security such that you can use security related expressions in Thymeleaf templates.
 - Add the following dependency to your `build.gradle` file:

```xml
dependencies {
	...
  compile group: 'org.thymeleaf.extras', name: 'thymeleaf-extras-springsecurity5'
}
```

<!-- ------------------------ -->
## What's next
Duration: 2

- Configure Spring Security
- Use Spring Security related expressions in Thymeleaf
