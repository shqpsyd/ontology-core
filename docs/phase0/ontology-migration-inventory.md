# Phase 0 — Ontology Migration Inventory

Status: **baseline / candidate mapping, not an approved DSL**. Date: 2026-10-26.
Source snapshots: supplier-decision-harness main `fcb54e72bc896231a3ff76be2d2d45a22b39d9b4`; data-collector-agent main `ab76892f5a07e1a7d362a83864f3e065197471f3`.

## 1. Scope and evidence
This inventory covers existing contract declarations and execution entrypoints, not every function. References: [Supplier core contracts](https://github.com/shqpsyd/supplier-decision-harness/blob/main/docs/design/02_domain_contracts.md), [boundary contracts](https://github.com/shqpsyd/supplier-decision-harness/blob/main/docs/design/06_domain_boundary_contracts.md), [data collector normative contract](https://github.com/shqpsyd/data-collector-agent/blob/main/docs/contracts.md). Existing authoritative business rules remain with supplier's `docs/implementation/tool_and_registry_contracts.md`, `policy.md`, and code.

## 2. Candidate semantic catalog

| Candidate ontology concept | Concrete existing source | Kind | Ontology owns | Existing runtime owns | Migration notes |
| --- | --- | --- | --- | --- | --- |
| RFQ | `src/supplier_harness/domain.py::RFQ` | Object | identity, properties, units, lifecycle vocabulary | parsing/normalization, storage | preserve partial/ambiguous fields; do not conflate draft with VALID |
| Decision / QuoteDecision | `domain.py::Decision`; `service.py::DecisionService` | Object | typed result, state labels, refs | state transition, persistence | distinguish decision from approved quotation/export |
| Review | `domain.py::Review`; `service.py::review` | Object/Action input | intent and human authority | reviewer authorization, optimistic revision | EDIT/REJECT/ACCEPT semantics preserved |
| Fact | `domain.py::Fact` | Object/Value | provenance, freshness, source, typed value | fact resolution, refresh | mtime/retrieval time not evidence of business freshness |
| Gate | `domain.py::Gate` | Value/Guard result | policy/registry ref, blocking, resolution | validation and escalation | gate source and resolution types must survive conversion |
| PriceResult / CalculationTrace | `domain.py::PriceResult, CalculationTrace` | Value | Decimal, units, trace contract | `calculator.py::calculate` | no duplicate pricing formula in YAML |
| PolicyRule | `domain.py::PolicyRule`; `policy_service.py` | Policy reference | stable identity/version/scope | approved policy evaluation and publication | separate hard constraints from parameters and arithmetic |
| ProductResolution / PricingCapability | `product_resolution.py`; `service.py::pricing_capability_for` | Result/Guard | semantic status vocabulary | resolver/validation | manufacturing approval ≠ pricing support |
| Delivery evidence / judgment | `delivery.py`; boundary contracts | Object/Evidence | scoped proof and version links | reconciliation and approval | evidence bound to quantity/scope/time |
| FactGatewayPort | `domain.py::FactGatewayPort.get/refresh` | Query binding | query input/output/quality contract | existing implementation | protocol is not an independent Fact Service |
| DecisionService.create | `service.py` | Action candidate | draft/decision contract and guards | workflow + calculation + storage | action name must not imply final approved quote |
| DecisionService.review | `service.py` | Action candidate | reviewer input/state transition contract | owner authorization/revision | human decision stays authoritative |
| DecisionService.reply | `service.py` | Query/Action candidate | distinguish preview/export semantic effect | expiry, approved state, audit | classify side effects before DSL freeze |
| Regression fixtures | `tests/test_regression_and_invariants.py`; `case_source.py` | Conformance evidence | expected equivalence conditions | test harness | source document owns fixture, generated JSON must match |
| Data FunctionSpec | `data_collector/models.py`; `functions.py` | Capability descriptor | semantic binding ID / typed signature (future) | existing function dispatch | inferred function ≠ proven business meaning |
| Data Manifest | `data_collector/models.py` / `releases.py` | Binding manifest | contract refs / quality requirements (future) | source selection & dependencies | version/hash/scope must remain |
| Stable invocation | `data_collector/api.py::invoke`, `functions.py` | Adapter boundary | binding lookup and capability reference | worker/isolation/validation | host MUST NOT import generated adapter |
| Immutable release | `data_collector/releases.py` | Implementation release | referencing released binding version | publish/ACTIVE/recovery | never rewrite active release |
| Scan / change declarations | `scanner.py`, `declarations.py` | Data-governance event | only exposed via approved contract | scanning, dedup, closure | do not replicate filesystem algorithm |

## 3. First vertical slice (strict boundary)
**Chosen:** incoming RFQ → normalized RFQ validation → obtain verified material fact → evaluate manufacturing/pricing capability and price draft → review/approval gate. This is a *slice of the full existing DecisionService path*, **not** a new shortened pricing shortcut.

Initial externally exposed candidate operations:
- `GetRFQ` (read/query; owner and persistence mapping to be confirmed)
- `GetMaterialPrice` (read/query; bind through `FactGatewayPort`; source/validity required)
- `GenerateQuotationDraft` (Action; adapt `DecisionService.create`, full preconditions remain)
- `ReviewQuotationDecision` (Action; adapt `DecisionService.review`, human authorization required)
- `GetApprovedReply` (possible read/controlled operation; classify actual effects and expiry checks before freezing)

Do **not** expose `calculator.calculate` as an unrestricted Agent-level quotation operation. Do **not** treat FactGateway refresh as read-only without determining its writes/external side effects.

## 4. Extraction classification
**Direct mechanical discovery:** typed Pydantic models, Protocol signatures, FastAPI paths, service method signatures, test fixture IDs.
**Semantic review required:** canonical object identity, currency/unit precision, policy scope, confirmation/authorization, action effects, temporal validity, fact freshness, provenance meaning, object lifecycle, privacy.
**Not ontology content:** runtime framework adapter, raw filesystem scan algorithm, storage transaction internals, AI prompt text, calculator arithmetic implementation, container execution internals.

## 5. Known gaps / phase 1 inputs
- Determine stable canonical business IDs vs transient persistence IDs.
- Confirm generated API operation IDs and exact status/error vocabulary.
- Audit `DecisionService.reply` side effects before Query/Action classification.
- Review actual Data Collector `FunctionSpec` and manifest serialization before writing a binding converter.
- Specify versioned ontology-to-implementation mapping and what represents missing/ambiguous/unverified data.
- Capture an executable golden slice, including the old path, isolated write-action replay, provenance, and policies.

**Exit:** This is a bounded, source-referenced inventory; candidate types are **not** accepted as normative until Phase 1 type and semantics review.
