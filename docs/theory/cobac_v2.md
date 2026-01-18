# CoBAC v2.0: Topological Polymorphism for Scale-Native Access Control in Vector Databases

**Author:** Ahmad Shah
**Date:** January 16, 2026
**Version:** 2.0 (Architectural Evolution)
**Reference v1.0 DOI:** [https://doi.org/10.5281/zenodo.18130690](https://doi.org/10.5281/zenodo.18130690)

---

## Abstract

Retrieval-Augmented Generation (RAG) systems introduce a security failure mode known as the **Hallucination of Authority**, where unauthorized data enters the model context prior to enforcement. CoBAC v1.0 addressed this risk by enforcing **Data-Control Isomorphism** through a fixed-dimensional coordinate manifold, enabling deterministic pre-retrieval filtering. However, this rigidity imposed a structural penalty that limited adoption in dynamic, multi-tenant organizations.

This paper presents **CoBAC v2.0**, an architectural evolution that introduces **Topological Polymorphism**: a path-based coordinate model that preserves authorization invariants while allowing variable hierarchy depth. We formalize the security invariants preserved under polymorphic re-encoding and clarify the transition from strict structural isomorphism to functional isomorphism.

Using **115,301 sentence-level chunks** derived from ArXiv abstracts, we evaluate CoBAC-compatible hierarchical encodings and demonstrate that path-based strategies eliminate the rigidity penalty without degrading retrieval performance. PostgreSQL **LTREE** exhibits depth-invariant behavior within observed bounds, while database-agnostic string paths incur predictable linear degradation.

These results establish CoBAC v2.0 as a practical foundation for **scale-native, zero-trust RAG infrastructure**.

**Keywords:** Access Control, RAG Security, Vector Databases, Topological Polymorphism, LTREE

---

## Contents

1. Introduction
   1.1 From Fixed Manifolds to Polymorphic Topologies
   1.2 The Rigidity Penalty

2. Topological Polymorphism
   2.1 Topology and Invariants
   2.2 Polymorphism
   2.3 Isomorphism Clarification

3. Methodology
   3.1 Admissible Encodings
   3.2 Disjoint Forest Preservation

4. Evaluation
   4.1 Experimental Infrastructure
   4.2 Dataset
   4.3 Experiment A: Retrieval Performance
   4.4 Experiment B: Hierarchy Restructuring
   4.5 Experiment C: Depth Sensitivity

5. Discussion

6. Future Work

7. Conclusion

---

## 1. Introduction

### 1.1 From Fixed Manifolds to Polymorphic Topologies

CoBAC v1.0 introduced **Data-Control Isomorphism**, coupling authorization semantics directly to the physical layout of vector storage. By embedding access coordinates into a fixed six-dimensional manifold, CoBAC enabled deterministic pre-retrieval filtering and eliminated post-hoc authorization waste.

While effective, this design imposed a **rigidity penalty**: organizational evolution required schema mutation and data migration.

### 1.2 The Rigidity Penalty

In real-world enterprises, organizational depth is not static. Teams subdivide, projects branch, and tenants evolve independently. Fixed-dimensional schemas cannot accommodate such change without structural rewrites. This limitation motivates a shift toward variable-depth representations that preserve security guarantees without sacrificing operational agility.

---

## 2. Topological Polymorphism

### 2.1 Topology and Invariants

In CoBAC v2.0, a **topology** is defined as a rooted containment structure scoped to a single authority namespace (e.g., organization or project). Each node represents an authorization boundary, and ancestry encodes containment.

The following invariants are enforced:

* **Disjoint Forest Topology**
  The authorization space is divided into a forest of independent trees, each rooted in a distinct authority namespace (organization, project, tenant). No implicit reachability or inheritance exists across roots. All authorization decisions are evaluated within exactly one tree.

* **Prefix-Constrained Reachability**
  Authorized access is determined by prefix containment over path-encoded coordinates. A resource is reachable if and only if its path is prefixed by at least one authority coordinate assigned to the subject within the same tree.

* **Pre-Retrieval Determinism**
  Authorization predicates are fully resolvable prior to vector similarity search. No post-retrieval filtering or probabilistic enforcement is permitted, eliminating hallucination of authority by construction.

### 2.2 Polymorphism

Topological polymorphism permits **variable hierarchy depth** within a fixed authority root. While the physical encoding of coordinates may change (fixed columns vs. paths), the preserved invariant is **authorized reachability equivalence**. Security semantics remain stable under re-encoding.

### 2.3 Isomorphism Clarification

CoBAC v2.0 intentionally relaxes **structural isomorphism** while preserving **functional isomorphism**. The storage schema no longer mirrors a fixed-dimensional security manifold; instead, compiled authorization intent remains behaviorally equivalent across admissible encodings.

---

## 3. Methodology

### 3.1 Admissible Encodings

We evaluate only hierarchical encodings compatible with CoBAC’s pre-retrieval compilation model:

* **Rigid (v1 Baseline)**
  Fixed-depth reference columns (e.g., UUIDs or KSUIDs), where each column represents an authority boundary.

* **LTREE**
  PostgreSQL-native hierarchical type with GiST indexing.

* **String Path**
  Database-agnostic materialized paths using prefix matching.

Gap-based encodings (e.g., Nested Sets) are excluded due to non-local updates and incompatibility with deterministic routing.

### 3.2 Disjoint Forest Preservation

Authority separation is enforced via root-scoped paths (e.g., `org:/`, `proj:/`). Compilation and query execution are always bound to a single root, preventing lateral privilege inheritance.

---

## 4. Evaluation

### 4.1 Experimental Infrastructure

Benchmarks were executed using a three-phase pipeline (embedding, seeding, benchmarking) across isolated PostgreSQL 17 instances. All HNSW parameters were held constant to isolate metadata filtering behavior.

### 4.2 Dataset

The corpus consists of **115,301 sentence-level chunks** extracted from **16,701 ArXiv abstracts** spanning 20 technical domains.

### 4.3 Experiment A: Retrieval Performance

**Table 1: Retrieval Latency Across Encodings**

| Strategy    | Avg (ms) | P95 (ms) | P99 (ms) |
| ----------- | -------- | -------- | -------- |
| Rigid       | 10.08    | 16.35    | 18.93    |
| LTREE       | 9.85     | 26.60    | 36.89    |
| String Path | 7.81     | 17.15    | 24.30    |

### 4.4 Experiment B: Hierarchy Restructuring

**Table 2: Write Cost During Hierarchy Mutation**

| Strategy    | Rows Updated | Duration (s) |
| ----------- | ------------ | ------------ |
| Rigid       | 2,745        | 1.99         |
| LTREE       | 2,784        | 1.90         |
| String Path | 2,867        | 1.96         |

### 4.5 Experiment C: Depth Sensitivity

Rigid encoding is excluded due to fixed depth.

**Table 3: P95 Latency vs. Hierarchy Depth**

| Depth | LTREE (ms) | String Path (ms) |
| ----: | ---------: | ---------------: |
|     2 |       0.93 |             0.96 |
|     4 |       0.54 |             0.97 |
|     6 |       0.74 |             1.11 |
|     8 |       0.62 |             1.13 |
|    10 |       0.30 |             1.50 |

---

## 5. Discussion

The results demonstrate that topological polymorphism eliminates structural rigidity without introducing retrieval overhead. **LTREE** provides depth-invariant performance and efficient write behavior, making it suitable for mutable, multi-tenant RAG hierarchies. **String paths** offer portability at the cost of linear depth sensitivity.

This trade-off reflects an intentional design choice: CoBAC v2.0 exchanges structural rigidity for operational flexibility while preserving security invariants.

---

## 6. Future Work

Planned efforts include:

* Extending CoBAC to non-SQL vector databases
* Providing a reusable open-source library abstraction
* Eliminating schema migrations and application-specific rewrites during hierarchy evolution

---

## 7. Conclusion

CoBAC v2.0 formalizes **topological polymorphism** as a secure, scalable alternative to fixed-dimensional access control in vector databases. By preserving functional isomorphism and disjoint forest topology, it enables zero-trust RAG deployments without incurring the rigidity penalty of prior designs.

---

## References

1. Shah, A. (2026). *CoBAC: A Coordinate-Based Access Control Architecture for Scalable Multi-Tenant RAG Systems*. Zenodo.
2. Malkov, Y., Yashunin, D. (2018). Efficient and robust ANN search using HNSW. *IEEE TPAMI*.
3. Sigaev, T., Bartunov, O. (2015). *LTREE—Hierarchical Data Type*. PostgreSQL Documentation.
4. Lewis, P., et al. (2020). Retrieval-Augmented Generation. *NeurIPS*.
