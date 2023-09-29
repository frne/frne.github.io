---
title: "Securing closed systems: Caveats of using plain OAUTH flows and how to solve them"
description: "While OIDC and OAUTH are well-known standards, they don't fit every purpose \"out of the box\". In businesses with special requirements, like health-care, non-functional requirements to auth can be challenging."
image: "images/post/2023-09-11-securing-closed-systems-caveats-of-plain-oauth-flows-how-to-solve/header.png"
imgcaption: "Photo by [MART PRODUCTION](https://www.pexels.com/photo/photo-of-doctor-looking-deeply-unto-the-screen-7088530/) from [Pexels](https://www.pexels.com/)"
date: 2023-09-11T11:10:00+02:00
author: "Frank Neff"
tags: ["oauth", "oidc", "security", "authentication", "authorization", "architecture"]
categories: ["Architecture", "Security"]
draft: false
---

While OIDC and OAUTH are well-known standards, they don't fit every purpose "out of the box." In businesses with **special
regulations** like banking, health care, etc., non-functional requirements to auth can be challenging. Different 
solutions and ways were evaluated to create a new identity provider for a medical network. The first approach was "just" 
using simple **OAUTH** by its most famous **Authorization Code Flow**. Of course, it failed fast, and I'll show why and 
how we solved it in this post.

<!--more-->

## The Challenge

While re-architecting and (partially) rebuilding a healthcare-related platform, the identity layer was one of the central 
components we wanted to work on first. While an identity provider has always been there, the platform developed the need 
to add features and provide (token-based) access to third parties. As the project was an integration case with dozens of 
clients providing and consuming data, not only security and performance but also adoption by implementers was crucial. The 
most straightforward solution came up reasonably quickly: 

> "Let's just do OAUTH! It's easy & secure, everyone knows it."

After architecting and implementing a PoC, we tested auth & integration with a client and gave it an internal security
audit. As the solution should be "straightforward" to all involved parties, we used Implicit Flow for our own frontends
and Authorization Code Flow for all backend clients, including external ones.

### Wrong Assumptions

While this worked for all internal needs, we immediately faced a massive architectural flaw.
We implicitly anticipated that all clients:

- Are browser-based tools
- Are hosted on a central server (SaaS)

{{< image 
  src="/diagrams/2023-09-11-securing-closed-systems-caveats-of-plain-oauth-flows-how-to-solve/high-level-architecture-assumption.svg" 
  alt="High-level architecture diagram showing wrong assumptions on auth-flow and deployment model"
  title="Assumed high-level architecture of the authentication flow"
  caption="Assumed high-level architecture of the authentication flow ([source](/diagrams/2023-09-11-securing-closed-systems-caveats-of-plain-oauth-flows-how-to-solve/high-level-architecture-assumption.drawio))" >}}

Of course, both assumptions were dead wrong. On the one hand, many clients were closed "fat clients" running locally on 
PCs. While they had an internet connection and could communicate with HTTP APIs, they struggled to open browser windows / 
redirect clients to web endpoints. But there was another, way more significant challenge, which lies in the deployment 
model of many healthcare SaaS providers:

### Nowhere to Redirect

Especially in high-security environments like health care, where one needs to process highly protected patient data, tenant
isolation is often done on a physical or instance level. This is contrary to SaaS, where we mostly have one big application 
that supports multi-tenancy and is centrally hosted. The majority of healthcare software suites, even if they're 
web-based, are hosted within the healthcare facility or an IT partner, using one isolated instance per tenant on their 
own machine with their dedicated database.

{{< image
  src="/diagrams/2023-09-11-securing-closed-systems-caveats-of-plain-oauth-flows-how-to-solve/high-level-architecture-reality.svg"
  alt="High-level architecture diagram showing realistic deployment model and caveats"
  title="Assumed high-level architecture of the authentication flow"
  caption="Reality high-level architecture & caveats ([source](/diagrams/2023-09-11-securing-closed-systems-caveats-of-plain-oauth-flows-how-to-solve/high-level-architecture-reality.drawio))" >}}

In this area (A) we have to deal with all sorts of security mechanisms like network access control, firewalling, 
isolation, IDS/IPS, and many more, which we cannot anticipate. Some minor, self-operated instances (C) even used a 
regular consumer DSL router with a dynamic IP. Finally, we needed to accept that those systems could talk to us, but we 
could not talk to them, which left us, at least for the very common flows, with a huge issue:

