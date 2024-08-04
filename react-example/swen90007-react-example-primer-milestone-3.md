# Milestone 3: Authentication

By the end of this milestone we will have:

- a single admin user that can be authenticated and authorised to perform certain privileged actions
- an API with login and logout endpoints
- an API that is able to generate and verify JSON Web Tokens, and set browser cookies
- a Spring Security configuration that secures privileged resources
- a UI that will manage a user session via Local Storage and cookies
- the system deployed to, and running on, Render's unified cloud

:::{admonition} Skipping milestones
:class: caution
This milestone builds upon the implementation contributed by previous milestones, if you've skipped those previous milestones, ensure that:

- your development environment is set up appropriately, as demonstrated in *Milestone -1: Tools*
- you have an appropriately set up PostgreSQL server running locally, with a database and user configured appropriately, as demonstrated in *Milestone 0: Hello world*
- you've checked out the source code contributed by *Milestone 2: Core Functionality*, and run the `.sql` scripts contributed by that milestone against your local PostgreSQL server.
:::

## An overview

### Access tokens

The extensions contributed by this milestone are relatively complex, they implement an authorisation flow that's negotiated between the UI and the API, lets take moment to discuss what we are trying to achieve.

The following - rather large - diagram details the authorisation flow that takes place when a user attempts to access the privileged admin dashboard.

```{mermaid}
sequenceDiagram
    actor user as User
    participant ui as UI
    participant ls as BrowserLocalStorage
    participant api as API
    participant dir as UserDirectory
    participant jwt as JWTService
    participant t as token:JWT
    
    user->>ui: navigate(adminDashboard)
    activate ui
    ui->>ls: token = getLoggedInUser()
    alt token does not exist
        ui-->>user: loginPage
        user->>ui: submitCredentials(credentials)
        ui->>api: login(credentials)
        activate api
        api->>dir: user = getUser(credentials)
        alt user exists
            api->>jwt: token = generateAndSign(user)
            api-->>ui: token
            ui->>ls: setLoggedInUser(token)
            ui-->>user: adminDashboard<br/>(authentication flow starts again)
        else 
            api-->>ui: 401 Unauthorised
            deactivate api
            ui-->>user: invalid credentials
        end
    else token exists
        ui->>t: roles = getClaims()
        alt roles.contains("ADMIN")
            ui->>api: getVotes(token)
            activate api
            api->>jwt: verified = verify(user.token)
            alt !verified
                api-->>ui: 401 Unauthorised
                ui->>ls: removeLoggedInUser()
                ui-->>user: loginPage
            else
                api->>t: roles = getClaims()
                alt roles.contains("ADMIN")
                    api-->>ui: votes
                    ui-->>user: adminDashboad with votes
                else
                    api-->>ui: 403 Forbidden
                    deactivate api
                    ui-->>user: You are not an admin!
                end
            end
        else
            ui-->>user: You are not an admin!
            deactivate ui
        end
    end
```

There's a fair amount going on here, so let's pick it apart:

1. when a user attempts to access the admin dashboard the UI checks the browser's Local Storage for any user details (such as a token) that may have been stored there from a previous successful login
2. if no user details exist in Local Storage, then we need to authenticate the user, and then authorise them as an admin before we can proceed. The following occurs:
    1. The UI redirects the user to the login page
    2. The user enters their credentials which the UI sends to the API to be verified
    3. if the credentials match a user in the UserDirectory the API, via the JWTService, generates an access token with the user principal (username) and roles added as claims - claims are just extra metadata that can be added to a token, they capture additional user information that may be of use to the UI. The token is finally returned to the UI where it is added to the browser's Local Storage and the user is redirected back to the admin dashboard where the flow starts again (this time with user details, of course)
    4. if the UserDirectory has no such user then the credentials are assumed to be wrong - an error is returned UI,  which prompts the user to try again
3. if user details exist and indicate that the current user does not have the required `ADMIN` role then the UI simply responds with an error, the user is not an admin.
4. however, if the user details are for an admin then the UI requests the votes from the API, passing the access token (from Local Storage) as authorisation. The following then occurs:
    1. before honouring the request, the API verifies the received token. The API verifies that the token is one it has generated (of course, we can't just accept any token generated by anyone - so tokens are signed with a secret key that only the API knows), that the token has not been tampered with (for example, by an attacker altering the token's claims to escalate their privilege), and finally that the token has not yet expired.
    2. if the token can't be verified, an error is returned to the UI, the user details are stripped from the Local Storage and user is redirected to the login page.
    3. otherwise, the API checks the token's claims for the required `ADMIN` role.  

### Refresh tokens

As mentioned above, the access token that the API generates will eventually expire. It's best practice to give tokens a very short lifespan - this ensures that if a token is stolen by an attacker it can only be used for a short period, thus minimising the opportunity the attacker has to access privileged resources. Of course, if the token's lifespan is very short then the user will need to fetch a new one often, and if they're required to login each time to do so then this will get rather annoying. We'll introduce the concept of a *refresh token* to enable users to stay logged in, while also ensuring that tokens expire shortly after they're generated. The refresh token will be generated alongside the access token, stored in the database, and can be presented by the UI to automatically receive a new access token.

