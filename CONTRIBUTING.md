# Contributing to CoBAC

## Getting Started

1. Read [ARCHITECTURE.md](ARCHITECTURE.md) - understand core principles
2. Check [GitHub Issues](issues) for "good first issue" tags
3. Join discussions (we're friendly!)

## Development Setup
```bash
git clone https://github.com/ahmadshah2103/cobac
cd cobac
npm install
npm test
```

## Contribution Process

1. **Fork** the repository
2. **Create a branch**: `git checkout -b feature/your-feature`
3. **Make changes** and add tests
4. **Run tests**: `npm test`
5. **Submit PR** with clear description

## What We're Looking For

**High Priority:**
- Vector database adapters (Weaviate, Milvus, etc.)
- Performance benchmarks
- Documentation improvements
- Bug fixes

**Medium Priority:**
- Additional database adapters (MySQL, MongoDB)
- Caching implementations
- Example applications

**Requires RFC (Request for Comments):**
- Core authorization model changes
- New security invariants
- Breaking API changes

## Code Review Process

- All PRs require 1 maintainer approval
- Core maintainers: 2 approvals
- Architecture changes: Lead Maintainer approval required

## Recognition

- Contributors added to CONTRIBUTORS.md
- Significant contributions → Collaborator status
- Sustained contributions (3+ months) → Maintainer nomination

## Questions?

Open a GitHub Discussion or ping @ahmadshah2103 in issues.
