# Phase 0 — Baseline Regression and Golden Slice Report

Date: 2026-10-26. **Evidence class: static repository inspection only. No test execution in this audit.**
Source revisions: supplier `fcb54e72bc896231a3ff76be2d2d45a22b39d9b4`; collector `ab76892f5a07e1a7d362a83864f3e065197471f3`.

## 1. What was verified
Reviewed source declarations in `src/supplier_harness/{domain,service,calculator}.py`, `tests/test_regression_and_invariants.py`, supplier `docs/design/{02_domain_contracts,06_domain_boundary_contracts}.md`, data collector `data_collector/{functions,releases}.py`, `docs/contracts.md`, and repository module/test trees. Existence of a test/claim is not a new pass. Existing documentation reports **386 pytest**, **61 discovery cases + 1 service trajectory**, and **470 assertions** from an earlier verification; these are historical reported numbers, **not reconfirmed as of this report**.

## 2. Static evidence and required replay oracles

| Gate | Static source / test evidence | Invariant for ontology migration | Phase-0 observed status |
| --- | --- | --- | --- |
| Incomplete/invalid RFQ | `docs/design/02_domain_contracts.md`; `test_invalid_geometry_blocks_even_if_profile_ranges_match` | no calculation for unresolved invalid geometry; expose missing/invalid/ambiguous separately | TEST DEFINED; NOT EXECUTED |
| Tool/fact failure | `test_tool_failure_becomes_auditable_escalation` | escalate, no calculation, no credential leakage in trace | TEST DEFINED; NOT EXECUTED |
| Human risk decision | `test_category_a_requires_specific_human_evidence` | evidence can lead to awaiting review, not automatic APPROVED | TEST DEFINED; NOT EXECUTED |
| Policy version/revision conflict | `test_stale_promotion_base_and_regression_changes_are_rejected` | stale promotions blocked; no lost optimistic version | TEST DEFINED; NOT EXECUTED |
| Review / reply expiry | `test_approved_quote_expiry_cannot_export` | expired approved quotes cannot be exported | TEST DEFINED; NOT EXECUTED |
| Delivery proof scope | `test_delivery_confirmation_is_bound_to_quantity` | evidence binds RFQ quantity/version/scope | TEST DEFINED; NOT EXECUTED |
| Feedback regression | `test_feedback_to_confirmed_regression_to_promotion` | feedback cannot silently promote a business policy | TEST DEFINED; NOT EXECUTED |
| Evaluations source consistency | `test_fixture_bundle_matches_source_document` | 61 discovery cases and source JSON remain in sync | TEST DEFINED; NOT EXECUTED |
| No privileged generated adapter import | `data_collector/functions.py`, `docs/contracts.md` | stable API invokes isolated active release | CODE/CONTRACT REVIEWED; NOT EXECUTED |
| Immutable release activation | `data_collector/releases.py::load_release,prepare` | release tests + hash validation before ACTIVE | CODE/CONTRACT REVIEWED; NOT EXECUTED |
| Business freshness | data collector `docs/contracts.md` | no mtime/retrieved_at inference of business freshness | CONTRACT REVIEWED; NOT EXECUTED |

## 3. Exact golden vertical slice
Input: an existing synthetic/demo RFQ fixture (e.g. `fixtures.DEMO_RFQ`) with a controlled FactGateway, fixed clock, policy/config snapshot and isolated temporary SQLite repository.

Expected steps:
1. `DecisionService.create` interprets/normalizes/validates the RFQ.
2. Resolve price fact through `FactGatewayPort` and retain provenance/freshness.
3. Evaluate geometry/manufacturing and pricing capability gates **before** any calculation.
4. Run the existing calculator with approved policy/config/profile parameters only if eligible.
5. Persist decision, trace, versions, gate outcomes and evidence.
6. Call `DecisionService.review` under authorized reviewer/revision semantics.
7. Call `DecisionService.reply` only if approved and current; reject expired/invalid state.

Capture an old-path oracle for each step before starting ontology migration; replay write-actions in an **isolated cloned repository** rather than twice against real state.

## 4. Executable verification commands (unrun in this report)
From a checkout at the pinned revision, with dependencies installed and local workspace isolated:

```bash
cd supplier-decision-harness
PYTHONPATH=src .venv/bin/python -m pytest -q tests
PYTHONPATH=src .venv/bin/python -m pytest -q tests/test_regression_and_invariants.py
PYTHONPATH=src .venv/bin/python -m supplier_harness.case_source --check
```

Check the repository's current README/CLI before running a version-specific command. To test the collector:

```bash
cd data-collector-agent
uv run pytest -q tests
```

The collector may require a Linux OCI runtime for sandbox integration tests; do not silently skip those tests or interpret an unsupported environment as a pass.

## 5. Golden record schema (proposal for Phase 1)
Store only synthetic / sanitized fixture references, not customer data:

```json
{
  "case_id": "ONTOLOGY-GOLDEN-QUOTE-001",
  "source_revision": "supplier-main-fcb54e7",
  "fixture_ref": "DEMO_RFQ",
  "policy_snapshot_ref": "PINNED_POLICY",
  "clock": "FIXED_UTC",
  "expected": {
    "decision_status": "AWAITING_REVIEW_OR_ESCALATED_BY_GATES",
    "calculation_allowed": "ONLY_IF_ALL_HARD_GATES_PASS",
    "human_approval_required": true,
    "reply_requires_current_validity": true,
    "trace_and_evidence_preserved": true
  }
}
```

This is **a test plan**, not a fabricated expected-value baseline. Freeze actual deterministic statuses, total prices, relevant policy references, and errors from a real execution before doing old/new parity comparisons.

## 6. Phase 0 acceptance and limitations
- [x] Identified candidate vertical slice and legacy owner paths.
- [x] Mapped existing explicit test coverage to safety invariants.
- [x] Defined repeatable test commands and golden record fields.
- [x] Disclosed static-vs-executed evidence boundary.
- [ ] Execute pinned suites in an isolated runtime and record exit code, time, interpreter/dependency versions.
- [ ] Freeze actual result snapshots for read, create, review, expiry and failure paths.
- [ ] Run initial schema-to-binding round trip (Phase 1 compiler spike).

**Readiness:** documentation baseline ready for DSL design; **behavioral parity gate not yet passed**. Do not claim migration is safe until the unchecked items are completed.