```{mermaid}
sequenceDiagram
    actor user as User
    participant ui as UI
    participant ls as BrowserLocalStorage
    participant api as API
    participant dir as UserDirectory
    participant jwt as JWTService
    participant rep as RefreshTokenRepository

    user->>ui: navigate(adminDashboard)
    activate ui
    ui->>ls: token = getLoggedInUser()
    Note over ui: assuming that the<br/>token has expired
    ui->>api: 401 Unauthorized = getVotes(token)
    activate api
    ui->>api: refresh(token, refreshTokenId)
    api->>rep: refreshToken = get(refreshTokenId)
    alt refreshToken does not exist || token != refreshToken.accessToken
        api-->>ui: 403 Forbidden
        ui-->>user: You are not an admin
    else
        api->>dir: user = getUser(refreshToken.username)
        api->>jwt: generateAndSign(user)
        activate jwt
        jwt->>jwt: newToken = new Token()
        jwt->>jwt: newRefreshToken = new RefreshToken(refreshToken.username, token)
        jwt->>rep: add(newRefreshToken)
        jwt->>rep: remove(refreshToken)
        jwt-->>api: newToken, newRefreshToken
        deactivate jwt
        api-->>ui: newToken,<br/>newRefreshTokenId = newRefreshToken.id
        deactivate api
        ui->>ls: setLoggedInUser(newToken)
        ui->>api: getVotes(newToken)
        activate api
        Note over api: authentication flow from above<br/>executes, ie validate token etc
        api-->>ui: votes
        deactivate api
        ui-->>user: adminDashboard with votes
        deactivate ui
    end
```

And here is a short explanation of how the refresh flow works:

