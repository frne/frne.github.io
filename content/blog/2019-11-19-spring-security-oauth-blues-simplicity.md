---
title: "The Spring Security Oauth2 Blues - Simplicity"
image: "images/post/2019-11-19-spring-security-oauth-blues-simplicity.jpg"
date: 2019-12-14T13:58:28+01:00
author: "Frank Neff"
tags: ["spring", "security" ]
categories: ["Programming"]
draft: false
---

I personally like the Spring Framework and its security components, because it's pretty full-featured and easy to use, 
but when it comes to Spring Security OAuth2, there's a huge quality breakdown. In this (probably series) of blogposts, 
I'll try to sum up the good, the bad, the evil and why I ended up completely dropping Spring Security OAuth2.

<!--more-->

## A new Project

It all began (as always) with a new project. There were some basic requirements to fulfill regarding the application 
and security architecture:

* Angular SPA with a Spring Boot backend
* Users should be able to log in using GitHub or GitLab
* Backend should map authenticated users to DB entities
* Backend communication should not be stateful (no session)
* OAuth tokens must be persisted b.c. used for server side API calls

Of course the first thought on this was:

> Easy peasy, Spring Security Oauth2 with custom success handler, done. &ndash; Me

But this was just wrong...

## The naive approach

Of course the first approach was just adding dependencies and some configuration and check if it works, like virtually 
always using spring boot. For the reference, we're doing Spring Boot `2.2.1.RELEASE` with Kotlin `1.3.60` on a JVM `11`:

Added dependencies:

```kotlin
implementation("org.springframework.security:spring-security-oauth2-client")
implementation("org.springframework.security:spring-security-oauth2-jose")
```

The whole OAuth2 config can theoretically be done using `application.yml` configuration. Btw. if you are looking to use 
GitLab as OAuth provider with Spring, the following config works at the time of writing this post:
```yml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            provider: github
            clientId: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            clientSecret: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            clientName: GitHub
            scope:
              - user:email
              - read:user
            # Custom attributes only parsed by com.example.shared.config.OauthExtraConfig:
            appId: 99999
            signingKeyPath: key/example-dev.der
          gitlab:
            provider: gitlab
            clientId: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            clientSecret: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
            clientName: GitLab
            authorizationGrantType: authorization_code
            redirectUri: "{baseUrl}/{action}/oauth2/code/{registrationId}"
            scope:
              - api
              - read_user
              - openid
              - profile
              - email
        provider:
          github:
            userNameAttribute: login
            # Default to predefined GitHub provider
            # See org.springframework.security.config.oauth2.client.CommonOAuth2Provider
          gitlab:
            authorizationUri: https://gitlab.com/oauth/authorize
            tokenUri: https://gitlab.com/oauth/token
            userInfoUri: https://gitlab.com/oauth/userinfo
            jwkSetUri: https://gitlab.com/oauth/discovery/keys
            userNameAttribute: nickname
```

As already the last step, we need to add some lines to security config:

```kotlin
override fun configure(http: HttpSecurity) {
  http

    // request auth
    .authorizeRequests()
      .antMatchers(
        "/login/**",
        "/oauth2/**",
      ).permitAll()

      // default
      .anyRequest()
        .authenticated()

    .and()

    // disable form login
    .formLogin()
      .disable()

    // enable oauth2 login
    .oauth2Login()
}
```

After some GitLab documentation reading and probably less than 30 mins of coding, this "works" out of the box.

Only some small changes are still to do:

- Make auth flow stateless (default is based on an http session)
- Map users to the database and store tokens to access GitHub / GitLab
- Issue own JWT tokens for Angular SPA instead of forwarding GitHub / GitLab tokens

These three *little* changes took me almost a week and finally forced me to give up Spring Security OAuth and implement 
stuff by hand.

## Things i'd like like to have known before

- Not all providers use the same standards. E.g. GitHub uses [Oauth2](https://oauth.net/2/) while GitLab uses 
[OIDC](https://openid.net/connect/).
- GitHub only sends the public email address within the user-info payload. If a users email is not public, you'll need 
to make an authorized API call to get the email addresses.
- GitHub does not provide long-lived auth-tokens and also now renew-tokens for "GitHub Apps". You'll need to 
re-authenticate the user on a very regular basis, or use the GitHub App JWT (installation) authentication which is 
experimental by now.
- The whole spring implementation depends (hardcoded) on the `state` parameter  which is passed along the auth flow, 
despite it's not required in specification: https://tools.ietf.org/html/rfc6749#section-4.1.1

Next post will be about how to (try to) make the auth flow stateless.