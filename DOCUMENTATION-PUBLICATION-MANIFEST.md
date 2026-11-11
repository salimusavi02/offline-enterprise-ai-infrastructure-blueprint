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
