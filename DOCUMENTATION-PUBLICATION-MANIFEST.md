# Documentation Publication Manifest

**Repository:** salimusavi02/offline-enterprise-ai-infrastructure-blueprint  
**Baseline branch:** main  
**Working branch:** docs/architecture-baseline-v0.1  
**Manifest date:** 2026-11-11  
**Purpose:** Track project documents, approval evidence, repository state, and publication actions. This manifest is a working inventory, not an approval of the HLA baseline.

## 1. Repository baseline

- Default branch: `main`
- Observed main HEAD: `c03550ef51c76d20fb555ebb79965954c452e041`
- Recursive tree audit: completed; GitHub response reported `truncated=false`.
- Branches observed before work: `main` only.
- Working branch created from `main`: `docs/architecture-baseline-v0.1`.

## 2. Documents staged on the working branch

| Document | Language | Project version | Content/approval status | State on main | Action on working branch |
|---|---|---:|---|---|---|
| Architecture Scope & Decision Register.md | English | 0.2 | Content approval recorded in project history; source header still says Draft — Pending Review | No file at repository root; no same-name file found in recursive tree | Added as new root file; reconcile status/header before baseline |
| محدوده معماری و دفتر ثبت تصمیمات.md | Persian | 0.2 (paired edition) | Content approval recorded in project history; verify final bilingual parity | No file at repository root; no same-name file found in recursive tree | Added as new root file; verify parity and status/header |
| Architecture Requirements & Constraints.md | English | 0.1 | Content approval recorded in project history; source header still says Draft — Pending Review | No file at repository root; no same-name file found in recursive tree | Added as new root file; reconcile status/header |
| ARC-HLA-001_HLA-v0.1_Review-Package.md | English | 0.1 review package | Draft for Architecture Review; RTM/traceability still provisional or incomplete | No file at repository root; no same-name file found in recursive tree | Added as review artifact; not a final approved HLA baseline |

All four staged files were fetched back from the working branch after creation and their complete text lengths matched the source text read from the project Library. They were added as separate commits by the GitHub Contents API.

## 3. Documents updated from approved v2.0 sources and remaining verification

| Document | Language | Target version known from project history | Current repository state | Required action |
|---|---|---:|---|---|
| Enterprise AI Infrastructure: Document Intelligence & RAG Ecosystem | English | 2.0 | Content approval recorded in project history (2026-11-10); source header remains Proposed / Pending Document Approval | Replaced on working branch; exact source-to-GitHub read-back verified; commit `f46f0912186e7bc7c57942f19d08c240de7c0eb2` |
| End-to-End Data Flow | English | 2.0 | Content approval recorded in project history (2026-11-10); source header remains Proposed / Pending Document Approval | Replaced on working branch; exact source-to-GitHub read-back verified; commit `e7695189fdd6aceae7c2a52b8f5feebf009df203` |
| Data-Lake-Discussion-Summary / Data Lake Architecture & Integration — Discussion Summary | English | 2.0 | Content approval recorded in project history (2026-11-10); source header remains Proposed / Pending Review | Replaced on working branch; exact source-to-GitHub read-back verified; commit `af0b97fd834c59bc050536f4f2298fe79113fc9d` |
| Architecture Requirements & Constraints | Persian | 0.1 | No same-name file found in the available project Library inventory or repository tree | Recover the final Persian source and verify bilingual parity before adding |
| Project Roadmap, Milestones & Progress Tracker | English and Persian | v0.1 (version/status require verification) | Older RoadMap_1.md is under Archive; current paired roadmap sources were not available in the current Library inventory | Recover exact latest sources; decide whether these are baseline dependencies or a later publication batch |
| Requirements Traceability Matrix (RTM) | Integrated, likely English-first | To be produced | No final requirement-level RTM identified | Build from the approved full requirement text (114 IDs); include as HLA Appendix C if appropriate, avoiding a duplicate standalone source unless needed |

### 3.1 Cross-document review findings

The three v2.0 documents were reviewed against one another and against the staged Scope & Decision Register, Requirements & Constraints, and HLA review package. The principal data-ownership and retrieval invariants are aligned: MinIO owns original and canonical content artifacts; PostgreSQL owns operational metadata, ACL, lifecycle, state, and lineage; Qdrant and OpenSearch are separate derived/rebuildable indexes; hybrid retrieval uses RRF; authorization is revalidated against PostgreSQL before canonical content fetch/context assembly; and the initial deployment remains single-node, offline-first, and Docker Compose-based.

The following issues remain visible in the source documents and were not silently edited during publication:

