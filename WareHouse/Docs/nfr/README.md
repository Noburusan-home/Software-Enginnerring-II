# Non-Functional Requirements (quality attributes)

App-wide **how good** bars for warehouse operations. Feature specs still authorize *what* to build.

| Artifact | Question |
|----------|----------|
| [Feature specs](../../features/) | What must the product *do*? |
| **This folder** | What quality bars apply *across* the product? |
| [ADRs](../adr/README.md) | *Why* this approach for a quality? |

## Files

| File | Contents |
|------|----------|
| [quality-attributes.md](./quality-attributes.md) | Attribute table (Target, Approach, Verify, Status, Links) |

## When to update

| When | Action |
|------|--------|
| New app-wide quality bar | Update [quality-attributes.md](./quality-attributes.md) |
| Choose *how* to meet a bar | Write or update an [ADR](../adr/README.md) and link the row |
| Quality that only one feature needs | Put it in that feature’s **FR-00N** / **SC-00N** |

**Status values:** **Accepted** (in force) · **Accepted (minimal)** · **Deferred** (documented, not a CI gate) · **Out of scope** (do not build for this bar)
