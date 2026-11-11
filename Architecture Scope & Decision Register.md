# Architecture Scope & Decision Register

**Document:** Architecture Scope & Decision Register  
**Version:** 0.2  
**Status:** Draft — Pending Review  
**Date:** 2026-11-10  
**Project:** Offline Enterprise AI Infrastructure — Document Intelligence & RAG Ecosystem

---

## 1. Purpose

This document establishes the architectural scope, accepted architectural decisions, open decisions, and governance principles for the Offline Enterprise AI Infrastructure project.

Its primary objectives are to:

- Define the boundaries of the High-Level Architecture.
- Establish a clear distinction between core architectural capabilities, initial production requirements, and future extensions.
- Maintain a centralized and traceable register of architectural decisions.
- Prevent scope expansion and premature technology commitments.
- Provide a consistent reference for subsequent architecture documents, detailed designs, implementation plans, and technical evaluations.

This document is a governance and architecture-control artifact. It does not replace the detailed architecture documents or implementation specifications.

## 2. Architectural Scope

The project follows a phased architecture and implementation approach.

The scope is divided into three distinct categories:

1. **Core Architecture Scope:** Capabilities and architectural components that define the baseline system architecture.
2. **Initial Production Release Scope:** Capabilities that must be implemented, validated, and hardened before declaring the first production release.
3. **Future Capabilities:** Extensions that are deferred, subject to feasibility studies, or explicitly excluded from the initial architecture baseline.

### 2.1 Core Architecture Scope

| Domain | Capability | Scope Classification | Notes |
|---|---|---|---|
| Deployment | Single-node physical architecture | In Scope | Initial deployment topology |
| Containerization | Docker Compose | In Scope | Initial orchestration approach |
| Data Lake | MinIO | In Scope | Foundational content storage |
| Operational Storage | PostgreSQL | In Scope | Metadata, lifecycle, ACL, state, and lineage |
| Vector Search | Qdrant | In Scope | Vector index |
| Lexical Search | OpenSearch | In Scope | BM25 and lexical retrieval |
| Retrieval | Hybrid Search | In Scope | Separate vector and lexical retrieval with fusion |
| Document Processing | Persian-first document processing | In Scope | Persian language support is a core requirement |
| Chunking | Document chunking capability | In Scope | Detailed strategy and parameters remain open |
| Embedding | Embedding generation and management | In Scope | Model selection remains open |
| LLM Integration | Local LLM inference integration | In Scope | Model selection and serving details remain open |
| Access Control | Basic document-level authorization | In Scope | Must be enforced from the initial implementation |
| Observability | Logging, metrics, tracing, and audit events | In Scope | Baseline operational observability |
| Application Architecture | Modular and maintainable application architecture | In Scope | Clear separation of responsibilities |
| Data Management | Document versioning and lineage | In Scope | Traceability across processing stages |

### 2.2 Initial Production Release Scope

The initial production release includes the implementation and operational hardening of the core architecture.

The following capabilities are required before the system can be considered production-ready:

- Reliable document ingestion and validation.
- Original document preservation.
- Processing and canonical chunk persistence.
- Metadata and lifecycle management.
- Embedding generation and index population.
- Hybrid retrieval and result fusion.
- Document-level authorization enforcement.
- Authorization revalidation before content retrieval and LLM context assembly.
- Document update, deletion, and reprocessing workflows.
- Index consistency validation and reconciliation.
- Failure handling and recovery mechanisms.
- Backup and recovery procedures.
- Security hardening and network segmentation.
- Operational monitoring and auditability.
- Deployment and recovery documentation.
- Acceptance testing against defined functional and non-functional requirements.

The completion of implementation phases alone does not establish production readiness. Production release requires passing the relevant security, reliability, operational, and acceptance gates defined in the project roadmap.

### 2.3 Future and Experimental Capabilities

The following capabilities are not mandatory for the initial production release.

They may be considered in subsequent phases after a clear business or technical requirement has been established.

| Capability | Classification | Condition |
|---|---|---|
| Distributed and multi-node deployment | Future | Requires demonstrated scaling or availability requirements |
| Kubernetes orchestration | Future | Requires operational justification |
| GPU-based production inference | Future | Requires performance and workload evaluation |
| Cloud or hybrid-cloud deployment | Out of Scope for baseline | Requires a separate architecture decision |
| Advanced Agentic Workflows | Future / Spike-PoC | Requires defined use cases and security evaluation |
| Fine-tuning and model training | Future / Spike-PoC | Requires validated business and data requirements |
| Iceberg-based Lakehouse | Spike / PoC | Requires structured-data and analytical workload justification |
| Trino integration | Spike / PoC | Requires distributed analytical query requirements |
| Hive Metastore or alternative catalog | Open Decision / Future | Relevant if a Lakehouse architecture is adopted |
| Entity-Centric Document Generation | Spike / PoC | Requires a separate use-case and feasibility assessment |
| Advanced structured-data processing | Future / Spike-PoC | Not a prerequisite for the baseline RAG architecture |

The existence of an architectural provision or storage zone does not imply that all associated capabilities must be implemented in the initial release.