> We can not work with redirects or basically any mechanism which implies bidirectional communication.

## On OAUTH Flows

The latter implication becomes effective when considering how the OAUTH **Authorization Code Flow** is designed.
In a simple scenario, the unauthenticated user needs to complete the following steps to obtain an access token:

1. User gets redirected from the original application UI to a login screen on the identity provider
2. User logs in (and accepts scopes needed by the original application)
3. User is redirected back to the original application URL, provided by the `redirect_uri` parameter supplied with the first call
4. To that redirect URI, a short-lived auth token is appended and with this supplied to the client
5. The client requests an access token by supplying the before-mentioned auth token, client id and secret to the IDP
6. The IDP issues the access token to the client

With the implications mentioned, this flow became impossible because the clients have no URI to redirect to. In fact, that
flow implies that all clients have a public URL to redirect to.

We looked at other well-known flows and came to the following conclusions:

- **Implicit Flow** doesn't work because it is for frontends only and also works with redirects (same problem)
- **Resource Owner Password Credentials Flow** would technically have been an option but could not be used for data
  security reasons. In fact, the users should not enter their credentials in any form but our own IDP by policy, which
  eliminated this option.
- **Client Credentials Flow** also couldn't be used because the client is not the resource owner and, therefore, potentially
  untrusted unless authorized by the resource owner by client consent.

### The OAuth 2.0 Device Code Flow

Have you ever set up a device like a smart tv, set-top box, etc., where you needed to log in using your smartphone or
browser and then enter a 4 to 6-digit code or scan a QR code that was shown on the device? That's a **Device Code Flow**.
It's actually designed to authenticate devices that are not fully capable of executing the other flows.

{{< image
  src="/images/post/2023-09-11-securing-closed-systems-caveats-of-plain-oauth-flows-how-to-solve/github-device-code-auth.png"
  alt="Screenshot of GitHubs device code auth screen"
  title="Screenshot of GitHubs device code auth screen"
  caption="GitHubs device code auth screen as used for GitHub CLI ([source](https://github.com/cli/oauth))"
  command="fill" class="img-fluid" >}}

By using this lesser-known OAUTH flow, we were actually able to work around the missing / distributed redirect URI
problem because all calls made are directed from the client to IDP, but not vice versa. Also, no redirects are used,
as explained in the following simplified scenario:

1. Client sends a **Device Authorization Request** to the IDP, containing the `client_id` and optionally a `scope`
2. IDP responds with a **Device Authorization Response**, containing `device_code`, `user_code` and `verification_uri`
3. The `user_code` and `verification_uri` need to be **presented to the user** in some way by the client
4. Finally, the client requests a **Device Access Token** from the IDP by sending `grant_type`, `device_code` and `client_id`
   to the IDP
5. The IDP responds either with a **Device Access Token** or one of the errors `authorization_pending`, `slow_down`,
   `access_denied` or `expired_token`, which are specific for device code auth. On `authorization_pending`, the user has
   not yet completed the flow; steps 4 & 5 should be repeated after some waiting time.

A more detailed explanation of the flow, incl. examples, can be found at the
[IETF RFC-8629 about the OAuth 2.0 Device Authorization Grant](https://datatracker.ietf.org/doc/html/rfc8628).

## Convenience Measures

For the convenience of the user, we added mechanisms to make the code auth (typing or copying a code) as painless as
possible. While some clients relied on the "hard way" because they could not open a browser, most could. Therefore, We 
have suggested that most clients open the URL with the code attached as a parameter directly in a browser instead of
instructing the user to use another device. Like that, no code needs to be typed or copied.

In addition, we enabled refresh tokens to be issued. Using those additionally granted, long-lived tokens and the client
secret, clients can extend or re-issue access tokens without redoing a full Device Code Flow. If
well-implemented, the user only needs to log in once using their browser and is authenticated automatically afterwords. 
From a useability perspective, this feels the same as an Authorization Code Flow but doesn't need a redirect URI and 
adds a convenient fallback solution for less tightly integrated clients.

### Added security with mTLS

To add an extra layer of security, mutual TLS by using client-side X509 certificates was evaluated and planned. As the
correct and secure implementation of mTLS can become complex to impossible for some clients, especially when dealing with
legacy software & and security mechanisms, the solution has been planned and rolled out as an additional measure and not as a hard
technical requirement.

The majority of all clients adopted the solution, including mTLS, or otherwise added different additional security measures on
their side.
