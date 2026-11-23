# ADR-0005: Vuetify as the UI component library

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

Operational UIs need tables, dialogs, alerts, and form fields. Screen Requirements quote empty/error copy and CTAs. The course kit uses Vuetify.

## Decision

Use **Vuetify** for layout, forms, dialogs, and `<v-alert type="error">` error display. Primary labeled actions follow course CTA conventions (`oc-cta` when the kit style is applied). Icon-only actions require `aria-label`s.

## Consequences

### Positive

- Empty, loading, and error states are consistent across features.
- Labels in Gherkin can match Vuetify controls.

### Negative / tradeoffs

- Theming is a later polish concern (NFR Usability is Deferred except quoted copy).

## Alternatives considered

| Option | Why not |
|--------|---------|
| **Unstyled HTML only** | Slower to match Screen Requirements |
| **A different component library** | Diverges from OC CS Speckit |

## Related artifacts

- Feature **Screen Requirements**
- NFR **Usability** / **Accessibility**