In particular, the Curated Zone may be retained as an architectural provision without requiring immediate implementation of Lakehouse, Entity-Centric, or advanced analytical capabilities.

## 3. Data Ownership and Source of Truth

The architecture must maintain explicit ownership boundaries between storage and operational components.

The term **Source of Truth** must always be interpreted in relation to a specific data domain.

| Component | Authoritative Responsibility | Data Ownership |
|---|---|---|
| MinIO Data Lake | Document content and content-related artifacts | Original files, canonical chunks, and processing artifacts |
| PostgreSQL | Operational and authoritative metadata | Document metadata, ACL, lifecycle, operational state, and lineage |
| Qdrant | Vector retrieval index | Derived vector representations |
| OpenSearch | Lexical retrieval index | Derived lexical and search representations |
| Application Services | Workflow orchestration and enforcement | Processing coordination, authorization enforcement, and business logic |

### 3.1 Ownership Principles

- MinIO is the authoritative storage location for original document binaries and canonical content artifacts.
- Canonical chunks are persisted in the Processed Zone of MinIO.
- PostgreSQL is the authoritative source for document metadata, access-control information, lifecycle state, and lineage.
- Qdrant and OpenSearch contain derived, rebuildable index data.
- Search indexes must not become the authoritative source of document content or authorization decisions.
- Application services are responsible for coordinating workflows and enforcing authorization policies.

Therefore, the Data Lake is the source of truth for document content and content artifacts, but it is not the universal source of truth for every system data domain.

## 4. Accepted Architectural Decisions

The following decisions are accepted as the current architectural baseline.

### ADR-001 — Data Lake as a Foundational Architectural Layer

**Status:** Accepted

The Data Lake is a foundational architectural component responsible for preserving original documents and managing content-related artifacts across processing stages.

### ADR-002 — MinIO as the Object Storage Platform

**Status:** Accepted

MinIO is selected as the primary object storage platform for the initial single-node deployment.

It stores original document binaries, canonical chunks, and relevant processing artifacts.

### ADR-003 — PostgreSQL as the Operational Metadata Store

**Status:** Accepted

PostgreSQL is selected as the authoritative operational metadata store.

It manages document metadata, ACL information, lifecycle state, operational state, and lineage.

### ADR-004 — Separate Vector and Lexical Search Engines

**Status:** Accepted

The initial architecture uses:

- Qdrant for vector search.
- OpenSearch for lexical and BM25 search.
- Application-level fusion, initially using Reciprocal Rank Fusion (RRF).

Both search indexes are derived representations and must be rebuildable from authoritative data.

### ADR-005 — Single-Node Deployment with Docker Compose

**Status:** Accepted

The initial physical deployment architecture is single-node and container-based, using Docker Compose.

Distributed deployment and orchestration platforms are deferred unless justified by future requirements.

### ADR-006 — CPU-Only Initial Production Baseline

**Status:** Accepted

The initial production baseline is designed around CPU-only execution.

GPU-based production inference is not a baseline requirement and may be evaluated as a future architectural extension.

### ADR-SSOT-001 — Repository Documentation Governance

**Status:** Accepted — Governance Decision

The repository maintains an explicit distinction between active authoritative documents and archived or superseded documents.

The current governance baseline identifies 11 active Source of Truth documents and 7 archived documents.

This decision governs documentation authority and repository organization. It is not a technical architecture decision and must be tracked separately from architectural ADRs.

## 5. Open Architectural Decisions

The following decisions remain unresolved and must be addressed before the relevant detailed design or implementation stage.

| ID | Decision Area | Description | Responsible Role | Target Stage |
|---|---|---|---|---|
| OPEN-001 | Entity-Centric Documents | Determine whether entity-centric document generation is required and whether it belongs in a future extension | Architecture Owner / Product Owner | Future evaluation |
| OPEN-002 | Lakehouse Catalog | Determine whether a catalog such as Hive Metastore is required if Iceberg or a Lakehouse architecture is adopted | Data Engineering Owner | Spike / PoC |
| OPEN-003 | Advanced Authorization Model | Define the detailed ACL model, role hierarchy, group membership, and future identity integration | Security Owner | Detailed Design |
| OPEN-004 | Backup and Recovery | Define backup architecture, retention, recovery procedures, and RPO/RTO objectives | Infrastructure/Ops Owner | HLA completion / Detailed Design |
| OPEN-005 | Embedding and LLM Selection | Select appropriate local embedding and LLM models through evaluation and benchmarking | NLP/ML Owner | Technical Evaluation |
| OPEN-006 | Application Layer Responsibilities | Finalize responsibility boundaries across the seven application layers and the foundational Data Lake layer | Architecture Owner | HLA completion |
| OPEN-007 | Persian Text Normalization | Finalize Unicode normalization and Persian character normalization policies, including NFC/NFKC and related transformations | NLP/ML Owner | Detailed Design |
| OPEN-008 | Persian Search Analyzer | Define and validate the Persian analyzer and lexical-search configuration for OpenSearch | Search/NLP Owner | Spike / PoC |
| OPEN-009 | Chunking Strategy | Select chunking strategy, tokenizer, chunk size, overlap, and document-specific handling | NLP/ML Owner | Detailed Design / Benchmark |
| OPEN-010 | Hardware Sizing | Define CPU, RAM, storage, and capacity requirements based on expected workloads | Infrastructure/Ops Owner | HLA validation / PoC |
| OPEN-011 | Index Activation and Version Transition | Finalize atomicity, validation, rollback, and activation behavior during document version changes and reindexing | Architecture Owner / Data Engineering Owner | Detailed Design |

