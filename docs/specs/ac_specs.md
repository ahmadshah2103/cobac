# Authorization Resolution Specification (v1)

## 1. Purpose and Scope

This document defines the **authorization model and resolution process** for the system. It specifies how access to resources (documents and document chunks) is determined for a given user, and how this authorization integrates with AI retrieval.

This specification is **normative** for v1 and should be treated as frozen unless explicitly revised.

---

## 2. Core Design Principles

1. **Single Authority**: The filesystem-like hierarchy (paths) is the sole structural authority for access control.
2. **Least Privilege**: Access is granted only through explicit role assignments scoped to locations.
3. **Passive Resources**: Documents and chunks do not participate in authorization decisions.
4. **AI Isolation**: AI systems never evaluate permissions and never see unauthorized content.
5. **Flexible Topology**: Nodes represent generic folders/locations, allowing arbitrary depth. Organizational semantics (department, project, team, squad) may be stored in hierarchy metadata (e.g., node types) but are **not used in authorization resolution**.

---

## 3. Conceptual Model

### 3.1 Hierarchy

- The system maintains a hierarchical, filesystem-like path structure.
- Each organization has a single root path.
- Nodes (folders/locations) are generic and can represent projects, departments, teams, squads, or other subunits.
- All resources exist at exactly one canonical path.

Example:

```
/org/{org_id}/projects/{project_id}/docs/{doc_id}
```

Nodes can be inserted at any depth, supporting flexible organization structures.

---

### 3.2 Resources

- **Document**: A logical container for content, identified by a canonical path.
- **Chunk**: A segment of a document used for retrieval and AI context.

Rules:

- A document has exactly one canonical path.
- A chunk belongs to exactly one document.
- A chunk always inherits the canonical path of its document.

---

### 3.3 Actions and Permissions

- The system defines a fixed set of **actions** (e.g., `read`, `write`, `update`, `delete`, `query`).
- A **permission** is defined as:

```
(resource, action)
```

Examples:

- `(document, read)`
- `(document, write)`
- `(chunk, query)`

The system enforces permissions; it does not understand roles.

---

### 3.4 Roles

- Roles are **organization-defined** constructs.
- A role is a named bundle of permissions.
- The system does not interpret role semantics.

Example:

```
Role: "Project Reader"
Permissions:
- (document, read)
- (chunk, query)
```

---

### 3.5 Role Assignments

A role assignment is defined as:

```
(user, role, path, should_inherit)
```

Where:

- `path` is the location at which the role applies
- `should_inherit` determines whether the role applies to descendant paths

Rules:

- Roles are never implicitly global.
- Inheritance is controlled explicitly by `should_inherit`.
- Nodes are generic; the same table is used to represent projects, teams, departments, squads, or personal subfolders.

---

## 4. Authorization Resolution Semantics

### 4.1 Applicable Role Resolution

Given:

- **U**: a user (principal) requesting access
- **P**: the canonical path of the target resource
- **R**: a role assigned to a user
- **A**: the anchor path at which the role is assigned
- **I**: a boolean inheritance flag indicating downward propagation

A role assignment **(U, R, A, I)** yields **Applicable(U, R, P, A, I) = true** if and only if:

- **P == A**, or
- **I == true** and **P** is a descendant of **A**

Formally:

```
Applicable(U, R, P, A, I) ⇔ (P = A) ∨ (I ∧ Descendant(P, A))
```

Where **Descendant(P, A)** denotes that **P** lies strictly below **A** in the resource path hierarchy.

---

### 4.2 Permission Expansion

- For all applicable roles, expand roles into their permissions.
- Effective permissions are the **union** of all expanded permissions.

There are:

- No deny rules
- No precedence rules
- No subtraction semantics

---

### 4.3 Access Decision

An access request is allowed if:

```
requested_permission ∈ effective_permissions
```

Otherwise, access is denied.

---

## 5. AI Retrieval and Authorization Integration

### 5.1 Authorization Boundary

Authorization must be fully resolved **before**:

- Vector retrieval
- Chunk scoring
- Prompt construction

AI systems never participate in authorization logic.

---

### 5.2 Pre-Filtered Retrieval

AI retrieval uses **pre-filtering** based on authorized paths.

Process:

1. Resolve user’s applicable role assignments
2. Compute authorized path prefixes
3. Inject path-based filters into vector search
4. Retrieve only authorized chunks

Unauthorized chunks must:

- Never be retrieved
- Never be logged
- Never be scored
- Never be sent to the model

---

### 5.3 Vector Store Constraints

- Vector store metadata includes:
  - `chunk_id`
  - `document_id`
  - `path`

- No role or permission logic exists in the vector store.
- Paths support materialized paths (e.g., Postgres LTREE) to efficiently support pre-filtering and inheritance queries.

---

## 6. Explainability

The system must be able to deterministically answer:

> "Why does user U have access to resource R?"

This explanation is derived from:

- Role assignments
- Path containment rules
- Permission expansion

No precomputed audit logs are required in v1.

---

## 7. Explicit Non-Goals (v1)

The following are intentionally out of scope:

- Document-level ACLs
- Deny rules or negative scopes
- Document clearance or sensitivity flags
- Shared documents or soft links
- Role precedence or depth-based overrides
- Compliance or regulatory audit frameworks

---

## 8. Stability Guarantee

All rules in this specification are considered **stable for v1**.

Future versions may extend this model, but must preserve:

- Path as the primary authority
- Pre-filtered authorization
- Passive resources
- AI isolation
- Flexible, generic topology supporting arbitrary nodes (projects, teams, departments, squads, personal folders, etc.)

---

## Appendix A: Authorization Invariants Checklist

This appendix provides a quick-reference checklist of the core and secondary invariants. For detailed explanations, see the relevant sections above.

### A. Core Invariants

| ID  | Invariant                                                         | Reference  |
| --- | ----------------------------------------------------------------- | ---------- |
| A1  | Path tree is the **only** structural authority                    | §2.1, §3.1 |
| A2  | Documents are **passive** — no permissions, roles, or allowlists  | §2.3, §3.2 |
| A3  | Role assignments: `(user, role, path, should_inherit)`            | §3.5       |
| A4  | Authorization resolved **before** vector retrieval                | §5.1       |
| A5  | Pre-filtering is **mandatory** — no post-hoc filtering            | §5.2       |
| A6  | Enforcement via path containment; vector store is policy-agnostic | §5.3       |
| A7  | AI sees only pre-authorized, pre-filtered chunks                  | §5.1, §5.2 |

### B. Secondary Invariants

| ID  | Invariant                                                     | Reference  |
| --- | ------------------------------------------------------------- | ---------- |
| B1  | `should_inherit` controls descendant propagation              | §3.5, §4.1 |
| B2  | Effective permissions = **union** of all applicable roles     | §4.2       |
| B3  | Documents have **exactly one canonical path**                 | §3.2       |
| B4  | System enforces permissions; roles are org-defined bundles    | §3.3, §3.4 |
| B5  | Authorization evaluated per-request (caching is optimization) | §4.3       |
| B6  | Chunks belong to exactly one document; inherit its path       | §3.2       |
| B7  | System can deterministically explain access decisions         | §6         |

### Explicitly Deferred (v1)

- Deny rules / negative scopes
- Document-level clearance
- Explicit user allowlists
- Shared documents / soft links
- Role precedence logic
- Compliance-specific auditing
