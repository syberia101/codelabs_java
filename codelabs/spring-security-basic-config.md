summary: Spring Security Basic Configuration
id: spring-security-basic-config
categories: cyber-security
tags: spring-security
status: Published
authors: Phil Smart


# Spring Security Basic Configuration
Duration: 0:20:00

<!-- ------------------------ -->
## Overview
Duration: 1

### What You’ll Learn

- How to add a simple configuration to Spring Security.
- How to protect a page with a form login.
- How to protect a range of pages with a form login.


<!-- ------------------------ -->
## Where to add a simple Spring Security configuration
Duration: 20

Positive
: https://spring.io/guides/gs/securing-web/

- The `WebSecurityConfigurer` interface allows customisation of an applications `WebSecurity`.
- Create a class that extends `WebSecurityConfigurerAdapter` and is annotated with both `@Configuration` and `@EnableWebSecurity`.

``` java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		 //put most of your configuration here.
	}
}
```

<!-- ------------------------ -->
## Protect a single page with the default Spring login page
Duration: 20

Positive
: https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-authorization-filtersecurityinterceptor

- Without configuration, by default, all pages (or technically all URLs) will require authentication.
- Once you start to explicitly configure the `HttpSecurity` object inside your own class that is annotated with `@EnabledWebSecurity`, that (and any other) default disappears.
- Lets protect a page called `dashboard` using the standard spring security login page.

``` java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .authorizeRequests(authorizeRequests ->
            authorizeRequests
                  .mvcMatchers("/dashboard").authenticated()
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

- **Notice**, the login page and logout page do **not** require authentication.
- **Notice**, we have added a final configuration item to **deny** any request which does not match the patterns defined. This is good a security posture.

Positive:
You may notice the above syntax is the newer DSL style. Both that and the old style are valid, see https://spring.io/blog/2019/11/21/spring-security-lambda-dsl to understand the difference.

<!-- ------------------------ -->
## Protect a range of pages with the default Spring login page
Duration: 20

Positive
: https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-authorization-filtersecurityinterceptor

- Extending the basic config from before, we can now secure all pages in the `/users/` and `/admin/` paths, but allow anybody to access the `/signup` page.

``` java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .authorizeRequests(authorizeRequests ->
            authorizeRequests
                .mvcMatchers("/dashboard").authenticated()
                .mvcMatchers("/users/**").authenticated()
                .mvcMatchers("/admin/**").authenticated()
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

- Therefore `/dashboard` and anything inside the paths `/users/*` and `/admin/*` requires authentication, but `/signup` does not.
- **NOTICE**, we have also added `/styles/**` to the permitAll list. That is, anything inside your `src/main/resources/static/styles` folder will be accessible e.g. your CSS files. You would need similar for your JavaScript files etc.
 - This is useful if you need your style sheets etc. to be usable on pages that do not require authentication e.g. the `signup` page and the `login` page.
