# Quality attributes

App-wide non-functional targets for the **warehouse operations** application.

**Policy:** Feature specs say *what* to build. This table says *how good* the system should be. Only **Accepted** rows (and feature **FR-00N** / **SC-00N**) constrain implementation. **Deferred** is documented but not a CI gate. **Out of scope** means do not build for that bar.

## Column meanings

| Column | Meaning |
|--------|---------|
| **Attribute** | Quality characteristic (“ility”) |
| **Target** | Measure — prefer a number |
| **Approach** | How the bar is met or why it is limited |
| **How we verify** | Tests, manual check, or N/A |
| **Status** | Accepted / Accepted (minimal) / Deferred / Out of scope |
| **Links** | ADR or feature pointer |

## Status values

| Status | Meaning |
|--------|---------|
| **Accepted** | In force; do not regress |
| **Accepted (minimal)** | Thin bar only — do not expand into a platform |
| **Deferred** | Written for the class; implement only if a feature or instructor requires it |
| **Out of scope** | Explicit non-goal |

---

Update this table when a bar changes. Feature-local bars stay in that feature’s **Functional Requirements** and **Success Criteria**.

| Attribute | Target | Approach | How we verify | Status | Links |
|-----------|--------|----------|---------------|--------|-------|
| **Security** | **100%** of `/warehouse/*` operational routes require auth; unauthenticated → **401**; manager-only actions by employees → **403**; unknown id → **404**; **0** trusted `createdBy` from the client | Layered API enforcement; roles `worker` / `manager` | Gherkin + API tests when implemented | Accepted | [ADR-0002](../adr/0002-security-and-role-architecture.md), [design.md](../../features/design.md) |
| **Data integrity** | **100%** of document lines that name a product include a **SKU** (unless unidentified return/receiving flags apply); quantities that represent physical counts are integers **> 0** unless a feature allows zero; FKs on line tables | MySQL + Sequelize; SKU as business key | Schema in feature Initial Data Models; Gherkin validation scenarios | Accepted | [ADR-0003](../adr/0003-mysql-relational-database.md), [ADR-0009](../adr/0009-sku-product-identity.md) |
| **Inventory safety** | **0** confirmed unusable / expired / uninspected-return units in **available** inventory used for shipping | Available qty excludes damaged, expired lots, and returns awaiting inspection | Gherkin in Features 2, 7, 8, 9 | Accepted | [feature-2](../../features/feature-2-damaged-product-handling.md), [feature-7](../../features/feature-7-product-returns.md), [feature-8](../../features/feature-8-receiving-inspection.md), [feature-9](../../features/feature-9-expiration-shelf-life.md) |
| **Auditability** | **100%** of create/report/count/move records store the acting employee from `req.user.id` | Server sets actor fields; ignore body `createdBy` | Gherkin actor fields; API tests | Accepted | [ADR-0001](../adr/0001-client-server-warehouse-architecture.md), [ADR-0002](../adr/0002-security-and-role-architecture.md) |
| **Reliability** | Failed writes return HTTP **4xx/5xx** with `{ "message" }` — never empty **200** | Single-process Express; no HA/retry layer | API tests on validation paths | Deferred | [ADR-0006](../adr/0006-node-express-api.md), [ADR-0007](../adr/0007-rest-json-api.md) |
| **Availability** | Local demo uptime during lab; **no** multi-region SLA | Single-node MySQL + API | N/A | Out of scope | [ADR-0001](../adr/0001-client-server-warehouse-architecture.md) |
| **Performance** | p95 API latency **&lt; 200 ms** local; primary warehouse screen first paint **&lt; 2 s** on a typical laptop | No load-test gate in CI yet | Manual / future timed tests | Deferred | — |
| **Scalability** | Correct for a **single warehouse** classroom demo; **no** horizontal scale design | Shared warehouse tenancy, not multi-site SaaS | Ownership/role tests; manual | Out of scope | [ADR-0001](../adr/0001-client-server-warehouse-architecture.md) |
| **Observability** | Unhandled server errors logged; HTTP access logged | Console or rotating file logs when backend exists | Logs in local runs | Accepted (minimal) | [ADR-0006](../adr/0006-node-express-api.md) |
| **Usability** | Quoted Screen Requirement labels match UI **100%**; empty/error states present on list views; employee can complete one primary floor task (e.g. create order or record count) in **≤ 3 minutes** once signed in | Vuetify + Screen Requirements | Manual walkthrough; later Vitest for labels | Deferred | [ADR-0005](../adr/0005-vuetify-ui-library.md), feature **Screen Requirements** |
| **Accessibility (a11y)** | Primary flows keyboard-reachable; **0** unlabeled icon-only actions on primary screens | Vuetify semantic components; `aria-label` on icon actions | Manual / future a11y tests | Deferred | [ADR-0005](../adr/0005-vuetify-ui-library.md) |
| **Internationalization (i18n)** | **1** locale (en-US); **0** translation catalogs | No i18n framework | N/A | Out of scope | — |
| **Maintainability** | **100%** of Gherkin scenarios have tests before merge; feature files stay the source of truth | Spec-driven development; `npm test` when code exists | Merge Definition of Done | Accepted | [features/README.md](../../features/README.md) |

---

## Feature-local NFRs

If only one feature needs a bar, put it in that feature’s **Functional Requirements** or **Success Criteria** (and Gherkin). Examples already in specs: duplicate P.O. number (Feature 1), location capacity (Feature 10), 30-day expiration warning (Feature 9).

## Changing a bar

1. Edit this table.
2. If **Approach** changes → update or add an ADR and put it in **Links**.
3. Promoting **Deferred** → **Accepted** requires a real verification method.
