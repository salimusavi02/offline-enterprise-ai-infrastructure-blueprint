# ARC-GAP-001 — Architecture Gap Register

**Version:** 0.1  
**Status:** Draft — Initial Gap Analysis  
**Date:** 2026-11-11  
**Inputs:** ARC-RTM-001 v0.1; ARC-HLA-001 HLA v0.1 Review Package; Architecture Scope & Decision Register v0.2  
**Working branch:** `docs/architecture-baseline-v0.1`

## 1. Purpose

This register captures material architecture gaps identified during the practical first-pass RTM. It is intended to guide HLA refinement and the next Architecture Gap Analysis cycle. It is not a defect register for implemented software and does not imply that every gap must be closed before Detailed Architecture.

## 2. Priority definitions

- **High:** affects an architectural invariant, security boundary, production acceptance prerequisite, or a decision needed before detailed design can safely proceed.
- **Medium:** important for completeness and operability, but can be elaborated during HLA refinement or Detailed Architecture if explicitly tracked.
- **Low:** editorial or non-blocking clarification. No Low items are included in this initial register.

## 3. Gap register

| Gap ID | Gap / finding | Related requirements | HLA chapter(s) | Related ADR / OPEN | Priority | Recommended action | Suggested resolution point |
|---|---|---|---|---|---|---|---|
| GAP-001 | Performance and capacity targets are not established | NFR-001…NFR-005 | 14, 15 | OPEN-010; OPEN-005 | High | Define representative workload profile and benchmark plan; establish latency, throughput, capacity and CPU resource targets before production acceptance. | Before production acceptance; detailed targets may be resolved during Detailed Architecture. |
| GAP-002 | Persian normalization policy is unresolved | FR-007, FR-014; NFR-020 | 8, 17 | OPEN-007 | High | Select and validate normalization policy; keep norm-v1.0 as implementation candidate until approved. | Resolve before production processing baseline. |
| GAP-003 | Persian lexical analyzer is not selected or benchmarked | FR-025, FR-031 | 9, 17 | OPEN-008 | High | Benchmark candidate OpenSearch analyzers against representative Persian corpus and define analyzer contract. | Resolve before production search acceptance. |
| GAP-004 | Chunking strategy and parameters are unresolved | FR-009, FR-010, FR-014 | 8, 17 | OPEN-009 | High | Define tokenizer, chunk boundary policy, size and overlap; validate retrieval quality and traceability. | Resolve before production indexing. |
| GAP-005 | Embedding and local LLM choices are unresolved | FR-023, FR-034, FR-035; NFR-005 | 8, 9, 10, 14 | OPEN-005 | High | Evaluate candidate models under offline and CPU-only constraints; document quality/resource trade-offs. | Before model and production baseline selection. |
| GAP-006 | Identity and advanced document authorization model is incomplete | SEC-001…SEC-014; FR-032, FR-039 | 2, 9, 11, 15 | OPEN-003 | High | Define identity source, principal/group/role mapping, ACL representation, revocation and failure behavior; retain PostgreSQL as authority. | Resolve before detailed security design and implementation. |
| GAP-007 | Index generation activation and rollback need a more explicit contract | FR-018…FR-022, FR-029; NFR-008…NFR-010 | 9, 12, 16 | OPEN-011 | High | Specify shared logical Generation ID, paired Qdrant/OpenSearch readiness, PostgreSQL active pointer, partial-failure recovery and rollback. | Resolve before detailed indexing/activation design. |
| GAP-008 | Backup, restore and recovery targets/procedures are open | DR-010; NFR-011…NFR-015; OPS-006, OPS-008 | 15, 16 | OPEN-004 | High | Define protected data domains, backup method/cadence, restore order, RPO/RTO and recovery validation. | Resolve before production acceptance. |
| GAP-009 | Operational telemetry and audit details are incomplete | SEC-010, SEC-014; NFR-021…NFR-025 | 11, 15 | — | Medium | Define critical event catalogue, metrics, logs, traces/correlation, retention and alerting responsibilities. | Can be elaborated during Detailed Architecture. |
| GAP-010 | Application component boundaries and integration contracts are not fully defined | FR-038…FR-042; NFR-016, NFR-017 | 5, 6, 13 | OPEN-006 | Medium | Clarify module responsibilities, ownership, inputs/outputs, synchronous/asynchronous interactions and failure contracts without prematurely splitting deployables. | Resolve during HLA refinement / Detailed Architecture. |
| GAP-011 | Integration requirement category has no requirement identifiers | INT category in Requirements taxonomy | 4, 5, 13 | — | Medium | Decide whether existing FR/NFR requirements sufficiently cover integration or add explicit INT requirements through controlled Requirements revision. | Resolve during Requirements maintenance; do not invent IDs in RTM. |
| GAP-012 | Offline deployment supply-chain and operational validation details are incomplete | OPS-004…OPS-008; SEC-011, SEC-012 | 11, 14, 15, 16 | OPEN-004; OPEN-010 | Medium | Define offline import/provenance/integrity process for images, packages, models and configuration; define post-install health checks. | Elaborate before deployment readiness. |

## 4. Cross-document consistency findings

1. **OPEN identifier collisions:** the Scope & Decision Register and legacy architecture documents assign different subjects to overlapping OPEN identifiers. Normalize the controlling register and update references before treating OPEN IDs as stable cross-document identifiers.
2. **ADR identifier collisions:** legacy v2.0 documents use ADR numbers with mappings that differ from the current Scope & Decision Register/HLA. The Scope & Decision Register should remain the controlling mapping until a deliberate reconciliation is approved.
3. **FR-036 normative discrepancy:** source wording says MUST while Priority is SHOULD. Preserve the source value in RTM and resolve through Requirements change control.
4. **INT taxonomy gap:** INT exists as a category but no INT-xxx identifiers are defined.
5. **HLA evidence depth:** chapter mappings are adequate for practical gap discovery but do not constitute paragraph-level verification evidence.
6. **Index-generation contract:** HLA describes controlled activation and avoids distributed-transaction claims; the shared logical generation identity and exact query/activation guardrails still need explicit consolidation.

## 5. Recommended HLA revision sequence

1. Resolve document-governance collisions in ADR/OPEN cross-references without silently renumbering accepted decisions.
2. Refine HLA chapters 7, 9 and 12 for data ownership, Strict Hybrid retrieval, shared index generation, activation and rollback.
3. Refine HLA chapters 8 and 9 for separate normalization, search analyzer and chunking contracts.
4. Refine HLA chapters 10 and 11 for evidence traceability, identity context and authorization enforcement.
5. Refine HLA chapters 14–16 for offline supply chain, observability, backup and recovery.
6. Revisit RTM coverage statuses after HLA changes.

## 6. Exit criteria for this practical pass

- All 114 source requirement IDs are represented in ARC-RTM-001.
- No unverified mapping is described as implementation or test evidence.
- Material gaps and open decisions are visible and assigned a suggested resolution point.
- HLA revision can proceed without waiting for paragraph-level RTM evidence.

## 7. Limitations

This is a first-pass register based on the current HLA review package and Requirements v0.1. Priority and resolution timing are recommendations, not approved project commitments. No implementation work, test execution, repository merge, or HLA baseline approval is implied.

---
