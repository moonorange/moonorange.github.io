---
title: 'CORS'
date: '2023-08-07'
categories: ["Security", "HTTP cookies", "CORS"]
tags: ["Web", "HTTP cookies", "English Article"]
---

## Introduction

This post summarizes what I have learned when I encountered issues related to CORS and cookies.

## Understanding Domains(hostname), Origins, and Sites

It's crucial to comprehend these concepts when dealing with CORS-related problems.

### Origin

An example of a web content's origin is <http://example.com:80>.
An origin is comprised of the `domain` (hostname), `port`, and `scheme`.

In the case above, `http` is the scheme, `example.com` is the domain, and `80` is the port.

Two URLs are considered to have the same origin only when all three of these components are identical.

### Site

A site, on the other hand, is a collection of web pages served by the same domain.

Two URLs are deemed to belong to the same site if the `registrable domain` portion of the domain (and, in some cases, the scheme) is the same.

The `registrable domain` consists of an entry of [Public Suffix List](https://publicsuffix.org/list/) or top level domains plus the portion of the domain name just before it.

For instance, 'example.com' is a registrable domain name as `com` part is a top level domain and `example` is the immediate preceding portion.

Let's examine examples to gain a clearer understanding of what constitutes the same site:

They are considered the same site due to their matching registrable domain and scheme:

- <http://main.example.com:80>
- <http://sub.example.com:80>

They are also considered the same site because the port is irrelevant:

- <https://example.com:8080>
- <https://example.com>

They are regarded as the same site in the context of a scheme-less same site scenario, but as different sites in the context of a scheme-ful same site scenario:

- <http://example.com>
- <https://example.com>


## Set-Cookie Behavior

The behavior of the Set-Cookie header is influenced by the SameSite policy for the cookie attributes.

For instance, if the SameSite policy is set to Lax or Strict, the cookie will not be included in cross-site requests.

So, if you intend to transmit the cookie, it's necessary for both the client and server to be considered the same site.

Imagine a scenario where you want to send a cookie from a client hosted at '<http://localhost:3000>' to an API hosted on <http://api.example.com:8080>.

You can achieve this locally by adding the domain (hostname) to your /etc/hosts file:

```text
127.0.0.1 example.com
```

Now, when you access <http://example.com:3000>, you can successfully send the cookie to the API server since they are treated as part of the same site.

## Strict-Transport-Security

Slightly deviating from the main topic, let me explain the concept of HTTP Strict-Transport-Security (HSTS) as well.

The `HTTP Strict-Transport-Security` response header, commonly known as `HSTS`, is employed to compel browsers to exclusively accept HTTPS connections.

By utilizing this header, browsers automatically redirect HTTP connections to HTTPS.

This header is useful, and should be correctly configured in the production environment to enforce secure connections, However, there might be certain situations where these redirects might not be desired.

Consider the example presented in the 'Set-Cookie Behavior' section.

When you access <http://example.com:3000> and the HTTP Strict-Transport-Security header is present, the access will automatically be redirected to <https://example.com:3000>.

Thus, the cookie won't be sent because <https://example.com:3000> and <http://api.example.com:8080> are scheme-ful different sites now.

In Chrome, you can manually remove the domain's security policy using the following steps:

- Navigate to `chrome://net-internals/#hsts`
- Remove domain security policies (e.g., 'example.com')

This enables that redirection won't occur again until you access the deleted domain through HTTPS.

## References

<https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy>

<https://developer.mozilla.org/en-US/docs/Glossary/Origin>