1. **ADR identifier collision:** the v2.0 Ecosystem document maps ADR-001 to Single-Node, ADR-002 to MinIO, ADR-003 to separate Qdrant/OpenSearch, and ADR-004 to canonical chunk ownership. The staged Scope & Decision Register/HLA use ADR-001…ADR-005 with different mappings (including PostgreSQL as ADR-003, search engines as ADR-004, and Single-Node/Docker Compose as ADR-005). The Data Lake summary also labels canonical chunks as ADR-004, conflicting with the staged ADR-004 mapping.
2. **OPEN identifier collision:** the v2.0 Ecosystem document assigns OPEN-001…OPEN-008 to normalization, chunking, tokenizer, ACL, PostgreSQL schema, analyzer, sizing, and backup objectives. The staged Scope & Decision Register/HLA assign these identifiers differently and extend through OPEN-011 (including Entity-Centric Documents, Backup/Recovery, normalization, analyzer, chunking, and index activation). Cross-references must be reconciled before treating the identifiers as authoritative.
3. **Logical architecture taxonomy:** the Data Lake summary describes Data Lake as part of a seven-layer logical architecture; the staged HLA uses three logical planes over a foundational data layer. The Ecosystem v2.0 uses its own logical-layer grouping. These descriptions need a common taxonomy or an explicit statement that they are views at different abstraction levels.
4. **Index-generation semantics:** the v2.0 documents require compatible active Qdrant/OpenSearch versions and coordinated validation/activation, but do not explicitly define the accepted shared Index Generation identity and Strict Hybrid behavior as a single normative contract. HLA describes paired readiness and controlled activation; the shared-generation invariant should be made explicit in the appropriate baseline/ADR.
5. **Approval/status headers:** project history records content approval of all three v2.0 documents on 2026-11-10, while their source headers still say Proposed / Pending Review. This publication preserved the exact source headers; content approval and document-status wording remain distinct and should be reconciled deliberately.

These are review findings, not corrections. No source content was changed to resolve them.

## 4. Existing repository material — retain, review, or archive

The recursive tree contains existing material beyond the publication candidates, including:

- `ADR-SSOT-001.md`, `AGENTS.md`, `PROJECT_RULES.md`, and `README.md`.
- Older architecture and roadmap material under `Archive/`.
- SINA operational documents and implementation artifacts.
- Legacy PDF, PPTX, PNG, and MP4 assets.

These are not automatically replaced or deleted by this work. In line with the current project decision, legacy PDF/PPTX/DOCX and image deliverables should be reviewed or regenerated after HLA stabilization; no such binaries are staged in this first pass.

## 5. Publication controls and open work

1. Do not merge this branch into `main` until the complete document set is reconciled and explicitly approved.
2. Do not infer approval from a file being present in this branch. Content approval, review status, and publication approval are separate states.
3. Complete requirement-level traceability for all 114 requirement IDs using the exact approved requirement statements.
4. Verify bilingual parity, cross-references, document versions, status headers, ADR/OPEN references, and README navigation.
5. Preserve legacy archive content unless a separately reviewed archival/removal decision is made.
6. No main-branch file has been modified, deleted, or merged as part of this staging pass.

## 6. Current staging summary

- Working branch created from `main`.
- Four Markdown files staged and read-back verified.
- Three existing root documents replaced from the exact v2.0 source files and read-back verified.
- Persian Requirements, roadmap sources, and final RTM remain outstanding.
- Cross-document ADR/OPEN identifier collisions, logical-architecture taxonomy, explicit shared Index Generation semantics, and approval/status header reconciliation remain open review items.
- HLA review package remains a draft review artifact, not a final baseline.

## 7. RTM and Architecture Gap Analysis — 2026-11-11

| Document | Version | Status | Action on working branch | Notes |
|---|---:|---|---|---|
| ARC-RTM-001_Requirement-Traceability-Matrix.md | 0.1 | Draft — Practical Baseline (Provisional) | Added as new root document | Contains all 114 source requirement IDs and statements, source priorities/verification methods where defined, initial HLA chapter/component mappings, qualitative coverage and limitations. Not a formal verification or acceptance record. |
| ARC-GAP-001_Architecture-Gap-Register.md | 0.1 | Draft — Initial Gap Analysis | Added as new root document | Captures 12 initial gaps and cross-document consistency findings derived from the practical RTM/HLA review. |

### 7.1 Publication and quality controls

- RTM rows were generated from the exact requirement table statements in the branch's Architecture Requirements & Constraints.md; source priority and verification method were retained for FR/DR/SEC/NFR/OPS. CON entries retain their source constraint wording and decision basis.
- All 114 identifiers were checked for presence. HLA mappings and coverage statuses are qualitative first-pass judgments, not paragraph-level evidence.
- RTM preserves the FR-036 source discrepancy (requirement sentence uses MUST; Priority column says SHOULD) without resolving it.
- The INT category is present in the source taxonomy but has no INT-xxx identifiers; none were invented.
- Known ADR/OPEN cross-reference collisions are recorded as unresolved. OPEN references follow the current Scope & Decision Register/HLA mapping provisionally.
- Both documents are drafts for project work. Their publication does not imply final content approval, HLA baseline approval, production readiness, or merge authorization to main.
- The existing three v2.0 technical documents were not modified in this publication batch.

