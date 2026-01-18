# A. Core Invariants (Already Frozen)

These define the **foundational architecture** and should be treated as non-negotiable.

---

## **A1 — Single Structural Authority**

- The filesystem-like hierarchy (path tree) is the **only** structural authority.
- Organization root defines isolation.
- All sensitivity is encoded in **location**, not documents.

---

## **A2 — Documents Are Passive Resources**

- Documents do **not**:
  - Store permissions
  - Store roles
  - Store user allowlists
  - Participate in authorization logic

- Documents only have:
  - Content
  - Canonical path
  - Metadata unrelated to access control

---

## **A3 — Location-Scoped Role Assignments**

- Roles are assigned as:

```
(user, role, path)
```

- Roles are never implicitly global.
- Access is always evaluated relative to a resource’s path.

---

## **A4 — Authorization Precedes AI Context Assembly**

- Authorization is resolved **before**:
  - Vector retrieval
  - Chunk scoring
  - Prompt construction

- AI never evaluates permissions.

---

## **A5 — Pre-Filtering Is Mandatory**

- Vector search is always pre-filtered using authorization-derived constraints.
- Unauthorized chunks are never retrieved, logged, or scored.

---

## **A6 — Path-Based Enforcement**

- Authorization enforcement uses:
  - Path containment
  - Path prefix matching

- Vector store remains policy-agnostic.
- No role logic inside vector queries.

---

## **A7 — AI Is a Non-Privileged Consumer**

- AI only sees:
  - Pre-authorized
  - Pre-filtered
  - Minimal document chunks

- AI never sees:
  - Unauthorized content
  - Access metadata
  - Role or permission data

---

# B. Secondary Invariants (Now Clarified and Frozen)

These define **behavioral semantics** and edge handling.

---

## **B1 — Explicit Path Inheritance**

- Role assignments include:

```
should_inherit: boolean
```

- If `true`, role applies to all descendant paths.
- If `false`, role applies only at the exact path.
- Inheritance is **not encoded in path syntax**.

---

## **B2 — Union-Based Permission Resolution**

- If multiple roles apply:
  - Effective permissions are the **union** of all permissions.

- No deny semantics.
- No precedence rules.
- “More access wins.”

---

## **B3 — Canonical Path Guarantee**

- A document has **exactly one canonical path**.
- Identical content in multiple locations is represented as **distinct documents**.
- No aliases, shortcuts, or soft links in v1.

---

## **B4 — System-Enforced Permissions, Org-Defined Roles**

- System understands:

```
(resource, action, path)
```

- System does **not** understand roles.
- Roles are organization-defined bundles of permissions.
- Authorization engine operates purely on permissions.

---

## **B5 — Per-Request Authorization Resolution**

- Authorization is evaluated on every request in v1.
- No caching is required initially.
- Caching is an optimization, not a semantic change.

---

## **B6 — Chunk-to-Document Relationship**

- Relationship is:

```
Document 1 ──▶ Chunks N
```

- Chunks belong to exactly one document.
- Chunks inherit the document’s canonical path.
- No chunk reuse across documents.

---

## **B7 — Deterministic Explainability**

- The system must be able to answer:

> “Why does user X have access to resource Y?”

- Explanation is derived via:
  - Role assignments
  - Paths
  - Inheritance rules

- No precomputed audit logs required in v1.

---

# Explicitly Deferred (Not Invariants Yet)

These are intentionally **out of scope** for v1:

- Deny rules / negative scopes
- Document-level clearance
- Explicit user allowlists
- Shared documents / soft links
- Role precedence logic
- Compliance-specific auditing
