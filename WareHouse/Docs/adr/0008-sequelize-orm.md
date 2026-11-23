# ADR-0008: Sequelize as the Node ORM

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

Express needs a way to map MySQL tables from feature **Initial Data Model** sections without hand-writing SQL for every CRUD path. The course kit uses Sequelize.

## Decision

Use **Sequelize** models and associations (`hasMany` / `belongsTo`) to implement feature tables. Associations are declared in the data-model section of each feature. Queries stay parameterized (no string-concatenated SQL).

## Consequences

### Positive

- Tables in the spec map to models with little translation.
- FKs and actor ids are explicit.

### Negative / tradeoffs

- ORM learning curve; complex reports may later need raw queries (out of scope now).

## Alternatives considered

| Option | Why not |
|--------|---------|
| **Raw `mysql2` only** | More boilerplate vs Speckit |
| **Prisma** | Different kit than the course Todo example |

## Related artifacts

- [ADR-0003](./0003-mysql-relational-database.md)
- Feature **Initial Data Model**