### 7.2 Current staging summary update

- Two new Markdown work products (RTM and Gap Register) are staged on the working branch.
- The working branch remains separate from main; no merge to main is included in this authorization.
- RTM v0.1 is the practical baseline for the next Architecture Gap Analysis/HLA revision cycle and is expected to be refined later.

## 8. Architecture Gap Analysis & HLA Refinement — 2026-11-11

### 8.1 GAP-001 — Performance and capacity targets

- HLA review package refined in English and Persian to require workload-based capacity planning and a representative benchmark before production acceptance.
- The benchmark scope now explicitly includes document volume/growth and format mix, ingestion/processing/indexing rates, concurrency, retrieval latency objectives, CPU-only local inference, and combined same-host resource impact across MinIO, PostgreSQL, Qdrant, OpenSearch, application workloads, and local inference.
- No numerical latency, throughput, concurrency, or hardware-capacity target was invented or approved.
- RTM v0.2 changes NFR-001 from TBD to Partial; NFR-002 and NFR-003 remain TBD; NFR-004 and NFR-005 remain Partial with refined evidence notes. NFR-006 remains Covered.
- GAP Register v0.2 introduces explicit per-gap status tracking: GAP-001 is Partially Resolved; GAP-002…GAP-012 remain Open at this point.
- OPEN-010 (hardware sizing) and OPEN-005 (embedding/LLM selection) remain open. Detailed workload values, benchmark execution, capacity sizing, and acceptance thresholds are deferred to Detailed Architecture / Benchmark and production acceptance.
- Changes are limited to the working branch `docs/architecture-baseline-v0.1`; no merge to `main` is performed.


## 9. Architecture Governance Update v0.4 — 2026-11-14

### 9.1 Accepted architecture contracts

| ADR | Decision | Baseline contract |
|---|---|---|
| ADR-007 | Baseline Document-Level Authorization | PostgreSQL is authoritative for document ACL. Qdrant/OpenSearch ACL pre-filters are defense-in-depth. Authorization is checked after RRF and before canonical MinIO fetch, then rechecked before context assembly. Only authorized canonical content may reach reranking, context, or the local LLM; uncertainty fails closed. |
| ADR-008 | Coordinated Index Set Activation & Rollback | Incremental membership per document version; paired Qdrant/OpenSearch entries reference the same document version and chunk manifest. PostgreSQL owns membership/build state and the active retrieval reference. Both engines must validate before activation; queries pin compatible ACTIVE membership; failed builds preserve prior active membership. |

### 9.2 Gap and traceability disposition

- GAP-006 and GAP-007 are **Resolved for the Initial Production Architecture Baseline** by ADR-007 and ADR-008 respectively. This is architecture-contract closure only.
- OPEN-003 remains open for advanced RBAC, nested groups, ACL inheritance, identity-provider selection, and identity-context freshness/revocation details.
- OPEN-011 is no longer an active open decision; its indexing-activation scope is resolved by ADR-008. API/schema, retry/reconciliation implementation, physical index naming, retention values, and in-flight query draining remain Detailed Architecture work.
- FR-029 and NFR-010 are **Covered** at architecture-design level. FR-021, FR-022, and NFR-009 remain **Partial**. No implementation or runtime verification is claimed.
- RTM coverage snapshot: **Covered 50, Partial 45, TBD 19, Not Covered 0 (114 total)**.
- Other open gaps and decisions remain open; this update does not close unrelated items.

### 9.3 Files updated on the working branch

| File | Revision | Summary |
|---|---:|---|
| Architecture Scope & Decision Register.md | 0.4 | Replaced ADR-008 with incremental Index Set membership and coordinated activation; retained ADR-007 authorization contract. |
| محدوده معماری و دفتر ثبت تصمیمات.md | 0.4 | Persian counterpart aligned with the English decision register. |
| ARC-HLA-001_HLA-v0.1_Review-Package.md | Review package 0.4 | Added two-checkpoint authorization sequence and separate document/membership lifecycle contracts in English and Persian. |
| ARC-RTM-001_Requirement-Traceability-Matrix.md | 0.4 | Updated ADR traceability and coverage judgments; FR-029 and NFR-010 are Covered; FR-021/022 and NFR-009 remain Partial. |
| ARC-GAP-001_Architecture-Gap-Register.md | 0.4 | Recorded GAP-006/007 as resolved for the Initial Production Architecture Baseline, with residual scope preserved. |
| DOCUMENTATION-PUBLICATION-MANIFEST.md | Section 9 | Recorded this coordinated governance update and its boundaries. |

The update is limited to `docs/architecture-baseline-v0.1`. No technical v2.0 source document, archive, or unrelated file is changed. No merge to `main`, push to another branch, or PR is included. The HLA remains **Draft for Architecture Review**; these changes do not establish implementation completion, runtime test evidence, production acceptance, or final HLA baseline approval.
