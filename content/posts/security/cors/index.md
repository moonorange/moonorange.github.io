---
title: 'CORS'
date: '2023-08-07'
categories: ["Security", "HTTP cookies", "CORS"]
tags: ["Web", "HTTP cookies", "English Article"]
---

## Introduction

This post summarizes what I have learned when I encountered issues related to CORS.

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

Registrable domain portion consists of an entry of [Public Suffix List](https://publicsuffix.org/list/) or top level domains and plus the portion of the domain name just before it, so example.com is a registrable domain name as `com` part is a top level domain and `example` is the immediate preceding portion.

Let's examine examples to gain a clearer understanding of what constitutes the same site:

They are considered the same site due to their matching registrable domain and scheme:

- <http://main.example.com:80>
- <http://sub.example.com:80>

They are also considered the same site because the port is irrelevant:

- <https://example.com:8080>
- <https://example.com>

## Strict-Transport-Security

Slightly deviating from the main topic, let me explain the concept of HTTP Strict-Transport-Security (HSTS) as well.

The `HTTP Strict-Transport-Security` response header, commonly known as `HSTS`, is employed to compel browsers to exclusively accept HTTPS connections.

By utilizing this header, browsers automatically redirect HTTP connections to HTTPS.

This header is useful, and should be correctly configured in the production environment to enforce secure connections, However, there are some cases where you might not want these redirects to occur.

In Chrome, you can manually remove the domain's security policy using the following steps:

- Navigate to chrome://net-internals/#hsts
- Remove domain security policies (e.g., 'example.com')

This ensures that redirection won't occur again until you access the deleted domain through HTTPS.

## References

<https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy>

<https://developer.mozilla.org/en-US/docs/Glossary/Origin>