1. assuming that the token has expired, the UI will make a call to retrieve the privileged votes resource, which will - naturally - be rejected by the API
2. the UI detects the 401 status received and begins negotiating a token refresh by requesting a new token via the API's refresh route, the UI provides the expired token as well as the refresh token's ID (set as a *cookie* after a previous login, more on cookies later)
3. if the provided refresh token ID is valid (a refresh token exists in the database) and the associated access token matches the token sent by the UI, then the API:
    1. generates a new access token
    2. generates a new refresh token and associates it with the new access token
    3. persists the new refresh token, and invalidates the original refresh token (so that it can't be used again) by purging it from the database
    4. returns both the new access and refresh tokens to the UI - it sets the refresh token as a cookie
4. the UI stores the new access token for later use, and the browser caches the refresh token cookie
5. the UI finally retries the original request to retrieve the votes resource with the new access token

## Persisting refresh tokens

The proposed security solution requires persisting refresh tokens, this means we'll need to extend the database schema. Make the following changes to `init.sql`:

```sql
BEGIN;

-- existing schema omitted, for brevity

CREATE TABLE app.refresh_token (
    id uuid NOT NULL UNIQUE,
    token_id VARCHAR(36) NOT NULL,
    username VARCHAR(255) NOT NULL,
    PRIMARY KEY (id)
);

CREATE INDEX refresh_token_username
ON app.refresh_token (username);

COMMIT;
```

Then be sure to update your local database with the new schema.

## An API secured by Spring Security

### Modelling the tokens

Our system will, obviously, need to manage access and refresh tokens. We'll create a very simple model for each, so that we can implement the rest of the system in a type-safe way.

The `Token` model merely captures the signed token as a string, as well as the associated refresh token's ID.

```java
package com.unimelb.swen90007.reactexampleapi.security;  
public class Token {  
    private String accessToken;  
    private String refreshTokenId;  
  
    // accessors omitted, for brevity  
}
```

And a `RefreshToken`, which can be persisted in the database (has an ID) and maps an access token's ID to a username.

```java
package com.unimelb.swen90007.reactexampleapi.security;  
public class RefreshToken {  
    private String id;  
    private String tokenId;  
    private String username; 
  
    // accessors omitted, for brevity  
}
```

### A refresh token repository

Our security components will need access to persisted refresh tokens. Just like we did for the votes, we'll implement a very simple mapper, `RefreshTokenRepository`. The interface is provided below, see the source code for the implementation - `com.unimelb.swen90007.reactexampleapi.port.postgres.PostgresRefreshTokenRepository`.

```java
package com.unimelb.swen90007.reactexampleapi.security; 
  
import java.util.Optional;  
  
public interface RefreshTokenRepository {  
    Optional<RefreshToken> get(String id);  
    void save(RefreshToken token);  
    void deleteAllForUsername(String username);  
    void delete(String id);  
}
```

One thing to note is that this interface exposes a `deleteAllForUsername` method, which will invalidate all refresh tokens for a user - effectively, logging them out.

### Generating and validating JSON Web Tokens

Our API will need to generate and validate access tokens in response to user requests. The [JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519) standard provides a simple, and popular, means for signing and verifying user claims (such as, in this instance, their role). There are many great implementations of this standard - for many languages ([jwt.io](https://jwt.io/libraries)maintains a comprehensive list) - we'll use the [Java JWT](https://github.com/jwtk/jjwt) project, it covers the entire specification and provides a nice, fluent API .

Add Java JWT to the project by providing the following Maven dependency coordinates:

```xml
<properties>    
    <jjwt.version>0.11.5</jjwt.version> 
</properties>

<dependancies>
 <dependency>  
     <groupId>io.jsonwebtoken</groupId>  
     <artifactId>jjwt-impl</artifactId>  
     <version>${jsonwebtoken.version}</version>  
     <scope>runtime</scope>  
 </dependency>  
 <dependency>  
     <groupId>io.jsonwebtoken</groupId>  
     <artifactId>jjwt-api</artifactId>  
     <version>${jsonwebtoken.version}</version>  
 </dependency>  
 <dependency>  
     <groupId>io.jsonwebtoken</groupId>  
     <artifactId>jjwt-jackson</artifactId>  
     <version>${jsonwebtoken.version}</version>  
     <scope>runtime</scope>  
 </dependency>
</dependancies>
```

There are components of the application that will need to process JWTs; a number of Servlet endpoints (for example, a login and refresh endpoint), which will generate tokens; and a Spring Security provided *Servlet Filter*, which will validate tokens provided as authorisation to privileged resources. To keep the handling of tokens consitant (and to observe *High Cohesion*), we'll implement a `TokenService` that both the Servlets and Spring Security components can delegate to.

```java
package com.unimelb.swen90007.security;  
  
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;  
import org.springframework.security.core.userdetails.UserDetails;  
  
public interface TokenService {  
    UsernamePasswordAuthenticationToken readToken(String accessToken);  
    Token createToken(UserDetails user);  
    Token refresh(String accessToken, String refreshTokenId);  
    void logout(String username);  
}
```

Let's clarify what each method provides and how it fits into the rest of the application:

1. both `UserDetails` and `UsernamePasswordAuthenticationToken` are Spring Security components; the former captures the details of users known to the application (ie users that exist in the *UserDirectory*, as shown in the high level flows diagramed at the start of this milestone); and the latter is used by Spring Security to enforce authorisation rules for privileged resources
2. the `readToken` method will translate a JWT (and its associated claims) into an instance of a `UsernamePasswordAuthenticationToken`, which Spring Security knows how to authorise
3. the `createToken` will generate a JWT for a Spring Security managed user, which implies we will need a way to configure Spring Security with our admin user details (more on that later)
4. the `refresh` method will manage the refreshing of access tokens, and to do so will need access to a `RefreshTokenRepository`

And, Let's take a look at the implementation, `JwtTokenServiceImpl`:

```java
package com.unimelb.swen90007.security;  
  
public class JwtTokenServiceImpl implements TokenService {  

    private SecretKey key;
        private final RefreshTokenRepository repository;  

    // constructors, some methods, and fields ommited, for brevity

    @Override  
    public UsernamePasswordAuthenticationToken readToken(String accessToken) {  
        try {  
            var jws = parse(accessToken);  
            return new UsernamePasswordAuthenticationToken(getUsername(jws.getBody()), null, getAuthorities(jws.getBody()));  
        } catch (ExpiredJwtException e) {  
            throw new CredentialsExpiredException("token expired");  
        } catch (JwtException e) {  
            throw new BadCredentialsException("bad token");  
        }  
    } 
  
    @Override  
    public Token createToken(UserDetails user) {  
        return generateToken(user.getUsername(), user.getAuthorities());  
    }  
  
    @Override  
    public Token refresh(String accessToken, String refreshTokenId) {  
        var claims = parseExpired(accessToken);  
        var refresh = repository.get(refreshTokenId)  
                .orElseThrow(() -> new BadCredentialsException("bad refresh token"));  
        if (!refresh.getTokenId().equals(claims.getId())) {  
            throw new BadCredentialsException("bad refresh token");  
        }  
        repository.delete(refreshTokenId);  
        return generateToken(getUsername(claims), getAuthorities(claims));  
    } 
  
    @Override  
    public void logout(String username) {  
        repository.deleteAllForUsername(username);  
    }  
  
    private Token generateToken(String username, Collection<? extends GrantedAuthority> authorities) {  
  
        var now = new Date();  
        var expires = Date.from(now.toInstant().plusSeconds(timeToLiveSeconds));  
        var id = UUID.randomUUID().toString();  
        var tokenStr = Jwts.builder()  
                .setIssuer(issuer)  
                .setSubject("swen90007-template-api")  
                .setAudience(issuer)  
                .setExpiration(expires)  
                .setNotBefore(now)  
                .setIssuedAt(now)  
                .setId(id)  
                .claim(CLAIM_USERNAME, username)  
                .claim(CLAIM_AUTHORITIES, authorities.stream().map(GrantedAuthority::getAuthority).toList())  
                .signWith(getKey())  
                .compact();  
  
        var refreshToken = new RefreshToken();  
        refreshToken.setId(UUID.randomUUID().toString());  
        refreshToken.setTokenId(id);  
        refreshToken.setUsername(username);  
        repository.save(refreshToken);  
  
        var token = new Token();  
        token.setAccessToken(tokenStr);  
        token.setRefreshToken(refreshToken.getId());  
        return token;  
    }  
    
    private Jws<Claims> parse(String token) {  
        return Jwts.parserBuilder()  
                .setSigningKey(getKey())  
                .requireAudience(this.issuer)  
                .build()  
                .parseClaimsJws(token);  
    }
  
    private SecretKey getKey() {  
        if (key == null) {  
            key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));  
        }  
        return key;  
    }  
}
```

1. JWTs are signed so that dependent systems can verify their authenticity and integrity. To sign tokens, we need a `SecretKey` - which is generated from a random (and thus, hard to guess) string that only the API knows. The `parse` method performs this verification, Java JWT does all the heavy lifting here
2. the service will manage access tokens as well as their associated refresh tokens, it needs access to an implementation of the `RefreshTokenRepository`
3. `readToken` simply parses the provided token and returns a new instance of `UsernamePasswordAuthenticationToken` for Spring Security to process. Spring Security uses *authorities* to identify which users are entitled to a resource; later we'll configure Spring Security to enforce these authorities - but for now, it's enough to know that we'll be keeping a record of the user's authorities within the token's claims, `readToken` simply has to extract them
4. `generateToken` is where the JWT magic starts; again, we'll rely on Java JWT to manage all the complicated stuff. The key takeaway here is that: the user's username and authorities are added as claims, which can be extracted for later use in subsequent requests; the token is signed; and a refresh token is registered
5. `refresh` performs a series of check before invoking and returning the result of `generateToken` - it verifies the access and refresh tokens, and asserts that they are associated.  
6. finally, `logout` invalidates all refresh tokens generated by a user - because the access tokens are short lived they'll soon expire, any cached (or stolen!) tokens will - very quickly - be useless.

### Login, refresh and logout resources

Our API will need to serve tokens to the UI, we'll add two new Servlets for this purpose; a `TokenResource` Servlet that will accept `POST` and `PUT` requests to an `/auth/token` endpoint - implementing login and token refresh, respectively; and `LogoutResource`, for - no surprises - logout. This is not our *first rodeo* with respect to Servlets, so we won't discuss either of them in depth here (check out the source code instead); however, we will very quickly take a look at how the `TokenResource` manages browser cookies.

:::{admonition} Cookies
:class: note
Cookies are managed by the browser and provide a means by which an application can cache such things as login information for session management, or other data to provide a personalised experience. The server can request that a user's browser cache a cookie by setting one or more  `Set-Cookie` headers on the response; after which point the browser includes the cookie with subsequent requests by setting the `Cookie` header. Because cookies often store sensitive information they are a prime target for attackers seeking to impersonate legitimate users, and so it's important to ensure that a few best practices are followed when using cookies - we'll discuss a few of these practices, but you should check out the [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) for an in-depth treatment.
:::

The Servlet API exposes the cookies on both the `HttpServletRequest` and `HttpServletResponse`, we can extract a `Cookie` object from the former by filtering the return of its `getCookies` method, and set a required `Cookie` on the later via its `addCookie` method.  The following `TokenResource` snippet shows how we'll create a cookie for the refresh token's ID.

```java
private Cookie refreshCookie(String refreshToken, String contextPath) {  
    var cookie = new Cookie(COOKIE_NAME_REFRESH_TOKEN, refreshToken);  
    cookie.setSecure(secureCookies);  
    cookie.setMaxAge(cookieTimeToLiveSeconds);  
    cookie.setHttpOnly(true);  
    cookie.setDomain(domain);  
    cookie.setPath(contextPath + PATH_AUTH_TOKEN); 
    return cookie;  
}
```

1. our cookie contains sensitive information that may be of interest to attackers, we'll keep it safe in production by ensuing that it's only sent over a secure HTTPS connection by setting `setSecure` to `true` - we won't be configuring HTTPS locally so we'll make sure the value passed to `setSecure` is configurable on a per environment basis
2. the cookie should eventually expire, but we'll make it's time to live relatively long, so that the user isn't required to login again if they return to the UI in the future
3. our UI's JavaScript won't need access to this cookie, so we'll set `httpOnly` to `true`, and in the process protect it from any malicious JavaScript that an attacker might be able to load with the user's browser
4. we don't want this cookie to be sent to any system other than the API, so we'll set the domain
5. the cookie is only required for `PUT` requests to `/auth/token`, so we indicate this to the browser by setting an appropriate path (taking into account the full context path on which the application will be served).

### Spring Security configuration

So we have an API that is able to serve, refresh and invalidate access tokens; and while this might be a great, and necessary, first step, it's not - on its own - sufficient to secure the application. For that we'll need to ensure that the security measures we've implemented are enforced. We'll leverage our current Spring Security integration for this purpose by extending it's configuration to manage a user directory as well as protect our privileged resources with the assistance of a custom Servlet Filter that knows how to validate a received access token.

#### Context aware configuration

As we make changes to the Spring Security configuration we'll define a number of components, such as a `UserDetailsService`, on which a number of our custom Servlet implementations will depend. We can expose these components to our Servlets by having our `WebSecurityConfig` implement the `ServletContextAware` interface, which exposes a single method, `setServletContext`. The `ServletContext` is passed as an argument to `setServletContext`, providing us an opportunity to register the Servlet dependencies.

#### A static, in memory user directory

Before our application can authenticate and authorise users it, naturally, needs to know which users exist. We can provide Spring Security information about which users exist by implementing and registering an implementation of the `UserDetailsService` interface. Thankfully Spring Security ships with a number of simple implementations, such an `InMemoryUserDetailsManager`, that are suitable for our purposes. The following methods of our extended Spring Security configuration construct the required `UserDetailsService`:

```java
@Bean  
public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {  
    var userDetailsService = new InMemoryUserDetailsManager();  
    userDetailsService.createUser(adminUser(passwordEncoder));  
    userDetailsService.createUser(nonAdminUser(passwordEncoder));  
    servletContext.setAttribute(VoteContextListener.USER_DETAILS_SERVICE, userDetailsService);  
    return userDetailsService;  
}

private UserDetails adminUser(PasswordEncoder passwordEncoder) {  
    return new User(System.getProperty(PROPERTY_ADMIN_USERNAME),  
            passwordEncoder.encode(System.getProperty(PROPERTY_ADMIN_PASSWORD)),  
            Collections.singleton(Role.ADMIN.toAuthority()));  
}  
  
private UserDetails nonAdminUser(PasswordEncoder passwordEncoder) {  
    return new User("user",  
            passwordEncoder.encode("user"),  
            Collections.singleton(Role.USER.toAuthority()));  
}  
  
@Bean  
public PasswordEncoder passwordEncoder() {  
    var passwordEncoder = new BCryptPasswordEncoder();  
    servletContext.setAttribute(VoteContextListener.PASSWORD_ENCODER, passwordEncoder);  
    return passwordEncoder;  
}
```

1. in `userDetailsService` we instantiate an `InMemoryUserDetailsManager` and register two users - the privileged admin user, and an unprivileged *test* user (which we'll make use of when demonstrating proper authorisation via Spring Security, you can - and probably should - remove this user once you're satisfied that you're implementation behaves as required)
2. `adminUser` creates the `UserDetails` for our admin user, configuring a username and password provided by environmental configuration, and `ADMIN` authority
3. user passwords are stored encrypted, even in memory, so we provide a `PasswordEncoder` that uses the [bcrypt](https://en.wikipedia.org/wiki/Bcrypt) password-hashing function

:::{admonition} In-memory user details
:class: caution
Note that this `UserDetailsService` is backed by an in-memory store; there is no integration with a persistent store, such as a database, and so all user details are lost when the application is restarted. This is fine for our purposes; we register a static directory of users on start up, and we have no use case *user registration*, which would require a dynamic directory that can persist the details of newly registered users to disk.
:::

#### Token aware Servlet Filter

The way Spring Security applies its configuration to a request is via a Servlet Filter. Servlet Filters are incredibly useful if you have a requirement to process each request to the application in a generic way - like enforcing authentication and authorisation (and another great use case is logging).

The Servlet Filter we need to implement extends Spring Security's `AbstractAuthenticationProcessingFilter`. Let's take a look at this implementation.

```java
package com.unimelb.swen90007.reactexampleapi.security;  
  
import jakarta.servlet.FilterChain;  
import jakarta.servlet.ServletException;  
import jakarta.servlet.http.HttpServletRequest;  
import jakarta.servlet.http.HttpServletResponse;  
import org.springframework.security.authentication.BadCredentialsException;  
import org.springframework.security.core.Authentication;  
import org.springframework.security.core.AuthenticationException;  
import org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter;  
import org.springframework.security.web.util.matcher.RequestMatcher;  
  
import java.io.IOException;  
import java.util.Optional;  
import java.util.regex.Pattern;  
  
public class TokenAuthenticationFilter extends AbstractAuthenticationProcessingFilter {  
    public static final String TOKEN_TYPE_BEARER = "Bearer";  
    private static final String HEADER_AUTHORIZATION = "Authorization";  
    private static final Pattern PATTERN_TOKEN = Pattern.compile("^" + TOKEN_TYPE_BEARER + " (.*)$");  
  
    private final TokenService jwtTokenService;  

    // constructors omitted, for brevity

    @Override  
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {  
        var authorizationHeader = Optional.ofNullable(request.getHeader(HEADER_AUTHORIZATION))  
                .orElseThrow(() -> new BadCredentialsException(String.format("%s header is required", HEADER_AUTHORIZATION)));  
  
        var matcher = PATTERN_TOKEN.matcher(authorizationHeader);  
        if (matcher.find()) {  
            var token = matcher.group(1);  
            return getAuthenticationManager().authenticate(jwtTokenService.readToken(token));  
        }  
        throw new BadCredentialsException(String.format("invalid %s header value", HEADER_AUTHORIZATION));  
    }  
  
    @Override  
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {  
        super.successfulAuthentication(request, response, chain, authResult);  
        chain.doFilter(request, response);  
    }  
}
```

1. in `attemptAuthentication` we're searching for, extracting (using a regex) and verifying (using the `TokenService`) an access token sent with the request.
2. if there is a valid token we pass the corresponding `UsernamePasswordAuthenticationToken` instance to an `AuthenticationManager` - another Spring Security component, with access to the user directory.
3. if the token is invalid we throw a `BadCredentialsException`, which will be caught by Spring Security and transmitted as an appropriate response to the client.
4. if the authentication is successful then Spring Security invokes the `successfulAuthentication` method, where we simply pass to the next component in the filter chain, which eventually invokes the Servlet itself.

The final thing to do is to actually configure Spring Security to use all the custom authentication that we have implemented. Primarily, we need to configure which routes require authentication and authorisation, but because we are using token based security we'll also configure Spring to ignore a whole bunch of default behaviour that doesn't suit our purpose.

```java
@Bean  
public SecurityFilterChain securityFilterChain(HttpSecurity http, AuthenticationManager authenticationManager, TokenAuthenticationFilter tokenAuthenticationFilter, AuthenticationEntryPoint authenticationEntryPoint) throws Exception {  
    return http  
            .sessionManagement()  
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)  
            .and()  
            .exceptionHandling()  
            .defaultAuthenticationEntryPointFor(authenticationEntryPoint, PROTECTED_URLS)  
            .and()  
            .addFilterBefore(tokenAuthenticationFilter, AnonymousAuthenticationFilter.class)  
            .authorizeHttpRequests(authorize -> authorize  
                    .requestMatchers(PROTECTED_URLS)  
                        .hasRole(Role.ADMIN.name())  
                    .anyRequest()  
                        .permitAll())  
            .authenticationManager(authenticationManager)  
            .cors(Customizer.withDefaults())  
            .csrf().disable()  
            .httpBasic().disable()  
            .formLogin().disable()  
            .logout().disable()  
            .build();  
}
```

The `securityFilterChain` method of our `WebSecurityConfig` includes the bulk of this configuration.

1. because we are using token based security we don't need Spring Security to manage a session so we'll set the `sessionCreationPolicy` to `STATELESS`
2. we'll register our custom token filter via `addFilterBefore`
3. `authorizeHttpRequests` is where we configure which resources require authorisation and which roles we'll accept, we configure the `/manage/vote` to require the `ADMIN` role (*authority*) - all other resources can be accessed anonymously.
4. we'll keep CSRF protections disabled, and also disable Basic Auth, the default login form and logout functionality.

### Verify API security

Before moving onto extending the UI, let's pause to verify that the API authentication works as expected.

We've introduced a bunch of new components, some of them with environmental configuration. Add the following Java System Properties to you run configuration:

| property                     | value                                                                                                                                                                                                        |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `jwt.secret`                 | `7zRrC0HsdVyzOAUrH+1FyN2+Iqv7QlM9A6oU0q7Qi+k=` (you should change this in production - you can generate your own sufficiently secure key, on Unix-like systems, with `sudo head -c32 < /dev/random \| base64`) |
| `jwt.timeToLive.seconds`     | `60`                                                                                                                                                                                                         |
| `jwt.issuer`                 | `react-example-api.unimelb.edu.au` (you should also change this for production)                                                                                                                              |
| `cookies.timeToLive.seconds` | `360000`                                                                                                                                                                                                     |
| `cookies.domain`             | `localhost`                                                                                                                                                                                                  |
| `cookies.secure`             | `false`                                                                                                                                                                                                      |
| `admin.username`             | `admin`                                                                                                                                                                                                        |
| `admin.password`             | `admin` (again, make sure you change this for production)                                                                                                                                                                                                             |

Once you have you server started, try to log in the admin user by creating an access token with the admin credentials:

```shell
http :8080/react_example_api_war_exploded/auth/token username=admin password=admin -v
```

You should receive something that looks like this:

```json
{
    "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJyZWFjdC1leGFtcGxlLWFwaS51bmltZWxiLmNvbSIsInN1YiI6InN3ZW45MDAwNy10ZW1wbGF0ZS1hcGkiLCJhdWQiOiJyZWFjdC1leGFtcGxlLWFwaS51bmltZWxiLmNvbSIsImV4cCI6MTY5NDQ4NDExMiwibmJmIjoxNjk0NDg0MDUyLCJpYXQiOjE2OTQ0ODQwNTIsImp0aSI6ImEzMTgzOWQyLWMyNDItNDQ0Ni05ZmM4LWVkOWE0MWZhNmE1OSIsInVzZXJuYW1lIjoiYWRtaW4iLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl19.iqHKPlmKpyODw9Tl2lXPlZ7aL_jCoxo3CbPgta11BmM",
    "type": "Bearer"
}
```

Also notice that we receive a `Set-Cookie` header in the response, make a note of it, we'll use it later

```txt
Set-Cookie: refreshToken=38e30e73-d63b-41d2-a394-3e0976094196; Max-Age=360000; Expires=Sat, 16 Sep 2023 06:25:17 GMT; Domain=localhost; Path=/auth/token; HttpOnly
```

now lets try to access the secured `/manage/vote` resource, providing - as authorisation - the token received previously.

```shell
http :8080/react_example_api_war_exploded/manage/vote Authorization:'Bearer <access-token>'
```

You should receive the votes. Now let's try without the `Authorization` header:

```shell
http :8080/react_example_api_war_exploded/manage/vote
```

You should receive an error, *401 Unauthorized*, as expected. Create a token for the unprivileged *test* user:

```shell
http :8080/react_example_api_war_exploded/auth/token username=user password=user
```

And attempt to access the privileged `/manage/vote` resource, you should get an error. Finally lets try to login a user that the system is unable to authenticate:

```shell
http :8080/react_example_api_war_exploded/auth/token username=unknown password=shhhhh
```

It should throw an error. Let's exercise the token expiration, wait for the admin's token to expire (about a minute with the configuration above), and resend the request to access the `/manage/vote` resource, it should fail. Finally let's try to refresh that expired token

```shell
http PUT :8080/react_example_api_war_exploded/auth/token accessToken=<expired-access-token> Cookie:'refreshToken=<refresh-token-from-previous>'
```

You should get a new access and refresh token. Now send the same request again, notice that it returns an error because the refresh token has already been claimed.

## UI extensions

Now that we have an API that's properly secured, let's move onto extending the UI to provide login and logout features, as well as authorising access to the admin dashboard.

### Client changes

We have a number of new resources exposed by the API (for example `/auth/token`, `/logout`, etc), naturally we'll need to extend our client to consume these new resources. Additionally, the API now requires a valid `Authorization` header to be sent for some resources, our extended client will need to handle this too. Let's first checkout the login and logout methods.

```jsx
async login(username, password, signal) {
    const res = await this.doCall('/auth/token', 'POST', { username, password }, signal);
    const token = await res.json();
    setTokenInStorage(token);
    return token;
}

async logout(username, signal) {
    await this.doCall('/auth/logout', 'POST', { username }, signal);
    localStorage.removeItem(STORAGE_ITEM_ACCESS_TOKEN);
    localStorage.removeItem(STORAGE_ITEM_TOKEN_TYPE);
}
```

1. both of these methods manipulate Local Storage. The `login` method sets the received token in Local Storage, and the `logout` method clears it (after sending a logout request to the API)
2. `login` returns the token to the calling code, so that it can extract the claims and provide a personalised experience

Now let's look at the changes to the `doCall` method:

```jsx
async doCall(path, method, data, signal) {
    const headers = {
        'Content-Type': CONTENT_TYPE_APPLICATION_JSON,
        Accept: CONTENT_TYPE_APPLICATION_JSON,
    };
    const accessToken = localStorage.getItem(STORAGE_ITEM_ACCESS_TOKEN);
    if (accessToken) {
        headers.Authorization = `${localStorage.getItem(STORAGE_ITEM_TOKEN_TYPE)} ${accessToken}`;
    }
    let body;
    if (data) {
        body = JSON.stringify(data);
    }
    const res = await fetch(
        `${this.baseUrl}${path}`,
        {
            method,
            headers,
            body,
            signal,
            credentials: 'include',
        },
    );
    if (res.status === 401 && accessToken) {
        await this.refreshToken(accessToken, signal);
        return this.doCall(path, method, data, signal);
    }
    if (res.status > 299) {
        throw new Error(`expecting success from API for ${method} ${path} but response was status ${res.status}: ${res.statusText}`);
    }
    return res;
}
```

1. the `doCall` method checks to see if there is a token in Local Storage, and if there is one, sets this in the `Authorization` header for each request
2. if the client receives a `401` response status from the API and there is a token stored in Local Storage it's likely that the token has expired, so the client attempts to refresh the token before retrying the request

Finally, here is the `refreshToken` method

```jsx
async refreshToken(accessToken, signal) {
    const path = '/auth/token';
    const res = await fetch(
        `${this.baseUrl}${path}`,
        {
            method: 'PUT',
            headers: {
                'Content-Type': CONTENT_TYPE_APPLICATION_JSON,
                Accept: CONTENT_TYPE_APPLICATION_JSON,
            },
            body: JSON.stringify({
                accessToken,
            }),
            signal,
            credentials: 'include',
        },
    );
    if (res.status > 299) {
            throw new Error(`expecting success from API for PUT ${path} but response was status ${res.status}: ${res.statusText}`);
    }
    const token = await res.json();
    setTokenInStorage(token);
    return token;
}
```

1. the `refreshToken` method invokes a `PUT` against the API's refresh resource
2. if the request is successful we update Local Store, just as we would for login
3. you might be wondering why we don't just invoke `doCall`, a fair amount of code is repeated here - the reason is that invoking `doCall` runs the risk of entering an infinite loop if the refresh request fails due to invalid authorisation

### An authentication provider Context

We learnt about React Contexts in previous milestones, and they're just as useful here. We'll use a Context to manage user session details, such as their name, if they're in fact authenticated or not, and a number of functions that other components can call to execute login or logout. The `AuthentcationProvider` implementation is rather involved - though nothing too different from what we've seen before with previous Contexts - so we'll just focus on a small section, you can check out the rest of the implementation on GitHub.

```jsx
const extractUserFromToken = (token) => JSON.parse(atob(token.split('.')[1]));

const login = async (username, password) => {
    setAuthenticating(true);
    setAuthenticationError(undefined);
    try {
        const token = await api.login(username, password);
        setUser(extractUserFromToken(token.accessToken));
    } catch (e) {
        setAuthenticationError(JSON.stringify(e));
    } finally {
        setAuthenticating(false);
    }
};

const value = useMemo(
    () => ({
        user, login, logout, authenticationError, authenticating,
    }),
    [user, authenticationError, authenticating],
);
```

1. the token returned by the API is not encrypted (though it *is* signed, to prevent tampering), the utility function `extractUserFromToken` extracts the token's claims (username and roles) by simply decoding the base64 encoded data
2. the `login` function invokes the `Api` client class we extended previously, if login is successful we'll extract the token's claims and set them on the Context, as well as update some housekeeping state such as `authenticating`, so that other components can be scheduled for re-render
3. like the `ApiProvider` Context, we'll manage our user session state internally with `useMemo`

### Login

We need a way for user to login, we'll provide that via `Login` container (page), that makes use of the new `AuthenticationProvider` Context and some fancy routing we've not seen yet. Let's take a look at the `handlLogin` function - the rest of the container is nothing we've not seen before.

```jsx
const handleLogin = async () => {
    await login(username, password);
    if (authenticationError) {
        return;
    }
    let next = '/';
    if (location.state && location.state.next) {
        next = location.state.next;
    }
    navigate(next);
};
```

1. `handleLogin` delegates all the heavy lifting to the `AuthenticationProvider` function `login`, it simply maps the fields of a form to `login`'s username and password parameters.
2. if the authentication fails, we'll exit early and display the error
3. otherwise we'll navigate the user to either the home page, or to the *next* page (in the case that whatever page originally redirected to this login page would rather us take the user elsewhere)

### Personalised experience and logout

If a user is logged in it would be nice if the UI indicated to the user that they we're in fact authenticated. The `LoggedInUserDetail` component does just this, it leverages the claims encoded within the received access token to greet the user, and additionally renders a button that the user can use to logout.

```jsx
export default function LoggedInUserDetail() {
    const { user, logout } = useAuthentication();
    
    return (
        user && (
            <div className={styles.LoggedInUserDetail}>
                <Card title={`Hi ${user.username || 'you'}`}>
                    <Button onClick={() => logout()}>Log out</Button>
                </Card>
            </div>
        )
    );
}
```

1. if a user exists then they must be logged in, so we'll render a button with an `onClick` attribute bound to the `logout` function provided by the `AuthenticationProvider` Context.
2. if a user exists then it might be that their access token has

### A *Higher-Order Component*

Higher-Order Components (HOC) are components that take a component and return that component wrapped in some additional functionality, or even another component entirely. HOCs are incredibly useful if you need to add some additional functionality to a component but would prefer not the change the implementation of that component in anyway (if you're familiar with the Gang of Four's *Decorator* pattern, then HOC is a classic example of this pattern in action). We already have a number of page components that work quite well, however some of them should only be accessed by authenticated admin users; introducing a HOC to manage authorisation enables us to both avoid extending these original page implementation (and run the risk of introducing a regression defect), and (perhaps more importantly) reuse our authorisation logic to secure any other parts of the application - current or future. Take a look at the `withAuthority` HOC:

```jsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuthentication } from '../contexts/AuthenticationProvider';

export default function WithAuthority({ children, authorities }) {
    const { user } = useAuthentication();
    const location = useLocation();
    
    if (user === undefined) {
        return null;
    }
    
    if (!user) {
        const next = location.pathname + location.search + location.hash;
        return <Navigate to="/login" state={{ next }} />;
    }
    
    if (user.authorities.filter((authority) => authorities.indexOf(authority) !== -1).length === 0) {
        return <Navigate to="/" />;
    }
    
    return children;
}
```

1. the `withAuthority` HOC wraps an arbitrary set of child components (`children`) and takes a list of strings (`authorities`) that configure which authorities the logged in user must have to access the child components
2. if there is no user yet - because, for example, the DOM is still in the process of rendering for the first time - then we'll simply return null and wait for the user to be initialised
3. if the user has been initialised and the user is `null`, then we know that the user is not authenticated, they'll need to authenticate before they can access the protected child components, se we'll redirect them to login page (you can provide the `state` prop to `Navigate` to enqueue a second redirect that - for example - the login page can use to return the user to the protected resource once authentication is complete)
4. if the user exists but doesn't have the required authority then we'll redirect them to the root of the application
5. if none of the above hold then it must be the case that the current user is authenticated and authorised to access the wrapped components, so we'll return them to be rendered

### Putting it all together with a new Router component

The final step is to adjust our React Router component implementation to expose the new login page, and apply our `withAuthority` HOC to the privileged `Votes` page.

```jsx
function AppRoutes() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/vote" element={<Vote />} />
                <Route
                    path="/admin/votes"
                    element={(
                        <WithAuthority authorities={['ROLE_ADMIN']}>
                            <Votes />
                        </WithAuthority>
                    )}
                />
                <Route path="/verdict" element={<Verdict />} />
                <Route path="/login" element={<Login />} />
                <Route path="*" element={<Navigate to="/" />} />
            </Routes>
        </BrowserRouter>
    );
}
```

Most of the above should look familiar - here's what's new:

1. the login page gets it's own route - `/login`
2. we're using the `withAuthority` HOC to wrap the `Votes` page, and configured it to only be exposed to users with the `ADMIN` authority

And that's it for the UI, you should be able to run this all locally now, just start the UI using NPM - as shown in previous milestones, remembering also to set the relevant environment variables. Try forc

## Deploy to Render

We've deployed to Render a few times now - if you need a refresher,  return to a previous milestone for detailed instructions on how to deploy; note that for this deployment you'll need to:

1. update your database schema
2. push changes for both the API and UI
3. and, finally, the API configuration has been extended, you'll need to add a few further properties to the `JAVA_OPTS` environment variable declared in Render, see the table below for details:

| property                     | value                                               |
| ---------------------------- | --------------------------------------------------- |
| `jwt.secret`                 | a long, hard to guess random string                 |
| `jwt.timeToLive.seconds`     | `60`                                                |
| `jwt.issuer`                 | set this to the domain of your Web Service          |
| `cookies.timeToLive.seconds` | `360000`                                            |
| `cookies.domain`             | set this - also - to the domain of your Web Service |
| `cookies.secure`             | `true`                                              |
| `admin.username`             | `admin`                                             |
| `admin.password`             | again, something hard to guess                      |

Congratulations, you now have a secure system!
