summary: Spring Security Test User Setup and Simple Authorisation
id: spring-security-users-authz
categories: cyber-security
tags: spring-security
status: Published
authors: Phil Smart


# Spring Security Test User Setup and Simple Authorisation
Duration: 0:20:00

<!-- ------------------------ -->
## Overview
Duration: 1

### What You’ll Learn

- How to add two in-memory users for testing
- How to restrict pages (URLs) to certain user roles

<!-- ------------------------ -->
## How to add two in-memory users for testing
Duration: 10

Positive
: https://docs.spring.io/spring-security/site/docs/current/reference/html5/#hello-web-security-java-configuration

- In the security configuration class you created in the `spring-security-basic-config` tutorial, add a `UserDetailsService` bean that configures an `InMemoryUserDetailsManager` (technically this bean could exist in any `@Configuration` class).
- Configure two users for testing; an `ADMIN` user, and a standard `USER` user.

``` java
	 @Bean
    public UserDetailsService userDetailsService() {
            InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
            manager.createUser(User.
                       withDefaultPasswordEncoder()
                       .username("user")
                       .password("password")
                       .roles("USER")
                       .build());
            manager.createUser(User.
                       withDefaultPasswordEncoder()
                       .username("admin")
                       .password("password")
                       .roles("ADMIN")
                       .build());
            return manager;
    }
```
- You can now login with either user (change the password if you want).

<!-- ------------------------ -->
## How to restrict pages (URLs) to certain user roles
Duration: 10

Positive
: https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-authorization-filtersecurityinterceptor

- Lets leave the `dashboard` page so it is accessible to **any** authenticated user.
- Lets ensure anybody trying to access anything inside the `/admin/` path requires the `ADMIN` role.
- Lets ensure anybody trying to access anything inside the `/user/` path requires either the `USER` or `ADMIN` role.

``` java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .authorizeRequests(authorizeRequests ->
            authorizeRequests
                .mvcMatchers("/dashboard").authenticated()
                .mvcMatchers("/user/**").hasAnyRole("USER","ADMIN")
                .mvcMatchers("/admin/**").hasRole("ADMIN")
                .mvcMatchers("/styles/**").permitAll()
                .mvcMatchers("/signup").permitAll()
                .anyRequest().denyAll()
        )
        .formLogin(formLogin ->
            formLogin
                .permitAll()
        ).logout(logout ->
             logout
                .permitAll());

    }
```
