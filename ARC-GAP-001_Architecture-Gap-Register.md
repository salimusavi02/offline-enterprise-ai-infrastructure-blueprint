# ARC-GAP-001 — Architecture Gap Register

**Version:** 0.4  
**Status:** Draft — In Progress  
**Date:** 2026-11-11  
**Inputs:** ARC-RTM-001 v0.4; ARC-HLA-001 review package revision 0.4; Architecture Scope & Decision Register v0.4  
**Working branch:** `docs/architecture-baseline-v0.1`

## 1. Purpose

This register captures material architecture gaps identified during the practical first-pass RTM. It is intended to guide HLA refinement and the next Architecture Gap Analysis cycle. It is not a defect register for implemented software and does not imply that every gap must be closed before Detailed Architecture.

## 2. Priority definitions

- **High:** affects an architectural invariant, security boundary, production acceptance prerequisite, or a decision needed before detailed design can safely proceed.
- **Medium:** important for completeness and operability, but can be elaborated during HLA refinement or Detailed Architecture if explicitly tracked.
- **Low:** editorial or non-blocking clarification. No Low items are included in this initial register.

## 3. Gap register

### 3.1 Status definitions

- **Resolved:** sufficiently addressed at HLA level; no material architecture gap remains for this stage.
- **Partially Resolved:** HLA direction or control is clarified, but measurable targets, detailed contracts, or validation evidence remain outstanding.
- **Deferred:** intentionally assigned to a later stage; the item remains visible and is not considered resolved.
- **Open:** no sufficient resolution has yet been established.

| Gap ID | Gap / finding | Related requirements | HLA chapter(s) | Related ADR / OPEN | Priority | Status | Recommended action | Suggested resolution point |
|---|---|---|---|---|---|---|---|---|
| GAP-001 | Performance and capacity targets are not established | NFR-001…NFR-005 | 14, 15 | OPEN-010; OPEN-005 | High | Partially Resolved | HLA now requires a representative workload profile and combined same-host benchmark; numerical targets and benchmark evidence remain outstanding. | Define workload and benchmark during Detailed Architecture; establish and validate targets before production acceptance. |
| GAP-002 | Persian normalization policy is unresolved | FR-007, FR-014; NFR-020 | 8, 17 | OPEN-007 | High | Open | Select and validate normalization policy; keep norm-v1.0 as implementation candidate until approved. | Resolve before production processing baseline. |
| GAP-003 | Persian lexical analyzer is not selected or benchmarked | FR-025, FR-031 | 9, 17 | OPEN-008 | High | Open | Benchmark candidate OpenSearch analyzers against representative Persian corpus and define analyzer contract. | Resolve before production search acceptance. |
| GAP-004 | Chunking strategy and parameters are unresolved | FR-009, FR-010, FR-014 | 8, 17 | OPEN-009 | High | Open | Define tokenizer, chunk boundary policy, size and overlap; validate retrieval quality and traceability. | Resolve before production indexing. |
| GAP-005 | Embedding and local LLM choices are unresolved | FR-023, FR-034, FR-035; NFR-005 | 8, 9, 10, 14 | OPEN-005 | High | Open | Evaluate candidate models under offline and CPU-only constraints; document quality/resource trade-offs. | Before model and production baseline selection. |
| GAP-006 | Identity and advanced document authorization model is incomplete | SEC-001…SEC-014; FR-032, FR-039 | 2, 9, 11, 15 | ADR-007; OPEN-003 | High | Resolved | Baseline document-level authorization, PostgreSQL authority, dual authorization checkpoints, canonical-content protection, and fail-closed behavior are defined by ADR-007. Advanced RBAC, group hierarchy, ACL inheritance, IdP selection, and identity-context freshness/revocation remain under OPEN-003. | Baseline contract closed; residual extensions remain for Detailed Security Design. |
| GAP-007 | Coordinated Index Set activation and rollback require an explicit contract | FR-018…FR-022, FR-029; NFR-008…NFR-010 | 9, 12, 16 | ADR-008 | High | Resolved | Incremental per-document-version membership, paired Qdrant/OpenSearch validation against the same document version and chunk manifest, PostgreSQL active retrieval reference, query pinning, and preservation of prior active membership are defined by ADR-008. | Baseline contract closed; API/schema, retry/reconciliation implementation, retention, and in-flight query draining remain Detailed Architecture work. |
| GAP-008 | Backup, restore and recovery targets/procedures are open | DR-010; NFR-011…NFR-015; OPS-006, OPS-008 | 15, 16 | OPEN-004 | High | Open | Define protected data domains, backup method/cadence, restore order, RPO/RTO and recovery validation. | Resolve before production acceptance. |
| GAP-009 | Operational telemetry and audit details are incomplete | SEC-010, SEC-014; NFR-021…NFR-025 | 11, 15 | — | Medium | Open | Define critical event catalogue, metrics, logs, traces/correlation, retention and alerting responsibilities. | Can be elaborated during Detailed Architecture. |
| GAP-010 | Application component boundaries and integration contracts are not fully defined | FR-038…FR-042; NFR-016, NFR-017 | 5, 6, 13 | OPEN-006 | Medium | Open | Clarify module responsibilities, ownership, inputs/outputs, synchronous/asynchronous interactions and failure contracts without prematurely splitting deployables. | Resolve during HLA refinement / Detailed Architecture. |
| GAP-011 | Integration requirement category has no requirement identifiers | INT category in Requirements taxonomy | 4, 5, 13 | — | Medium | Open | Decide whether existing FR/NFR requirements sufficiently cover integration or add explicit INT requirements through controlled Requirements revision. | Resolve during Requirements maintenance; do not invent IDs in RTM. |
| GAP-012 | Offline deployment supply-chain and operational validation details are incomplete | OPS-004…OPS-008; SEC-011, SEC-012 | 11, 14, 15, 16 | OPEN-004; OPEN-010 | Medium | Open | Define offline import/provenance/integrity process for images, packages, models and configuration; define post-install health checks. | Elaborate before deployment readiness. |

