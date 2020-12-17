summary: Spring Security Content Security Policy
id: spring-security-csp
categories: cyber-security
tags: spring-security
status: Published
authors: Phil Smart


# Spring Security Content Security Policy
Duration: 0:60:00

<!-- ------------------------ -->
## Overview
Duration: 1

### What You’ll Learn

- How to add a simple Content Security Policy using Spring Security
- Where do I add my resources

<!-- ------------------------ -->
## How to add a Content Security Policy
Duration: 20

Positive
: https://docs.spring.io/spring-security/site/docs/current/reference/html5/#headers-csp and https://content-security-policy.com/

- Helps to prevent Cross-site Scripting attacks - or other forms of content injection.
- Is not a guaranteed defence. For example, almost by design you may have to whitelist a resource which later becomes untrustworthy and injects a malicious script onto your site.
- It is not easy to establish a good policy (there is no requirement for it to be watertight for your project).
- This can be achieved by setting the `ContentSecurityPolicy` header in spring security e.g. extending the example from the previous spring security tutorials.

``` Java
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
                .permitAll())
        .headers().contentSecurityPolicy(csp ->
            csp.policyDirectives("default-src 'self'; object-src 'self'")        
        );       
    }
```

- This simple (and certainly not a good) policy will prevent any resources (JavaScript, StyleSheets, Images etc) loading from any location other than those hosted on the site's own origin (see here for help on same-origin and same-site https://web.dev/same-site-same-origin/).
 - This is unlikely to work well for your site e.g. assuming you take resources from other origins, for instance; bootstrap, fontawesome etc.
  - You will need to relax this basic constraint to allow for trusted sources.
 - You can test your CSP on sites like: https://csp-evaluator.withgoogle.com/

 <!-- ------------------------ -->
## How do I get my inline JavaScript working again
  Duration: 3

- Problem: I have inline (embedded in the HTML) JavaScript that I want to run, but the basic CSP policy is preventing inline scripts from running. What do I do?
- You have a few options (described on the following pages):
 - Allow unsafe inline script execution.
 - Externalise your JavaScript.
 - Whitelist the hash of your inline JavaScript.

 <!-- ------------------------ -->
## Allow unsafe inline script execution
 Duration: 3

 - Allow unsafe inline script execution e.g. ` csp.policyDirectives("default-src 'self'; script-src 'unsafe-inline'; object-src 'self'") `. However, **do not do this** as you are allowing possibly malicious JavaScript to run if it gets reflected back onto the page.

 <!-- ------------------------ -->
## Externalise your JavaScript
 Duration: 15

  - Externalise your JavaScript into a `.js` file served from, for example, your static resource folder. This is then loaded using the <script type="text/javascript" th:src="@{/js/test.js}"></script> tag. As this will be located on the same origin (so origin is `'self'`) the browser will load, trust, and execute it.
  - **Note**, this will disable inline event handlers as well, for instance anything in `onclick=""` etc. These need to be replaced with `addEventListener()` calls. For example:

 ``` JavaScript
  //In your HTML
  <script>
      function alertMe(){
      	window.alert("hi");
      }
  </script>
  <div onclick="alertMe()"/>
 ```

  Should be replaced by:

 ``` JavaScript

  //In your HTML
 <div id="clickable-div"/>

 //In your external js file
 function alertMe(){
         window.alert("hi");
 }
 document.addEventListener('DOMContentLoaded', function () {
   document.getElementById('clickable-div')
     .addEventListener('click', alertMe);
 });

 ```

<!-- ------------------------ -->
## Whitelist the hash of your inline JavaScript
Duration: 20

 - If you must use inline JavaScript, you can whitelist the hash of the script you want to run.
 - Firstly, take the script you want to run, remove the script tags, and canonicalise the text (remove all the formatting e.g. whitespaces etc.). You do not have to do this, but if you do not then the formatting becomes part of the hash, and hence any changes to the format (e.g. you introduce a space) will require the generation of a different hash.
  - **Note**, you can not whitelist the inline event handler e.g. the `onclick`, so you need to include the event listener in your inline script as well.
 - For example:

 ``` JavaScript
   //In the HTML
   <script>
   		function alertMe(){
   		 window.alert("hi");
   		}
   		document.addEventListener('DOMContentLoaded', function () {
     document.getElementById('game-container')
       .addEventListener('click', alertMeExternally);
   	});
   </script>
 ```

 becomes the 'string to hash' of:

 ``` javascript
   function alertMe(){window.alert("hi");}document.addEventListener('DOMContentLoaded',function(){document.getElementById('game-container').addEventListener('click', alertMe);});
 ```

which you must also use in your \<script\> tag (replacing the formatted one in the first snippet):

 ``` JavaScript

<script>function alertMe(){window.alert("hi");}document.addEventListener('DOMContentLoaded',function(){document.getElementById('game-container').addEventListener('click', alertMe);});</script>

 ```

  - Next, compute the sha* hash (as base64) of the 'string to hash'. For example, a base64 encoded sha256 hash of the above is `siPmkQwTblLsYDVnOPqTPEfWGML0dXY0o+SdZF6GGB0`. A hash can be generated using various command line tools or websites e.g. https://approsto.com/sha-generator/.
  - Add this to the CSP policy as a `script-src` ; ` csp.policyDirectives("default-src 'self'; script-src 'sha256-siPmkQwTblLsYDVnOPqTPEfWGML0dXY0o+SdZF6GGB0'; object-src 'self'") `
 - Of benefit:
   - If an attacker manipulates the script, the hash will not match and the browser will not execute it.
   - If an attacker injects their own script, they have no way of adding the hash of it to the CSP headers the browser received from the server, and hence the browser will not execute it.
