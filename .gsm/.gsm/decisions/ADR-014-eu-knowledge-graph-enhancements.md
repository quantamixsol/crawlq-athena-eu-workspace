# ADR-014: EU Knowledge Graph Enhancements
**Date:** 2026-02-11 | **Status:** ACCEPTED

**Context:** US KG implementation uses Neo4j + LLM prompts for graph extraction, KG+RAG hybrid retrieval for chat, and TRACE-based explainability. Investigation (COMMIT 15) revealed several shortcomings that the EU version should address while maintaining full compatibility.

**Decision:** The EU KG frontend will replicate the US visualization approach (Neo4j NVL) while adding enhanced filtering, search, statistics, and export capabilities. Key enhancements:

1. **Enhanced Node Visualization** - Color-coded by 12+ entity types (vs US 4 types), importance-based sizing with graduated scale, confidence-based opacity
2. **Advanced Filtering** - Filter by entity type, confidence threshold, relationship type, source document
3. **In-Graph Search** - Real-time fuzzy search with highlighting and focus
4. **Graph Statistics** - Entity distribution, relationship density, centrality metrics, coverage assessment
5. **Export Capabilities** - JSON, CSV, SVG, PNG with metadata
6. **Cross-Document View** - Visual indicators showing which documents contributed to each entity
7. **Confidence Visualization** - Edge thickness/opacity based on relationship confidence scores
8. **TRACE Integration** - Clickable entities link to TRACE lineage steps

**Consequences:**
- (+) Richer user experience than US version
- (+) Better compliance visibility (GDPR Art. 15 right of access)
- (+) Improved decision-making with statistical overview
- (-) Slightly more complex codebase (~10 components vs US 3)
- (-) May need performance optimization for large graphs (>500 nodes)
