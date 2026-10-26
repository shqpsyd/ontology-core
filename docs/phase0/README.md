# Ontology migration — Phase 0

Phase 0 is a **source-referenced documentation baseline**, not a completed behavioral parity certification.

- [Migration inventory](ontology-migration-inventory.md) — existing symbols, candidate Ontology concepts, adapter candidates and first vertical slice.
- [Semantic boundaries](semantic-boundaries.md) — owners, authority, runtime/agent responsibilities, extension and safety rules.
- [Regression baseline](baseline-regression-report.md) — source-supported invariants, historical vs executed testing, golden slice and outstanding verification steps.

Pinned sources (as reviewed 2026-10-26): [supplier-decision-harness](https://github.com/shqpsyd/supplier-decision-harness/tree/fcb54e72bc896231a3ff76be2d2d45a22b39d9b4) and [data-collector-agent](https://github.com/shqpsyd/data-collector-agent/tree/ab76892f5a07e1a7d362a83864f3e065197471f3).

Phase 1 must not silently promote draft schema names or guessed invariants into production permissions. Existing Supplier domain service and Data Collector sandbox remain authoritative execution layers.
