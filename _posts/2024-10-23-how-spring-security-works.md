---
title: "Spring Security - How it Works?"
last_modified_at: 2024-10-21T16:20:02-05:00
categories:
  - Blog
  - Frameworks
tags:
  - Java 
  - Spring 
  - Spring Boot
  - Security
classes: wide
---

# Spring Security Architecture Overview

Spring Security helps with the following use cases:

1. Protect against exploits
2. Authentication
3. Authorization

# How are these functionalities implemented?

Spring Security is based on a few core objects. They are key to understanding the implementation:

- `Filter`
- `FilterChain`
- `Authentication`
- `SecurityContext`
- `AuthenticationManager`
- `AuthenticationProvider`

## Filter

The filter concept is what we know from the servlet archiecture. 
Filters operate on HTTP request and HTTP response objects and carry out important functionalites before the request reaches the controller, which processes the request and generates a response.

If a request reaches a filter, it checks if the the request meets its requirements.
In case the requirements are met, the filter does its job, then calls back the filter chain, which continues with the calling the next filter. 

If the request is not valid or not allowed from the perspective of the filter, then it manipulates the HTTP response object - e.g. sets status code and error messages - and does not call the filter chain anymore. As a result, the remaing filters in the chain are not called and the HTTP response object is sent to the client.

![Security Filter Chain](/blog/assets/images/spring/security-filter-chain.png)

## FilterChain

The filter chain contains all registered filters. It traverses the filters and calls them one after the other. Each filter carries out its functionality in case the request is valid and then calls back into the filter chain. 

As a consequence the stack trace would look like:

![Filter Chain](/blog/assets/images/spring/security-filter-chain-impl.png)


The stack trace would look like:  
![FC Stack Trace](/blog/assets/images/spring/filter-chain-stack-trace.png)

## Authentication

The `Authentication` object serves 2 use cases:

  1. It represents an authentication request
  2. The result of an authentication action

It consists of:
  - Principal: Identifies the user making the request
  - GrantedAuthorities: The permissions / roles
  - isAuthenticated(): Whether the user is authenticated or not
  - Details: Details about the request, such as IP etc.
  - Credentials: "password", often erased after successful login - set to null

## SecurityContext

The authenticated `Authentication` object is stored in the `SecurityContext`.

The stored `Authentication` can be accessed via:
```java
  SecurityContext ctx = SecurityContextHolder.getContext();
  Authentication authentication = ctx.getAuthentication();
```

The `SecurityContext` can also be injected into a controller method.

## AuthenticationManager

Converts a not authenticated `Authentication` object into an authenticated one:

![Authentication Manager](/blog/assets/images/spring/authentication-manager.png)


## AuthenticationProvider

`AuthenticationProvider` is used by `ProviderManager` to perform a specific type of authentication.

You can create a custom `AuthenticationProvider` by implementing that interface.

---
# DEMO

