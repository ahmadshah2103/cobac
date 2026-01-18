# CoBAC Architecture

**Author:** Ahmad Shah  
**Last Updated:** January 2026

## Core Principles (Non-Negotiable)

These principles define CoBAC and cannot be changed without a major version bump:

1. **Pre-Retrieval Determinism**: Authorization MUST happen before vector retrieval
2. **Single Structural Authority**: Path is the ONLY source of truth for access control
3. **Passive Resources**: Documents NEVER store permissions
4. **Disjoint Forest Topology**: No cross-organization privilege escalation

## Design Decisions & Rationale

### Why Paths Instead of Graph-Based Auth?

**Decision:** Use hierarchical paths (`/org/proj/team`) instead of graph traversal (Zanzibar-style)

**Rationale:**
- Paths compile to O(1) vector DB filters
- Graph traversal requires separate system + post-filtering
- Paths are human-readable and debuggable
- 95% of organizations are hierarchical, not graph-like

**Trade-off:** Cannot model complex non-hierarchical relationships (deferred to v2)

### Why Boolean Inheritance Instead of lquery Patterns?

**Decision:** Role assignments use `should_inherit: boolean` flag

**Rationale:**
- Easier for non-technical users to understand
- UI-friendly (checkbox instead of pattern syntax)
- Compiles to efficient SQL predicates
- 80% of use cases are simple inheritance

**Trade-off:** Less expressive than full lquery (advanced users can use raw patterns)

### Why LTREE as Default But Support String Paths?

**Decision:** Recommend LTREE, support string paths for compatibility

**Rationale:**
- LTREE: 5-10x faster prefix queries, native indexing
- String paths: Work on any SQL database
- Performance difference negligible for ≤5 levels (95% of orgs)

**Migration Path:** Start with strings, upgrade to LTREE as needed

## System Components

[Include your technical architecture diagrams here]

## Security Invariants

These MUST hold for any CoBAC implementation:
```typescript
// Invariant 1: Disjoint Forest Isolation
function checkForestIsolation(assignment: RoleAssignment): boolean {
  const orgRoot = getOrgRoot(assignment.org_id);
  return isDescendant(assignment.path, orgRoot);
}

// Invariant 2: Pre-Retrieval Filtering
function vectorSearch(query: Vector, user: User): Chunk[] {
  const authorizedPaths = compileUserPaths(user); // MUST happen first
  return vectorDB.search(query, { filter: { path: authorizedPaths } });
}

// Invariant 3: Passive Resources
// Documents NEVER have a `permissions` field
interface Document {
  id: string;
  content: string;
  path: string; // Location-based authority
  // ❌ NO: permissions: Permission[]
}
```

## Extension Points

Where future contributors can add features:

1. **Database Adapters** (`/adapters/database/*.ts`)
2. **Vector DB Adapters** (`/adapters/vector/*.ts`)
3. **Metadata Generators** (for DBs without prefix support)
4. **Caching Strategies** (optional performance optimization)

## What NOT to Change

- Core authorization resolution algorithm
- Security invariants
- Path-based enforcement model
- Pre-retrieval requirement

Changes to these require RFC + Lead Maintainer approval.
