# Architecture Decision Records (ADRs)

Durable **why** decisions for the warehouse application. Feature specs still say *what* to build.

**NFR table:** [../nfr/quality-attributes.md](../nfr/quality-attributes.md)

## When to write an ADR

| Situation | Write an ADR? |
|-----------|---------------|
| New product behavior for one feature | No — use `features/feature-N-*.md` |
| Significant stack or architecture choice | **Yes** |
| Security, roles, or data-isolation model | **Yes** |
| Database engine / API style | **Yes** |

## File naming

`NNNN-short-kebab-title.md` — four-digit number, sequential, never reused.

**Status:** `Proposed` | `Accepted` | `Deprecated` | `Superseded by ADR-NNNN`

## Template

```markdown
# ADR-NNNN: Short title

**Status:** Proposed | Accepted
**Date:** YYYY-MM-DD
**Deciders:** team / role names

## Context
## Decision
## Consequences
### Positive
### Negative / tradeoffs
## Alternatives considered
## Related artifacts
```

## Index

| ADR | Title | Status |
|-----|-------|--------|
| [0001](./0001-client-server-warehouse-architecture.md) | Client–server architecture for shared warehouse data | Accepted |
| [0002](./0002-security-and-role-architecture.md) | Layered security and employee/manager roles | Accepted |
| [0003](./0003-mysql-relational-database.md) | MySQL relational database | Accepted |
| [0004](./0004-vue-frontend-framework.md) | Vue 3 as the frontend framework | Accepted |
| [0005](./0005-vuetify-ui-library.md) | Vuetify as the UI component library | Accepted |
| [0006](./0006-node-express-api.md) | Node.js and Express as the API runtime | Accepted |
| [0007](./0007-rest-json-api.md) | REST and flat JSON as the API style | Accepted |
| [0008](./0008-sequelize-orm.md) | Sequelize as the Node ORM | Accepted |
| [0009](./0009-sku-product-identity.md) | SKU as the product business key | Accepted |
