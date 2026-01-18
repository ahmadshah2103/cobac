# CoBAC: Pre-Retrieval Authorization for RAG Systems

**Creator:** Ahmad Shah  
**Status:** Alpha (v0.1.0)  
**License:** MIT  
**Research Paper:** [v1](https://doi.org/10.5281/zenodo.18130690)

## What is CoBAC?

CoBAC (Coordinate-Based Access Control) is the first open-source framework 
for **pre-retrieval authorization** in RAG (Retrieval-Augmented Generation) 
systems. Unlike traditional post-filtering approaches that waste 50-70% of 
retrieved chunks, CoBAC enforces access control BEFORE retrieval.

## The Problem

Current RAG systems have a critical security flaw: unauthorized documents 
are retrieved and then filtered out, wasting compute and leaking sensitive 
data into LLM context (Hallucination of Authority).

## The Solution

CoBAC treats authorization as a **spatial routing problem**:
- Paths encode organizational structure: `/org/project/team/doc`
- Permissions compile to searchable coordinates
- Vector search filters on authorized paths only
- Zero unauthorized chunks retrieved

## Key Features

- ✅ **Pre-Retrieval Filtering**: Only authorized chunks retrieved
- ✅ **Hierarchical Access Control**: Variable-depth organizational trees
- ✅ **Database Agnostic**: PostgreSQL, MySQL, MongoDB support
- ✅ **Vector DB Ready**: Qdrant, Weaviate, Milvus, Pinecone adapters
- ✅ **Zero Trust**: Disjoint forest topology prevents privilege escalation

## Quick Start
```bash
npm install @cobac/core @cobac/adapter-qdrant

# See examples/quickstart.ts
```

## Project Status

- [ ] Core authorization model (in progress)
- [ ] PostgreSQL LTREE adapter (planned)
- [ ] Vector DB adapters (planned)
- [ ] Python SDK (planned)
- [ ] Audit logging (planned)

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md).

**Note:** This project is in active development. Architecture decisions 
require approval from the Lead Maintainer (see [GOVERNANCE.md](GOVERNANCE.md)).

## Research

Based on academic research:
- v1 Paper: [CoBAC: Coordinate-Based Access Control for RAG](https://doi.org/10.5281/zenodo.18130690)
- v2 Paper: [Coming Soon - Theory to Practice]

## License

MIT License - see [LICENSE](LICENSE)

## Citation
```bibtex
@software{cobac2026,
  author = {Shah, Ahmad},
  title = {CoBAC: Pre-Retrieval Authorization for RAG Systems},
  year = {2026},
  url = {https://github.com/yourusername/cobac}
}
```