## 4. Cross-document consistency findings

1. **OPEN identifier collisions:** the Scope & Decision Register and legacy architecture documents assign different subjects to overlapping OPEN identifiers. Normalize the controlling register and update references before treating OPEN IDs as stable cross-document identifiers.
2. **ADR identifier collisions:** legacy v2.0 documents use ADR numbers with mappings that differ from the current Scope & Decision Register/HLA. The Scope & Decision Register should remain the controlling mapping until a deliberate reconciliation is approved.
3. **FR-036 normative discrepancy:** source wording says MUST while Priority is SHOULD. Preserve the source value in RTM and resolve through Requirements change control.
4. **INT taxonomy gap:** INT exists as a category but no INT-xxx identifiers are defined.
5. **HLA evidence depth:** chapter mappings are adequate for practical gap discovery but do not constitute paragraph-level verification evidence.
6. **Index-generation contract:** Resolved by ADR-008 and incorporated into HLA revision 0.3. The contract explicitly avoids distributed-transaction claims; runtime verification and detailed recovery mechanics remain future work.

## 5. Recommended HLA revision sequence

1. Resolve document-governance collisions in ADR/OPEN cross-references without silently renumbering accepted decisions.
2. Incorporate ADR-007 and ADR-008 into HLA security and index activation chapters; the baseline contracts are now recorded in HLA revision 0.3.
3. Refine HLA chapters 8 and 9 for separate normalization, search analyzer and chunking contracts.
4. Refine HLA chapters 10 and 11 for evidence traceability, identity context and authorization enforcement.
5. Refine HLA chapters 14–16 for offline supply chain, observability, backup and recovery.
6. Revisit RTM coverage statuses after HLA changes.

## 6. Exit criteria for this practical pass

- All 114 source requirement IDs are represented in ARC-RTM-001.
- No unverified mapping is described as implementation or test evidence.
- Material gaps and open decisions are visible and assigned a suggested resolution point.
- HLA revision can proceed without waiting for paragraph-level RTM evidence.

## 7. GAP-006 and GAP-007 closure record — 2026-11-14

### GAP-006 — Resolved for Initial Production Architecture Baseline

- Accepted decision: ADR-007 — Baseline Document-Level Authorization.
- PostgreSQL is authoritative for document-level ACL and lifecycle authorization. Qdrant/OpenSearch ACL pre-filters are defense-in-depth only.
- The retrieval flow revalidates each fused candidate against current PostgreSQL authorization before canonical MinIO fetch, validates document version/manifest/integrity, and performs a second authorization check before context assembly.
- Only authorized canonical content may enter reranking, context, or the local LLM. Authorization uncertainty fails closed.
- OPEN-003 remains for advanced RBAC, nested groups, ACL inheritance, identity-provider selection, and identity-context freshness/revocation details.

### GAP-007 — Resolved for Initial Production Architecture Baseline

- Accepted decision: ADR-008 — Coordinated Index Set Activation & Rollback.
- Index membership is incremental per document version. Paired Qdrant/OpenSearch memberships must reference the same document version and chunk manifest.
- PostgreSQL is authoritative for membership/build state, compatibility, and the active retrieval reference. Both engine representations must be complete and validated before activation.
- Membership lifecycle is `BUILDING → VALIDATING → READY → ACTIVE → RETIRED`; failures enter `FAILED`. A failed new build preserves the prior active membership.
- Query Planner uses only ACTIVE compatible memberships and pins the selected active retrieval reference for each query. No full-corpus rebuild is required for a single document update.
- No distributed transaction is assumed; the active pointer change is atomic only within PostgreSQL. API/schema, retries/reconciliation, retention, and in-flight query draining remain Detailed Architecture work.

These closures are architecture-contract closures for the Initial Production Baseline. They are not implementation completion, test execution, security certification, production acceptance, or HLA baseline approval.

## 8. Limitations

This is a first-pass register based on the current HLA review package and Requirements v0.1. Priority and resolution timing are recommendations, not approved project commitments. No implementation work, test execution, repository merge, or HLA baseline approval is implied.

---
