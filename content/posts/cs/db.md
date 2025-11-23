---
title: 'Notes around database'
date: '2025-11-23'
categories: ["CS"]
tags: ["DB", "English Article"]
---

## Introduction

Random notes and summary of database.

## Connection pooling

Connection pooling is a technique to keep the reusable open database connections so that applications do not need to open and close a new connection for every request.

Why connections are expensive:

- Creating a connection requires TCP handshake
- The DB must authenticate and allocate memory/resources
- PostgreSQL forks a process per connection
- MySQL creates a thread per connection (context switching for thread is 5x faster than process)

### Connection Pool Size

**Formula (HikariCP recommendation):**

```text
connections = (core_count * 2) + 1
```

**General guidelines:**

- Small apps: 5-10 connections
- Medium apps: 10-20 connections
- Large apps: 20-50 connections

**Important considerations:**

- More connections ≠ better performance (causes context switching overhead and lock contention)
- Pool size should be based on CPU cores, not expected traffic
- PostgreSQL needs fewer connections than MySQL (processes vs threads)
- Total connections across all app instances must be < database `max_connections`

**Example calculation:**

```text
Database max_connections: 100
Application instances: 4
Reserve for admin: 20
Available: 80
Pool size per instance: 80 / 4 = 20 connections max
Recommended: Start with 10, monitor and tune
```

**Key metrics to monitor:**

- Connection wait time (increase pool if apps wait)
- Database CPU utilization (decrease pool if saturated)
- Active vs idle connections ratio
