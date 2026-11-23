# ADR-0006: Node.js and Express as the API runtime

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

The SPA needs HTTP endpoints for warehouse documents. The course kit uses Node.js + Express (ES modules) with Jest + supertest.

## Decision

Implement the API as a **Node.js** process using **Express**. Routes live under `/warehouse/`. Middleware order: authenticate → validate → controller. One process is enough for the classroom draft (no cluster/HA).

## Consequences

### Positive

- Same test style as Speckit (supertest against routes).
- Clear place for role checks ([ADR-0002](./0002-security-and-role-architecture.md)).

### Negative / tradeoffs

- Single-node availability (NFR Availability is Out of scope).

## Alternatives considered

| Option | Why not |
|--------|---------|
| **Java Spring / .NET** | Off the course Node kit |
| **Serverless-only** | Harder local Gherkin/API tests for this class |

## Related artifacts

- [ADR-0007](./0007-rest-json-api.md)
- Feature **API Requirements**
