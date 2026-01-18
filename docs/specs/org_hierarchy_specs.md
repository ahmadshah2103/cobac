# Organizational Hierarchy Management Specification (v1)

## 1. Purpose and Scope

This specification defines how the **hierarchical organizational structure** is modeled, stored, and managed in the system. It establishes:

* Default node types and hierarchy
* Rules for parent-child relationships between nodes
* Customization mechanisms for organization admins

This document is **normative** for v1 and serves as a reference for system implementation, provisioning, and UI behavior.

---

## 2. Core Principles

1. **Single Root Authority**: Each organization has a single root node; all nodes exist under this root.
2. **Path-Based Hierarchy**: Nodes are arranged in a **filesystem-like hierarchy**; each node has a unique materialized path.
3. **Node Types**: Each node has a **type** (Project, Team, Department, Generic Folder). Types may carry **structural semantics** (allowed children, generic folder allowance).
4. **Admin-Controlled Semantics**: Only organization admins can define allowed node types and parent-child rules.
5. **Default Structure**: Every organization receives a pre-defined, valid hierarchy on creation.
6. **Authorization Independence**: Node types and hierarchy rules do **not impact access control**, which is path + role-based.

---

## 3. Node Model

### 3.1 Node Entity

| Field       | Description                                |
| ----------- | ------------------------------------------ |
| `id`        | Unique identifier for the node             |
| `org_id`    | Organization the node belongs to           |
| `name`      | Human-readable node name                   |
| `type_id`   | Foreign key to node_types table            |
| `parent_id` | Parent node reference (null for root)      |
| `path`      | Materialized path for hierarchy resolution |

* **Path** format: `/org/{org_id}/{node1}/{node2}/.../{node_n}`
* Every node belongs to exactly **one parent** (except root).

---

### 3.2 Node Types

| Field              | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `id`               | Unique identifier for the node type                          |
| `org_id`           | Organization this type belongs to                            |
| `name`             | Type name (Project, Team, Department, Generic Folder)        |
| `allow_generic`    | Boolean; whether generic folders are allowed under this type |
| `allowed_children` | Array of node_type IDs allowed as children                   |

* Node types **define valid hierarchy**, not access control.
* Default types are provided; admins may customize.

---

## 4. Default Organizational Hierarchy

| Parent Node Type  | Allowed Child Types  | Generic Folder Allowed | Notes                                                   |
| ----------------- | -------------------- | ---------------------- | ------------------------------------------------------- |
| Organization Root | Project, Department  | No                     | Projects and departments are top-level nodes            |
| Project           | Team, Generic Folder | Yes                    | Projects group multiple teams; generic folders optional |
| Team              | Generic Folder       | Yes                    | Teams organize resources into folders                   |
| Department        | Generic Folder       | Yes                    | Departments are functional divisions; may store folders |
| Generic Folder    | Generic Folder       | Yes                    | Optional nested folders                                 |

**Default Hierarchy Example**

```
Organization Root
├── Project
│   ├── Team
│   │   └── Generic Folder
│   └── Generic Folder
├── Department
│   └── Generic Folder
```

---

## 5. Organization Provisioning

1. **Organization Creation**:

   * Default node types and hierarchy are created automatically.
   * Root node is established.

2. **Admin Customization (Optional)**:

   * Admin may define additional node types.
   * Admin may adjust allowed parent-child relationships.
   * Admin may enable/disable generic folders for specific types.
   * Changes are persisted in the `node_types` table.

3. **Node Creation by Users**:

   * Users create nodes under allowed parents.
   * System validates node type against parent’s allowed children and `allow_generic` rules.
   * Invalid creations are rejected (hard enforcement) or warned (soft enforcement).

---

## 6. Node Operations

| Operation     | Rules / Behavior                                                     |
| ------------- | -------------------------------------------------------------------- |
| Create Node   | Must follow type constraints; path auto-generated; parent must exist |
| Rename Node   | Updates name; path updates propagated to descendants                 |
| Move Node     | Parent must allow new type; path updated recursively                 |
| Delete Node   | All descendant nodes deleted or reparented per policy                |
| List Children | Only valid child types displayed for UI creation options             |

---

## 7. Authorization and AI Integration

* **Access Control**: Determined purely by path + role assignments; node types **do not impact permissions**.
* **AI Retrieval**: Paths used for pre-filtering; hierarchy ensures logical scoping of documents and chunks.

---

## 8. Explainability

The system must support queries such as:

* “What is the type of this node?”
* “What child types are allowed under this node?”
* “What is the path from root to this node?”

These support **UI enforcement** and **auditable hierarchy reasoning**.

---

## 9. Non-Goals

* Node types are **not access control objects**.
* Node types do **not define roles, permissions, or inheritance**.
* This specification does **not address authorization logic**, which is defined separately.

---

## 10. Stability Guarantee

* Default hierarchy and type rules are **stable for v1**.
* Admin customization allows flexibility without impacting path-based authorization.
* Future versions may extend node types, allowed children, or folder rules, but must preserve:

  * Single root authority
  * Path-based hierarchy
  * Authorization independence

---

## 11. User-to-Organization Relationship

### 11.1 Models

The system supports two conceptual models for user-to-organization relationships:

1. **Single-Organization Users** (default MVP)

   * Each user belongs to exactly one organization.
   * The organization root serves as the **implicit scope** for all role assignments and hierarchical nodes.
   * Simplifies access control queries: the `org_id` can be inferred from the user.

2. **Multi-Organization Users (Future/Optional)**

   * Users can belong to multiple organizations.
   * Users may have **different roles, permissions, and access scopes in each organization**.
   * Access control computations are **scoped by both organization and path**, ensuring strict isolation.
   * All nodes, role assignments, and paths must include `org_id` in queries to prevent cross-org access.

---

### 11.2 Access Control Implications

| Aspect                   | Single-Org Model                     | Multi-Org Model                                                 |
| ------------------------ | ------------------------------------ | --------------------------------------------------------------- |
| Role Assignment          | `(user, role, path, should_inherit)` | `(user, role, path, should_inherit, org_id)`                    |
| Node Paths               | Unique per organization              | Unique per organization; `org_id` required in queries           |
| Authorization Resolution | `Applicable(U, R, P)`                | `Applicable(U, R, P, org_id)`                                   |
| AI Pre-filtering         | Paths scoped automatically           | Paths must include `org_id` filter to avoid cross-org retrieval |
| Complexity               | Simple                               | Slightly higher, but isolation logic remains clear              |

**Notes:**

* The **core access control rules** (role → permission → path → inheritance) **do not change**.
* The multi-org model introduces **explicit scoping and isolation**, without affecting permission semantics.
* Single-org is suitable for MVP implementations; multi-org can be introduced later for scalability.

---

### 11.3 Implementation Guidance

1. Include `org_id` in all role assignment, node, and query structures.
2. Use materialized paths under the root node per organization:

```
/org/{org_id}/project/{project_id}/team/{team_id}/folder/{folder_id}
```

3. Pre-filter AI retrieval by `org_id` and path to enforce strict isolation.
4. Admin interfaces must reflect the organization context for node creation, role assignment, and access inspection.

---

This spec now formalizes:

* **Default node types and hierarchy**
* **Rules for parent-child relationships**
* **Admin-driven customization during provisioning**
* **Operations and path management**
* **Separation from access control logic**
