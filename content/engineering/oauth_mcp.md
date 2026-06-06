---
title: Authorization on MCP servers
type: page
description: How to authorize MCP servers against a multi-tenant database?
topic: AI, MCP
---

## Introduction

This will be a fairly simple and short blog post about using an MCP server together with a database. The code can be found [here](https://github.com/memgraph/ai-toolkit/pull/206) in the form of a pull request.

## Problem description

I am an engineer working on a dog-food project of using the Memgraph database within the company. I (as an admin in this case) deployed an MCP server and wanted every employee to be able to "get the help" from a single instance of that server, but scoped to their own tenant. I assume you're familiar with multi-tenancy (each user has access to one or more tenants). As an admin, I also want to be able to query multiple tenants myself.

The problem was that our MCP server (based on FastMCP) could only be used with a single tenant, specified at startup. That was unacceptable to me, because it meant that whenever someone wanted to use the MCP server, I would have to manually restart it and reconfigure which tenant it pointed at.

## Solution

The solution is to use JWT tokens together with the Keycloak authorization server. Based on the JWT claims, the MCP server routes each request to a different logical database.

I used Keycloak as the authentication provider. After starting it (I used its Bitnami chart and deployed it through ArgoCD), you create a set of users and specify which user has access to which database. When a user starts using the MCP server, Keycloak intercepts the request and requires the user to authenticate via OAuth with username and password. Once authenticated, the `tenants` array claim is extracted from the JWT and, based on it, a `SessionAuth` object is created. The flow sounds simple, and it actually is — thanks to a couple of really important RFC documents:

- **RFC 9728**: Describes how MCP clients find out which authorization server to use (Keycloak in our case).
- **RFC 8414**: Defines a standard JSON format that OAuth 2.0 clients use to automatically discover and retrieve configuration details about an authorization server.

Based on these documents, we expose a few well-known paths: `/.well-known/oauth-protected-resource`, `/.well-known/openid-configuration`, and `/register`.

One catch: Claude Code forces Dynamic Client Registration even when you already have a pre-registered `clientId`. To work around that, I made the MCP server lie a little to such clients — it returns the same pre-registered `client_id` for every DCR request. Differentiation between users then happens per request, from the JWT claims, rather than per client.

On the dependency side, this added `pyjwt[crypto]` and `uvicorn` at runtime, with `cryptography` used in tests to generate self-signed JWTs.

This is a first iteration. Supporting full Dynamic Client Registration and additional authorization providers is left as future work, but the single-client-id approach was enough to get true multi-tenant access working without ever restarting the server.
