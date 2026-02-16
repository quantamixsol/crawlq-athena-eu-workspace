# ADR-039: OWL/SHACL Foundation — Python-Side Ontology Validation

**Date:** 2026-02-15 | **Status:** ACCEPTED

## Context

The CrawlQ EU Knowledge Graph on Neo4j had no formal ontology constraints:
- Entity types were free-text strings (LLM could produce any type)
- No cardinality enforcement on relationships
- No domain/range constraints (e.g., WORKS_AT could link any two entity types)
- Generic relationship labels (RELATED_TO, MENTIONS) polluted the graph
- No type hierarchy for semantic reasoning (can't query "all Governance entities")

The Enterprise AI Playbook mandates OWL class hierarchy and SHACL validation gates.

## Decision

Implement a Python-side ontology module (`shared/ontology.py`) that provides:

1. **OWL Class Hierarchy**: 7-branch taxonomy (Agent, Spatial, Temporal, Governance, Artifact, Measurement, Cognitive, Lineage) with 25+ leaf entity types
2. **SHACL-like Validation**: Property constraints (confidence 0-1, importance 0-10, name max 500 chars) and relationship domain/range rules
3. **Validation Gates**: `validate_and_fix_entity()` and `validate_and_fix_relationship()` functions called before Neo4j storage
4. **Ontology Prompt Injection**: `get_ontology_prompt_context()` injected into LLM extraction prompts to guide toward valid types
5. **Auto-fix**: Invalid values are clamped/corrected rather than rejected (graceful degradation)

Integration points:
- EUGraphBuilder: validates entities/relationships before `MERGE` in Neo4j
- EUResponseKGExtractor: injects ontology into Bedrock extraction prompt

## Consequences

- (+) Entity types conform to a formal hierarchy — enables type-based reasoning
- (+) Invalid confidence/importance values are caught and corrected before storage
- (+) LLM extraction guided toward valid types (fewer UNKNOWN entities)
- (+) Foundation for future semantic reasoning (transitive relationships, class queries)
- (-) Python-side validation, not RDF-native — no SPARQL/OWL reasoning engine
- (-) Validation is warn-only for relationships (non-blocking) to avoid data loss
