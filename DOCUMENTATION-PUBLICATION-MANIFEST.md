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

## 3. Documents requiring source recovery or further verification

| Document | Language | Target version known from project history | Current repository state | Required action |
|---|---|---:|---|---|
| Enterprise AI Infrastructure: Document Intelligence & RAG Ecosystem | English | 2.0 | Existing root file; older content (header 1.3; internal title/version 1.1; blob observed: 0f16a44b2d0aa9c2ab374dc4372e55ad6332c668) | Recover the exact approved v2.0 source, diff against main, then update on this branch |
| End-to-End Data Flow | English | 2.0 | Existing root file; header 1.1; blob observed: 310cbf7902a8362b1cf553c0414c94c9f01b6492 | Recover the exact approved v2.0 source, diff against main, then update on this branch |
| Data-Lake-Discussion-Summary / Data Lake Architecture & Integration — Discussion Summary | English | 2.0 | Existing root file; header 1.1; blob observed: b5c54b223d9cd9332b9ff6a7179c8120fe01e56e | Recover the exact approved v2.0 source, diff against main, then update on this branch |
| Architecture Requirements & Constraints | Persian | 0.1 | No same-name file found in the available project Library inventory or repository tree | Recover the final Persian source and verify bilingual parity before adding |
| Project Roadmap, Milestones & Progress Tracker | English and Persian | v0.1 (version/status require verification) | Older RoadMap_1.md is under Archive; current paired roadmap sources were not available in the current Library inventory | Recover exact latest sources; decide whether these are baseline dependencies or a later publication batch |
| Requirements Traceability Matrix (RTM) | Integrated, likely English-first | To be produced | No final requirement-level RTM identified | Build from the approved full requirement text (114 IDs); include as HLA Appendix C if appropriate, avoiding a duplicate standalone source unless needed |

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
- Three existing root documents identified for later exact-source updates.
- Persian Requirements, roadmap sources, and final RTM remain outstanding.
- HLA review package remains a draft review artifact, not a final baseline.