**Note:** The identifiers OPEN-007 through OPEN-011 are proposed identifiers. Before adoption, they must be reconciled against the repository's existing ADR and open-decision identifiers to prevent duplication or conflicting references.

### 5.1 Decision Management Rules

- Open decisions must not be presented as accepted architectural commitments.
- An open decision may be resolved through an ADR, a technical evaluation, a benchmark, or a documented PoC.
- A decision must identify an accountable owner and its expected resolution stage.
- Any decision affecting multiple architecture documents must be reflected consistently across all affected documents.
- A decision may be deferred only when its deferral does not compromise architectural integrity or production readiness.

## 6. Security and Authorization Principles

Security and authorization are foundational architectural concerns, not optional future enhancements.

### 6.1 Baseline Authorization Requirements

The initial implementation must provide:

- Authentication of users and services.
- Document-level authorization.
- Authorization-aware retrieval.
- ACL filtering in Qdrant and OpenSearch where supported.
- Authoritative authorization revalidation using PostgreSQL before fetching canonical content.
- Authorization enforcement before assembling LLM context.
- Fail-closed behavior when authorization cannot be established.

### 6.2 Responsibility Boundaries

- JWT may carry authenticated identity and claims, but token validation alone does not constitute authorization.
- MinIO IAM governs object-storage access and does not replace application-level document or chunk authorization.
- PostgreSQL remains authoritative for ACL and document authorization metadata.
- Search-engine filters provide an initial retrieval restriction but do not replace authoritative authorization revalidation.
- Application services enforce the complete authorization workflow.

Advanced RBAC, complex group hierarchies, and enterprise identity integrations may be implemented incrementally. However, the baseline document-level authorization model must exist from the initial implementation.

## 7. Architectural Governance Principles

The following principles govern architectural decisions and changes:

1. **Architecture Before Implementation:** Major implementation decisions must be supported by an approved architectural direction.
2. **Explicit Decision Tracking:** Important architectural choices must be documented and assigned an ADR or an open-decision identifier.
3. **Separation of Concerns:** Data ownership, application responsibilities, and infrastructure responsibilities must remain explicit.
4. **Single Source of Truth per Data Domain:** Each authoritative data domain must have a clearly identified owner.
5. **Security by Design:** Authorization, data protection, and auditability must be included in the baseline architecture.
6. **Offline-First Design:** Core capabilities must operate without mandatory dependency on external cloud services.
7. **Incremental Complexity:** Advanced technologies must be introduced only when justified by validated requirements.
8. **Rebuildable Derived Data:** Search indexes and other derived representations must be reproducible from authoritative data.
9. **Controlled Scope:** Future and experimental capabilities must not silently become baseline requirements.
10. **Document Consistency:** Architectural decisions must remain consistent across all authoritative project documents.

## 8. High-Level Architecture Stabilization Exit Criteria

The High-Level Architecture stage is considered complete only when all the following conditions are satisfied:

- The architectural scope and exclusions are clearly documented.
- Core Architecture Scope is distinguished from Initial Production Release Scope.
- Data ownership and Source of Truth boundaries are explicit.
- Physical and logical architecture decisions are consistent.
- Major architectural components and their responsibilities are defined.
- End-to-end write and read paths are documented.
- Document lifecycle and version activation principles are established.
- Baseline authorization and security principles are defined.
- Backup, recovery, and operational requirements are sufficiently characterized.
- Open architectural decisions are identified, assigned, and scheduled.
- No accepted ADR contradicts another accepted decision or an authoritative architecture document.
- Future, experimental, and out-of-scope capabilities are consistently reflected in the project roadmap.
- The primary architecture documents have been reviewed together for consistency.
- The architecture baseline has been formally approved.

## 9. Proposed Next Steps

The recommended sequence is:

1. Review and approve this document as Architecture Scope & Decision Register v0.2.
2. Prepare and review Architecture Requirements & Constraints.
3. Reconcile the approved High-Level Architecture documents with the scope, decisions, and requirements baseline.
4. Resolve any remaining architectural inconsistencies and document justified open decisions.
5. Establish the initial High-Level Architecture baseline.
6. Present the final document set for explicit approval.
7. Obtain separate authorization before publishing or modifying any repository files.
8. After repository publication, proceed to detailed subsystem architecture and technical design.

## 10. Document Status and Change Control

This document is a proposed architectural governance baseline.

Its approval does not automatically authorize changes to the GitHub repository.

Any repository modification, including file creation, editing, deletion, commit, or push, requires separate explicit authorization.

Changes to accepted architectural decisions must be documented, reviewed, and approved through the project's architecture governance process.