> Please find the source code on [GitHub](https://github.com/tokalak/spring-security-demo)

## Step 1: Create A Spring Boot Application

The easiest way to create a Spring Boot application is to use the Spring Initializer web application `https://start.spring.io`. You can use the language and language version, build tool and the dependencies: 

![Spring Initializer](/blog/assets/images/spring/spring-initializer.png)

You can also create a Spring Boot application in IntelliJ very easily. 

I'm using latest stable Spring Boot version and the dependencies Spring Security and Spring Web and Thymeleaf template engine and some testing libraries.
`build.gradle`: 

```groovy
...
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}
...
```
## Step 2: Configure Spring Security

- Create a configuration class
- Mark it as configuration class with @Configuration
- Enable Spring Security with @EnableWebSecurity
- Define a method which creates a Bean of type `SecurityFilterChain`

Using `HttpSecurity`, we configure:
- **What** to protect 
- **How** to protect
- How to login

```java
@Configuration
@EnableWebSecurity
class SecurityConfig {

  @Bean
  SecurityFilterChain securityFilterChain(final HttpSecurity httpSec) throws Exception {
    return httpSec
        .authorizeHttpRequests(
            authorizeHttp -> {
              // Recommended pattern to follow:
              //   1. List the accessible resource explicitly
              //   2. Block non-authenticated access to all other resources

              authorizeHttp.requestMatchers("/").permitAll();
              authorizeHttp.requestMatchers("/css/*").permitAll();
              authorizeHttp.requestMatchers("/error").permitAll();
              authorizeHttp.anyRequest().authenticated(); // block all other resources
            }
        )
        .formLogin(l -> l.defaultSuccessUrl("/secured"))
        .logout(l -> l.logoutSuccessUrl("/"))
        .httpBasic(withDefaults())
        .build();
  }
```
We have only two end points: 

1. `/`: a public end point
2. `/secured`: a secured end point. Accessible by authenticated users only. For this very simple demo, it will be accesible by the admin user only.

The corresponding the HTML pages for the mentioned end points:

1. `public.html`
2. `secured.html`: includes some Thymeleaf elements operating on parameters set in the controller

`secured.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>A Secured Page</title>
  <link rel="stylesheet" href="css/style.css">
</head>

<body>
<h1><em>Secured Area!</em></h1>
<p>Hello, <th:block th:text="${name}" />!</p>
<p>You are allowed to access this private page.</p>

<form method="post" action="/logout" th:if="${_csrf != null}">
  <input name="_csrf" type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}">
  <button class="btn" type="submit">Log out</button>
</form>

</body>
</html>
```

# Step 3: A Custom Filter Implementation

We'll create a very simple filter. We have the choice between extending `java.jakarta.Filter` or stay in the Spring Security realm by extending `OncePerRequestFilter`. I prefer going with `OncePerRequestFilter`. 

In this filter, I override the `doFilterInternal()` method. It allows us to access the HTTP request and response. We're in the position to carry out some tasks before the request reaches other filters in the filter chain and the controller in the end.

We can also short-circuit the request, by manipulating the HTTP response object and not calling the filter chain.

```java 
public class SloganFilter extends OncePerRequestFilter {

  @Override
  protected void doFilterInternal(
      HttpServletRequest request,
      HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {

    // You can prevent processing remaining filters in the chain by just returning:
    // return;

    if (Objects.equals(request.getHeader("x-secret-slogan"), "let_me_in")) {
      // do whatever you want here, e.g. log, enhance request data, store some information in session etc.
      System.out.println("Successfully passed with expected slogan!");
    }

    // Call back into the filter chain to continue processing the remaining filters
    filterChain.doFilter(request, response);
  }
}
```
As next, we've to register our custom filter in class `SecurityConfig`:

```java
...
SecurityFilterChain securityFilterChain(...){
  return httpSec.authorizeHttpRequests(...)
                ...
                .httpBasic(withDefaults())
                // place our filter before logout filter
                .addFilterBefore(new SloganFilter(), LogoutFilter.class)
                .build();
...
```
To verify I our custom filter is picked up by Spring Boot, you can check the logs if logging is enabled. 
After starting the application, Spring Boot logs the registered filters. Look out for `DefaultSecurityFilterChain`. You should see your filter in the list.

```bash
 o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with filters: DisableEncodeUrlFilter, WebAsyncManagerIntegrationFilter, SecurityContextHolderFilter, HeaderWriterFilter, CsrfFilter, SloganFilter, LogoutFilter, UsernamePasswordAuthenticationFilter, DefaultLoginPageGeneratingFilter, DefaultLogoutPageGeneratingFilter, BasicAuthenticationFilter, RequestCacheAwareFilter, SecurityContextHolderAwareRequestFilter, AnonymousAuthenticationFilter, ExceptionTranslationFilter, AuthorizationFilter
```
A much more elegant way is to set logging level for Spring Security to `TRACE` - don't do it on production environment!.
You should see the list of all registered filters, the invocation of them one after the other and at which position they are located in the filter list:

```bash
...
2024-10-24T10:19:27.274+03:00 DEBUG 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Securing GET /secured?continue
2024-10-24T10:19:27.274+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Invoking DisableEncodeUrlFilter (1/16)
2024-10-24T10:19:27.274+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Invoking WebAsyncManagerIntegrationFilter (2/16)
2024-10-24T10:19:27.275+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Invoking SecurityContextHolderFilter (3/16)
2024-10-24T10:19:27.275+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Invoking HeaderWriterFilter (4/16)
2024-10-24T10:19:27.275+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Invoking CsrfFilter (5/16)
2024-10-30T10:19:27.275+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.csrf.CsrfFilter         : Did not protect against CSRF since request did not match CsrfNotRequired [TRACE, HEAD, GET, OPTIONS]
2024-10-24T10:19:27.275+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Invoking SloganFilter (6/16)
2024-10-24T10:19:27.275+03:00 TRACE 75650 --- [0.1-8080-exec-5] o.s.security.web.FilterChainProxy        : Invoking LogoutFilter (7/16)
...
```
We can see that there are 16 filters in total and they are executed one after the other. Our filter is at position 6.


To see our filter in action, make a call like: 

```bash
curl localhost:8080 -H "x-secret-slogan: let_me_in" -v
```
You should see the following message printed in the console:
```bash
2024-10-24T09:14:19.519+03:00  INFO 68062 --- [0.1-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 0 ms

Successfully passed with expected slogan!
```
### Tests

Tests are very important. Every functionality should be backed by a set of sensible tests.

In the following test, we check that the public end point is accessible without authentication.

```java
@SpringBootTest
@AutoConfigureMockMvc
class SecureResourceControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @Test
  public void accessHomepageUnauthenticated() throws Exception {
    mockMvc.perform(get("/"))
           .andExpect(status().isOk());
  }
} 
```

## Step 4: Custom AuthenticationProvider Implementation

`AuthenticationProvider` allows us to implement a specific type of authentication.

As our implementation is for demonstration purposes, we wanna restrict access to the secured resource to the admin user. And to simplify it further, we even won't check the password. 

The `AuthenticationManager` will call our authentication provider. If our custom `AuthenticationProvider` cannot authenticate the user, the `AuthenticationManager` will task the next one in its list of authentication providers until the user gets authenticated. The request will fail with a HTTP status `401 Unauthorized`, if the user cannot be authenticated.

```java
public class AdminAuthProvider implements AuthenticationProvider {

  @Override
  public Authentication authenticate(Authentication auth) throws AuthenticationException {
    // for the admin user, ignore the password available in auth.getCredentials()
    if (Objects.equals(auth.getName(), "admin")) {
      final UserDetails user = User.withUsername("admin")
          .password("** cleared-for-security-reasons **")
          .roles("ADMIN")
          .build();

      return
          UsernamePasswordAuthenticationToken.authenticated(user, null, user.getAuthorities());
    }

    // reaching this point means that authentication was unsuccessful.
    // signal the AuthenticationManager, that it should try the 
    // next AuthenticationProvider in its list.

    return null;
  }

  @Override
  public boolean supports(Class<?> authentication) {
    return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
  }
}
```
Now we've to register our `AuthenticationProvider` in `SecurityConfig`:

```java
return 
    httpSec.authorizeHttpRequests(...)
       ...
       .httpBasic(withDefaults())
       // place our filter before logout filter
       .addFilterBefore(new SloganFilter(), LogoutFilter.class)
       // our custom AuthenticationProvider
       .authenticationProvider(new AdminAuthProvider()) 
       .build();
...
```

Add some tests:

```java
...
  @Test
  public void denyUnauthenticatedAccessToSecuredEndpoint() throws Exception {
    mockMvc.perform(get("/secured"))
        .andExpect(status().isUnauthorized());
  }

  @Test
  public void allowAuthenticatedAccessToSecuredEndpoint() throws Exception {
    // first login
    // after a successful login we're redirected to /secured end point
    mockMvc.perform(
            formLogin("/login")
                .user("admin")
                .password("doesNotMatterAsItIsIgnored")
        )
        .andExpect(status().is3xxRedirection())
        .andExpect(header().string("Location", "/secured"))
        .andExpect(authenticated().withUsername("admin"));
  }
...
```

## Wrap-Up

In this post I did give a short overview of the Spring Security. We did implement a very simple demo to demonstrate it.

In the next posts we'll cover also `Authorization`, which we did leave out completely.

---
# References
- [Source code on GitHub](https://github.com/tokalak/spring-security-demo)
- [Spring Security Project](https://spring.io/projects/spring-security)

