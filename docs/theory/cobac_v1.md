# CoBAC: A Coordinate-Based Access Control Architecture for Scalable Multi-Tenant RAG Systems

**Author:** Ahmad Shah
**Date:** January 3, 2026
**Version:** 1.0
**DOI:** [https://doi.org/10.5281/zenodo.18130690](https://doi.org/10.5281/zenodo.18130690)

---

## Abstract

Retrieval-Augmented Generation (RAG) reduces AI hallucination, but applying it to organizational knowledge bases introduces a critical security vulnerability: the **Hallucination of Authority**. Standard retrieval pipelines lack granular access awareness, risking the exposure of sensitive data to unauthorized users.

Existing mechanisms (RBAC, ABAC) rely on inefficient post-filtering, discarding up to **72% of retrieved context**. **CoBAC (Coordinate-Based Access Control)** is a deterministic architecture enforcing **Pre-Retrieval Filtering**. By coupling data storage coordinates with a **6-dimensional access manifold**, CoBAC ensures that only authorized chunks are retrieved.

Benchmarks demonstrate a **25× reduction in storage overhead**, a **50× improvement in write efficiency**, and optimal token utilization.

**Suggested Citation:**
Shah, A. (2026). *CoBAC: A Coordinate-Based Access Control Architecture for Scalable Multi-Tenant RAG Systems*. Zenodo. [https://doi.org/10.5281/zenodo.18130690](https://doi.org/10.5281/zenodo.18130690)

---

## Contents

1. Introduction
   1.1 The Sequence: Power, Risk, and Waste
   1.2 The Problem: Pre-Filtering Gap

2. Methodology: The CoBAC Architecture
   2.1 Coordinate-Based Addressing
   2.2 The Disjoint Forest: Bulkheading Authority
   2.3 Compiled Intent: Pre-Retrieval Routing

3. Evaluation and Results
   3.1 Experimental Setup
   3.2 Storage Efficiency
   3.3 Write Efficiency
   3.4 Security Validation

4. Discussion
   4.1 The Isomorphism Trade-off: Rigidity vs. Velocity

5. Future Work
   5.1 Towards Topological Polymorphism

6. Conclusion

---

## 1. Introduction

### 1.1 The Sequence: Power, Risk, and Waste

RAG grounds LLMs in private data, reducing hallucination. However, applying RAG to organizational knowledge bases introduces severe access-control risks. Without strict controls, a junior intern could retrieve confidential strategy documents.

Most architectures rely on **Post-Filtering**: retrieving top-*k* documents and discarding unauthorized ones. This wastes computation and allows unauthorized data to enter the LLM context—producing a **Hallucination of Authority**.

**Figure 1:** Hallucination of Authority — post-filtering allows unauthorized data into the LLM context.

### 1.2 The Problem: Pre-Filtering Gap

Existing access control systems cannot efficiently pre-filter vector retrieval:

* **RBAC / ABAC** rules are too complex to push into vector queries.
* **Graph-based systems (e.g., Zanzibar)** can answer authorization checks but cannot enumerate searchable assets.

---

## 2. Methodology: The CoBAC Architecture

### 2.1 Coordinate-Based Addressing

CoBAC treats authorization as a **spatial coordinate system**, not a rule engine. Every user *U* and resource *R* is mapped to a *(2 + N)*-dimensional manifold.

In the reference implementation (*N = 4*), this yields a **6-dimensional coordinate**:

```
C = (Res, Act, Horg, Hproj, Vdept, Vteam)
```

This enables **Data-Control Isomorphism**: the security layer and the storage layer share the same geometry. A document is searchable *only if* it resides in a partition matching its access coordinates.

### 2.2 The Disjoint Forest: Bulkheading Authority

To prevent lateral privilege escalation, CoBAC enforces a **Disjoint Forest Topology**:

* **Orthogonal Roots**
  Organization (*Horg*) and Project (*Hproj*) trees share no common ancestor.

* **Namespace Isolation**
  Authority in one tree provides no implicit authority in another.

* **Explicit Bridging**
  Access to sensitive projects requires explicit coordinate tuples, acting as a bulkhead against “God Mode” compromise.

This breaks traditional connected RBAC graphs into mathematically independent trees.

### 2.3 Compiled Intent: Pre-Retrieval Routing

**Compiled Intent** is the operational core of CoBAC. Authorization is evaluated **at session initiation**, not at retrieval time.

**Pipeline:**

```
User Session → CoBAC Compiler → Compiled Intent (Coordinate Set)
                                      ↓
                                Vector DB Filter → LLM
```

**Figure 2:** The Compiled Intent Pipeline — authorization is transformed into a database address before retrieval.

The compiled intent `CI(u)` is a list of authorized coordinates. Retrieval becomes a **Point-in-Space Lookup**, implemented as a metadata filter (e.g., `WHERE team_id IN (...)`). Modern vector engines execute this at **O(1)** relative to dataset size, ensuring **100% authorized tokens**.

---

## 3. Evaluation and Results

### 3.1 Experimental Setup

Benchmarks were conducted on macOS (Darwin ARM64) using PostgreSQL. The dataset consisted of **50,000 synthetic text chunks** across **50 teams**, simulating enterprise complexity. Tests included high-concurrency read/write workloads.

### 3.2 Storage Efficiency

**Table 1: Storage Overhead Scaling**

| Teams/User |  RBAC | ABAC | CoBAC | Factor |
| ---------: | ----: | ---: | ----: | -----: |
|          1 |   100 |  100 |   200 |   0.5× |
|         10 | 1,000 |  100 |   200 |   5.0× |
|         50 | 5,000 |  100 |   200 |  25.0× |

### 3.3 Write Efficiency

**Table 2: Write Amplification**

| Teams Added | RBAC Ops   | CoBAC Ops | Improvement |
| ----------: | ---------- | --------- | ----------- |
|           5 | 5 INSERTS  | 1 UPDATE  | 5:1         |
|          10 | 10 INSERTS | 1 UPDATE  | 10:1        |
|          50 | 50 INSERTS | 1 UPDATE  | 50:1        |

### 3.4 Security Validation

Threat modeling assumed an internal adversary with valid Team A credentials. Attempts at lateral movement and privilege escalation were **blocked 100%**. Invalid coordinate requests failed before query execution, leaking **zero unauthorized tokens**.

---

## 4. Discussion

### 4.1 The Isomorphism Trade-off: Rigidity vs. Velocity

CoBAC enforces **Data-Control Isomorphism**, coupling schema and security.

* **Cost (Rigidity)**
  Hierarchy changes (e.g., inserting a “Squad” layer) require schema migration and re-indexing.

* **Benefit (Velocity)**
  Authorization becomes a routing operation (**O(1)**) rather than a search (**O(N)**). For high-scale RAG systems, this trade-off eliminates post-filtering waste and reduces latency.

---

## 5. Future Work

### 5.1 Towards Topological Polymorphism

The v1.0 reference implementation uses a fixed 6-dimensional schema. Future iterations will introduce **path-based coordinates**:

```
org:acme/proj:apollo/env:prod
```

This enables **topological polymorphism**, allowing tenants to define variable-depth hierarchies within the same engine—bridging CoBAC’s runtime efficiency with structural flexibility.

---

## 6. Conclusion

RAG systems expose the inefficiencies and security risks of post-retrieval authorization. CoBAC reframes access control as a **deterministic coordinate function applied before retrieval**.

By enforcing pre-filtering through disjoint forests and compiled intent, CoBAC guarantees **zero-waste token utilization** and provable isolation. While trading schema flexibility for performance, empirical results confirm CoBAC as a robust foundation for AI-native, multi-tenant systems.

---

## References

1. Ferraiolo, D. F., Kuhn, D. R. (1992). Role-Based Access Control. *National Computer Security Conference*.
2. Hu, V. C., et al. (2014). *Guide to Attribute Based Access Control (ABAC)*. NIST SP 800-162.
3. Hu, V. C., Kuhn, D. R., Ferraiolo, D. F. (2015). Attribute-Based Access Control. *IEEE Computer*.
4. Pang, R., et al. (2019). Zanzibar. *USENIX ATC*.
5. Silva, E., et al. (2018). Distributed ABAC. *Future Generation Computer Systems*.
6. Sladić, G., et al. (2013). Context-Aware Access Control for NoSQL. *ComSIS*.
7. Lewis, P., et al. (2020). Retrieval-Augmented Generation. *NeurIPS*.
