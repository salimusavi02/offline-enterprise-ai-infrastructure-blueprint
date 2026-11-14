# HLA Work Package — ARC-HLA-001
## Enterprise AI Infrastructure — High-Level Architecture
**Document ID:** ARC-HLA-001  
**Proposed Version:** 0.1  
**Review Package Revision:** 0.4 — incremental Index Set and authorization-flow refinement  
**Status:** Draft for Architecture Review  
**Review baseline date:** 2026-11-10  
**Working repository branch:** `docs/architecture-baseline-v0.1`  
**Change control:** This revision updates the review package on `docs/architecture-baseline-v0.1` only; it does not modify `main`, merge branches, or approve the HLA baseline.

---

# Part I — Architecture Input Baseline

## 1. Purpose and status

This work package translates the accepted architecture baseline and the approved HLA Blueprint into a reviewable High-Level Architecture (HLA) specification. It is deliberately technology-aware where decisions are already accepted, but it avoids turning the HLA into a container inventory, deployment runbook, detailed schema, or implementation specification.

The HLA describes:
- system boundaries and actors;
- major architectural planes and logical components;
- responsibilities, ownership, and trust boundaries;
- primary write and read paths;
- security and authorization enforcement;
- document lifecycle and index activation;
- deployment and operational constraints;
- accepted decisions, unresolved decisions, assumptions, and traceability.

It does **not** finalize unresolved implementation details such as chunk size, model selection, hardware sizing, exact APIs, detailed database schemas, backup targets, or advanced data-platform capabilities.

## 2. Architecture baseline

### 2.1 Accepted baseline

| ID | Accepted baseline | HLA consequence |
|---|---|---|
| ADR-001 | Data Lake is the foundational data layer | Depict it as a foundational layer, not merely as another storage service |
| ADR-002 | MinIO is the selected object-storage foundation | MinIO stores original documents, canonical chunks, and content artifacts |
| ADR-003 | PostgreSQL is the authoritative metadata/control store | PostgreSQL owns metadata, ACL, lifecycle, operational state, lineage, and active-version pointers |
| ADR-004 | Qdrant + OpenSearch are separate retrieval indexes | Vector and lexical retrieval remain distinct; do not portray Whoosh or a unified index as the baseline |
| ADR-005 | Single-node deployment with Docker Compose | Initial deployment view is one physical node; do not imply Kubernetes or multi-node production |
| ADR-006 | CPU-only initial production baseline | GPU is not a baseline dependency |
| ADR-SSOT-001 | Content ownership and document SSOT are governed explicitly | Every document topic has one authoritative source; HLA references related documents rather than duplicating their detailed content |

### 2.2 Cross-cutting baseline

1. On-premises and air-gapped operation; no runtime dependency on public cloud or Internet services.
2. Offline-first model and artifact lifecycle, including controlled import and integrity verification.
3. Document-level authorization with index-side pre-filtering and authoritative PostgreSQL revalidation before content fetch and context construction.
4. Fail-closed authorization: inability to validate authorization must not result in content disclosure.
5. MinIO is authoritative for original documents, canonical chunks, and content artifacts.
6. PostgreSQL is authoritative for metadata, ACL, lifecycle, operational state, lineage, and activation pointers.
7. Qdrant and OpenSearch are derived, rebuildable indexes—not canonical content stores.
8. Hybrid retrieval uses separate vector and lexical retrieval paths followed by RRF fusion.
9. Canonical chunk content is stored in MinIO Processed Zone; PostgreSQL stores references and metadata, not the canonical chunk body.
10. The initial physical deployment is single-node and CPU-only.
11. Index activation across independent systems is a controlled workflow, not a distributed atomic database transaction.
12. Document-level authorization follows ADR-007: PostgreSQL is authoritative, index ACL pre-filters are defense-in-depth, and authorization is revalidated before canonical content fetch/context construction.
13. Qdrant and OpenSearch are activated as one generation under ADR-008; PostgreSQL alone owns the active-generation pointer and hybrid queries fail closed on generation/engine inconsistency.
12. Normalization, search analysis, and chunking are distinct contracts and must not be conflated.

### 2.3 Logical architecture model

The approved model is represented as three logical planes over one foundational data layer:

- **Experience & Access Plane:** user-facing access, API boundary, identity context, request admission, and response delivery.
- **Application & Intelligence Plane:** orchestration, ingestion, parsing, NLP, embedding, retrieval, fusion, RAG, and LLM runtime.
- **Governance & Operations Plane:** authorization policy enforcement, lifecycle control, index activation, audit, observability, and operational administration.
- **Foundational Data Layer:** MinIO and PostgreSQL as authoritative stores, with Qdrant and OpenSearch as rebuildable derived indexes.

These are logical responsibility groupings, not necessarily separate hosts, networks, containers, or independently deployable services.

## 3. Known constraints and evidence limitations

- The repository was inspected through read-only GitHub connector operations. The repository metadata identifies `main` as the default branch. The previously observed baseline commit was `c03550ef51c76d20fb555ebb79965954c452e041`; this work package does not claim that this is still the latest commit.
- Key files were fetched again, including README, AGENTS, ADR-SSOT-001, the main architecture document, Data Flow, Data Lake summary, RAG tools, and SINA operational documents. The tool response was too large to preserve every fetched byte in this work package; the prior repository audit findings remain relevant.
- Recursive Tree Audit and Cross-Reference Inventory remain incomplete. Therefore file-existence, path, and cross-reference conclusions from earlier audits must be treated as provisional until those audits are completed.
- The full text of the newly approved Requirements & Constraints v0.1 was not available in this turn's repository baseline. Requirement mapping below uses known identifier ranges and topic summaries, and is explicitly **provisional**.
- The Persian Chunking design document is intentionally not modified or re-baselined here. Its detailed content is deferred to Detailed Architecture.
- ADR-007 (Baseline Document-Level Authorization) and ADR-008 (Coordinated Index Generation Activation & Rollback) are accepted in the Scope & Decision Register and are incorporated into this HLA revision. Their standalone ADR files are not required for this practical pass.

---

# Part II — HLA Chapter Traceability Matrix

## 4. Chapter-by-chapter matrix

The following matrix is the control document for authoring HLA v0.1. Requirement IDs refer to the known Requirements baseline groups: FR-001…042, DR-001…010, SEC-001…014, NFR-001…025, OPS-001…008, CON-001…015. Exact requirement-to-chapter mapping remains provisional until the full requirements text and IDs are cross-checked.

| Ch. | Chapter / objective | Key architecture questions | Required content / outputs | Requirement traceability (provisional) | ADR / OPEN mapping | Diagrams | Primary references | HLA boundary / deferred detail | Gaps / assumptions |
|---|---|---|---|---|---|---|---|---|---|
| 1 | **Executive Summary** — communicate purpose, scope, architecture shape, and key constraints | What problem does the platform solve? What is in the initial release? What are the defining constraints? | One-page overview; system capabilities; architectural summary; accepted baseline; exclusions; decision status legend | FR; NFR; CON | ADR-001…006; ADR-SSOT-001; OPEN-001…011 | D01, D02 | Architecture Requirements & Constraints; Scope & Decision Register; approved HLA Blueprint | No implementation plan, sizing, or detailed technology comparison | Executive summary must not imply that every candidate capability is committed |
| 2 | **Architecture Context, Scope & Assumptions** — establish boundaries and operating context | Who uses and administers the system? Which external systems exist? What is air-gapped? What is explicitly excluded? | Context statement; actors; external dependencies; deployment assumptions; in-scope/out-of-scope table; assumptions and constraints register | FR; SEC; NFR; CON | ADR-005, ADR-006; OPEN-001, OPEN-003, OPEN-004, OPEN-005, OPEN-006, OPEN-010 | D01 | README; Architecture Requirements; Scope & Decision Register; SINA Roadmap (context only) | No detailed network design, ports, firewall rules, or server sizing | External identity provider, source systems, and user population require confirmation |
| 3 | **Architecture Principles & Design Philosophy** — define rules that guide design and trade-offs | Which principles are mandatory? How are replaceability, offline operation, security, and data ownership enforced? | Principles with rationale and consequences; architectural invariants; accepted-vs-proposed terminology | SEC; NFR; OPS; CON | ADR-001…006; ADR-SSOT-001; OPEN-003, OPEN-006, OPEN-007…010 | D02 | Scope & Decision Register; AGENTS; ADR-SSOT-001; Architecture Requirements | No coding standards or detailed design patterns | Keep hexagonal architecture as a design principle only where it does not conflict with explicit ownership and data-flow decisions |
| 4 | **System Context & External Interactions** — define system boundary and actors | Which human and machine actors interact? Which systems provide documents, identity, models, or administration? What crosses the trust boundary? | System context narrative; actor/system catalogue; interaction descriptions; trust-boundary annotations | FR; SEC; INT (category currently unresolved); CON | OPEN-003, OPEN-005, OPEN-006 | D01, D10 | End-to-End Data Flow; Requirements; SINA operational docs | No detailed API contract, protocol version, or integration schema | Requirement group INT is defined but no INT-xxx IDs were authored; reconcile before final traceability |
| 5 | **Logical Architecture & Architectural Views** — present planes, components, and interactions | What are the logical components? Which plane owns each responsibility? What are the key interfaces and dependencies? | Plane model; component overview; responsibility catalogue; dependency rules; logical interaction narrative | FR; DR; SEC; NFR; OPS | ADR-001…006; OPEN-003, OPEN-005, OPEN-006, ADR-008 | D02, D03 | Main Architecture; Data Flow; Data Lake; Decision Register | No process topology, container list, class diagram, or detailed API | Plane boundaries are logical, not necessarily deployment boundaries |
| 6 | **Core Components & Responsibilities** — assign clear responsibility and ownership | What does each component own? Which components are authoritative? What must not be duplicated? | Component responsibility matrix; inputs/outputs; ownership; failure responsibilities; dependency constraints | FR; DR; SEC; OPS | ADR-003, ADR-004; OPEN-003, OPEN-006, ADR-008 | D03 | Main Architecture; Data Flow; Requirements | No class-level design, internal algorithms, or complete service decomposition | Component names are logical placeholders; deployment/service granularity is TBD |
| 7 | **Data Architecture & Ownership** — establish canonical data and derived-index boundaries | What is canonical? Where is each artifact stored? Which systems may be rebuilt? How are references and lineage maintained? | Data domain catalogue; system-of-record matrix; zone model; canonical chunk rule; derived-index policy; lineage overview | DR; SEC; NFR; OPS | ADR-001, ADR-002, ADR-003, ADR-004; OPEN-002, OPEN-004 | D04 | Data Lake Discussion Summary; Main Architecture; Decision Register | No bucket names, object-key format, table DDL, partition specification, or catalog implementation | Catalog and lakehouse choices remain OPEN; MinIO Processed Zone canonical chunk placement is accepted |
| 8 | **Document Ingestion & Processing Architecture** — explain secure transformation from source document to canonical artifacts | How is a document admitted, validated, parsed, normalized, chunked, embedded, and made indexable? How are failures handled? | Write-path narrative; processing stages; artifact outputs; state transitions; validation and quarantine concepts; idempotency requirements at HLA level | FR; DR; SEC; NFR; OPS | ADR-001…003; OPEN-005, OPEN-007, OPEN-009, OPEN-010 | D05, D08 | End-to-End Data Flow; Data Lake; Persian Chunking document (reference only); norm-v1.0 code as implementation evidence | No chunk size, overlap, parser library, OCR engine, normalization rule details, or worker concurrency | norm-v1.0 remains Implementation Candidate under OPEN-007; Chunking parameters are TBD/benchmark |
| 9 | **Hybrid Search & Retrieval Architecture** — describe separate retrieval paths and fusion | How do vector and lexical retrieval operate? How are candidates filtered, fused, ranked, and traced? | Query-path narrative; vector and lexical branch responsibilities; pre-filtering; RRF; retrieval result contract; failure/degraded-mode considerations | FR; DR; SEC; NFR | ADR-004; OPEN-003, OPEN-005, OPEN-007, OPEN-008, OPEN-009 | D06 | Main Architecture; Data Flow; RAG tools; Requirements | No analyzer configuration, embedding model selection, exact top-k, score thresholds, or index mappings | Qdrant + OpenSearch are baseline; Whoosh is legacy/conflicting and not baseline |
| 10 | **RAG & LLM Integration Architecture** — show grounded answer generation and model boundaries | How are retrieved chunks authorized and assembled? How is context sent to the LLM? What happens when evidence is insufficient? | RAG sequence; context construction; citation/evidence handling; model runtime boundary; refusal/not-found behavior; offline model lifecycle reference | FR; SEC; NFR; OPS | ADR-006; OPEN-005, OPEN-006, OPEN-010 | D06, D10 | Main Architecture; Data Flow; SINA LLMOps (supporting operational reference) | No prompt templates, model benchmark, quantization parameters, token budget, or model selection | LLM and embedding model choices remain OPEN; CPU-only is the initial baseline, not a performance guarantee |
| 11 | **Security & Authorization Architecture** — define trust boundaries and authorization enforcement | Where is identity established? How is document ACL enforced at retrieval and fetch time? What happens when policy or metadata is unavailable? | Security zones; identity context; ACL evaluation sequence; pre-filter plus authoritative revalidation; fail-closed rules; audit points | SEC; FR; NFR; OPS | OPEN-003, OPEN-006; cross-cutting security baseline | D07 | Requirements; Scope & Decision Register; Main Architecture; Data Flow | No detailed IAM integration, cryptographic key lifecycle, network ACLs, or policy language | PostgreSQL authoritative revalidation is mandatory before content fetch and context construction |
| 12 | **Document Lifecycle, Versioning & Index Activation** — explain controlled document and index state transitions | How do document versions progress? When is a document active? How are both indexes built and validated? How is rollback handled? | Lifecycle state model; version relationships; index readiness; activation controller; active-pointer transition; rollback and supersession concepts | FR; DR; SEC; NFR; OPS | ADR-008; ADR-003, ADR-004; OPEN-004 | D08, D09 | Data Flow; Decision Register; SINA LLMOps (only as supporting pattern) | No distributed transaction implementation, alias syntax, retry algorithm, or retention period | PostgreSQL cannot make Qdrant and OpenSearch activation one atomic transaction; use controlled workflow |
| 13 | **Integration & Communication Architecture** — identify logical interaction patterns | Which interactions are synchronous or asynchronous? How are events, commands, and references exchanged? What must be reliable? | Interaction catalogue; communication pattern map; event/command distinction; correlation and idempotency expectations | INT (unresolved); FR; NFR; OPS | OPEN-005, OPEN-006, ADR-008 | D10 | Data Flow; Main Architecture; Requirements | No final message broker, API schema, protocol, or event topic design | Do not assume a message broker until explicitly selected; synchronous/asynchronous split is to be validated |
| 14 | **Deployment & Infrastructure Architecture** — define initial physical deployment | What runs on the initial node? What are the infrastructure boundaries? What does air-gapped operation require? | Single-node deployment view; logical runtime groupings; local model/artifact stores; offline supply-chain boundary; CPU-only statement | NFR; OPS; CON; SEC | ADR-005, ADR-006; OPEN-010 | D11 | Decision Register; SINA Roadmap; SINA LLMOps; Requirements | No exact VM/container sizing, ports, OS hardening profile, HA topology, or Kubernetes | Single-node is accepted initial baseline; future scale-out requires a new decision |
| 15 | **Operational Architecture & Observability** — define operations, diagnostics, and controlled maintenance | What is monitored? What is audited? How are failures detected and recovered operationally? | Operational responsibilities; metrics/logs/traces/audit concepts; health and readiness; backup/restore domains; maintenance and offline update lifecycle | OPS; NFR; SEC | OPEN-004, OPEN-006, OPEN-010, ADR-008 | D12 | SINA RiskQA; SINA LLMOps; Requirements; Decision Register | No final alert thresholds, retention, dashboards, runbooks, or SLO values | Existing SINA operational numbers are legacy candidates, not approved project targets |
| 16 | **Reliability, Backup & Recovery** — describe resilience limits and recovery principles | Which data is authoritative and must be backed up? Which indexes can be rebuilt? What recovery ordering is required? | Failure-domain overview; backup scope by data class; restore dependency order; rebuildability matrix; DR decision register | DR; NFR; OPS; CON | OPEN-004; ADR-001…004 | D04, D11, D12 | Data Lake; SINA LLMOps; SINA RiskQA; Requirements | No RPO/RTO commitments, backup cadence, replication topology, or recovery test procedure | Existing RPO/RTO values in SINA docs are not accepted; must be re-evaluated |
| 17 | **Architecture Constraints, Risks & Open Decisions** — expose uncertainty and control future scope | What is accepted, open, candidate, deferred, or excluded? What are the material risks and decision dependencies? | Consolidated decision status; OPEN register references; risk summary; assumptions; future/experimental boundary; next design gates | All groups | ADR-001…006; ADR-SSOT-001; OPEN-001…011; RA-001…018 | D02 plus relevant diagrams | Scope & Decision Register; Requirements; Roadmap; Architecture Review findings | No closure of OPENs and no unapproved technology commitment | Ensure each OPEN has owner, expected output, exit criteria, and target phase in the controlling register |

## 5. Traceability rules

1. The matrix is **provisional** until exact requirement statements and IDs are verified against the approved Requirements document.
2. Requirement group ranges are not evidence that every requirement in a group applies to every chapter. Chapter mapping is thematic, not a substitute for requirement-level traceability.
3. ADR-001…ADR-006 are accepted in the Decision Register; standalone ADR documents are intentionally deferred.
4. OPEN items must remain visibly open in HLA. Mentioning an OPEN in the HLA does not close it.
5. Implementation evidence (for example, norm-v1.0 code) does not automatically establish architecture acceptance.
6. Existing SINA performance, backup, and operational figures are historical design inputs until reconciled with approved requirements and project constraints.
7. Any future capability—Kubernetes, multi-node, GPU, Iceberg, Trino, DuckDB, Spark, Entity-Centric Documents—must be labelled Future / Experimental / Spike / PoC unless separately accepted.

---

# Part III — Architecture Diagram Specifications

## 6. Diagram standards

All HLA diagrams should:
- use stable IDs `HLA-D01`…`HLA-D12`;
- show logical responsibilities rather than premature deployment decomposition;
- distinguish authoritative stores from derived/rebuildable indexes;
- show trust boundaries and security enforcement where relevant;
- use consistent names with the component glossary;
- label unresolved choices as `OPEN` or `TBD`;
- avoid implying distributed atomicity where none exists;
- remain readable in both English and Persian documents (diagram labels may remain English with a bilingual glossary).

## 7. Diagram catalogue and specifications

### HLA-D01 — System Context
**Purpose:** Show users, administrators, source systems, identity boundary, offline model/artifact import boundary, and the platform as one system.

**Elements:** End User; Administrator/Operator; Enterprise Document Sources; Identity Provider (if applicable, TBD); Offline Model/Artifact Import Process; Enterprise AI Platform; authorized response and audit/administration interactions.

**Flows:** document submission/import; authenticated query; response with evidence references; administrative operations; controlled offline model/artifact import.

**Trust boundaries:** enterprise user boundary; platform security boundary; offline transfer boundary. Do not depict public SaaS as a runtime dependency.

**Do not show:** internal microservices, ports, protocols, or final IAM product.

### HLA-D02 — Logical Architecture Overview
**Purpose:** Show the three planes over the foundational data layer.

**Elements:** Experience & Access Plane; Application & Intelligence Plane; Governance & Operations Plane; Foundational Data Layer. Place major capability groups in the appropriate plane.

**Key relationships:** access invokes application capabilities; governance controls authorization/lifecycle/activation and observes all planes; application reads/writes authoritative data and derived indexes through defined responsibilities.

**Important annotation:** planes are logical responsibility groupings, not deployment tiers.

### HLA-D03 — Component Architecture
**Purpose:** Show major logical components and their responsibility boundaries.

**Components:** Document Intake; Ingestion Controller; Parser/OCR Adapter; Normalization Engine; Chunking Engine; Embedding Service; Canonical Artifact Manager; Metadata & Lifecycle Service; Index Builder; Index Activation Controller; Query Processing; Hybrid Retrieval; RRF Fusion; Authorization Enforcement; RAG Orchestrator; LLM Runtime; Audit & Observability.

**Rules:** distinguish control-plane components from processing components; avoid assigning final container/service boundaries; draw dependencies, not every internal call.

### HLA-D04 — Data Ownership & Source of Truth
**Purpose:** Make canonical ownership and rebuildability unambiguous.

**Stores and ownership:**
- MinIO Raw Zone: original source documents.
- MinIO Processed Zone: canonical chunks and processed content artifacts.
- MinIO Curated Zone: only if/when its role is approved; do not imply mandatory initial use.
- PostgreSQL: metadata, ACL, lifecycle, operational state, lineage, activation pointers.
- Qdrant: derived vector index.
- OpenSearch: derived lexical/BM25 index.

**Critical annotations:** PostgreSQL does not store canonical chunk bodies; Qdrant/OpenSearch are rebuildable; canonical content is fetched from MinIO only after authoritative authorization revalidation.

### HLA-D05 — Document Ingestion Write Path
**Purpose:** Explain the document-to-canonical-artifact/index preparation path.

**Sequence:** intake → validation/quarantine → metadata registration → parsing/OCR as applicable → normalization contract → chunking contract → canonical chunk/artifact persistence in MinIO → metadata/lineage update in PostgreSQL → embedding and index build in Qdrant/OpenSearch → validation → readiness handoff to activation controller.

**Failure paths:** rejected input; parser/OCR failure; processing failure; partial index build; validation failure. Preserve prior active version.

**Do not specify:** chunk size/overlap, parser brand, OCR model, embedding model, queue product.

### HLA-D06 — Secure Retrieval & Read Path
**Purpose:** Show query processing, hybrid retrieval, authorization, context assembly, and answer generation.

**Sequence:** authenticated query → query processing → vector and lexical retrieval in parallel → index-side ACL pre-filter → candidate merge/RRF → authoritative PostgreSQL authorization revalidation → authorized canonical chunk fetch from MinIO → context construction → LLM generation → evidence-grounded response → audit/telemetry.

**Security invariant:** no canonical chunk content is fetched into the generation context before authoritative revalidation succeeds. Any authorization uncertainty fails closed.

**Failure paths:** one index unavailable; PostgreSQL authorization unavailable; no authorized evidence; LLM unavailable. Degraded behavior must not bypass authorization.

### HLA-D07 — Security & Authorization Flow
**Purpose:** Make identity context and authorization enforcement explicit.

**Elements:** identity/authentication boundary; request context; policy/ACL metadata in PostgreSQL; Qdrant/OpenSearch pre-filter; authoritative authorization revalidation; MinIO content fetch; context builder; audit trail.

**Sequence:** authenticate → derive principal/claims → pre-filter candidates → revalidate each candidate/document against authoritative ACL → permit or deny fetch → audit decision.

**Required annotations:** fail-closed; deny-by-default; no trust in index ACL alone; no direct user-to-object-store bypass.

### HLA-D08 — Document Lifecycle State Machine
**Purpose:** Show the accepted high-level lifecycle.

**States:** `RECEIVED → PROCESSING → PROCESSED → INDEXING → VALIDATING → ACTIVE`.

**Exceptional states:** `FAILED`, `REJECTED`, `SUPERSEDED`, `DELETED`.

**Transitions:** label transition guards at high level (validation success, indexing readiness, activation success, replacement accepted). Do not invent retry limits or retention periods.

### HLA-D09 — Index Versioning & Activation
**Purpose:** Explain safe activation of independently built Qdrant and OpenSearch index versions.

**Sequence:** create candidate document/index version → build vector index → build lexical index → validate both → record readiness and version metadata → activation controller evaluates readiness → update authoritative active pointer/state in PostgreSQL → serve new version → retain prior version for rollback window → retire only after explicit policy.

**Critical annotation:** not a distributed atomic transaction. Use controlled orchestration, readiness checks, recoverable state, and rollback. If one index is not ready, do not expose a partially activated version as fully active.

### HLA-D10 — Integration & Communication
**Purpose:** Catalogue interactions without prematurely choosing middleware.

**Show:** synchronous request/response interactions; asynchronous processing interactions where needed; event/command categories; correlation IDs; error/timeout boundaries; offline artifact/model import.

**Label:** transport technology `TBD` unless already accepted. Do not insert Kafka/RabbitMQ/NATS or another broker as a decision.

### HLA-D11 — Deployment Architecture
**Purpose:** Show initial physical deployment boundary.

**Show:** one on-premises physical/virtual node; containerized logical runtime groups under Docker Compose; local persistent storage for MinIO/PostgreSQL/Qdrant/OpenSearch; local model vault; offline administration/import boundary; backup target as `TBD`.

**Annotations:** CPU-only initial production baseline; air-gapped; no Kubernetes or multi-node initial production topology. The diagram is conceptual, not a sizing or network deployment drawing.

### HLA-D12 — Observability & Operations
**Purpose:** Show operational visibility and control across the platform.

**Signals:** application metrics; logs; traces; security/audit events; health/readiness; index and ingestion status; model/version metadata; backup/restore status.

**Consumers:** operations/admin; incident response; QA/evaluation; audit/compliance.

**Do not freeze:** Prometheus/Grafana/Loki/Jaeger configuration, thresholds, retention, or SLOs unless formally accepted.

---

# Part IV — HLA v0.1 Draft Content

## 8. English draft — controlled authoring text

### 8.1 Executive Summary

The Enterprise AI Infrastructure is an on-premises, air-gapped platform for enterprise document intelligence and retrieval-augmented generation (RAG), with explicit support for Persian-language document processing. Its initial production architecture is designed for a single-node, CPU-only deployment and avoids runtime dependencies on public cloud services.

The architecture separates user access, application and intelligence capabilities, governance and operations, and a foundational data layer. MinIO provides authoritative object storage for original documents, canonical chunks, and content artifacts. PostgreSQL is the authoritative store for metadata, access-control data, lifecycle state, lineage, operational state, and active-version pointers. Qdrant and OpenSearch provide separate, derived vector and lexical retrieval indexes. These indexes are rebuildable from canonical data and are not authoritative content stores.

The platform supports a controlled document-ingestion lifecycle and a secure retrieval path. Hybrid retrieval combines vector and lexical candidates through Reciprocal Rank Fusion (RRF). Document-level authorization is enforced both as an index-side pre-filter and through authoritative PostgreSQL revalidation before canonical content is fetched and included in the LLM context. Authorization uncertainty must fail closed.

This HLA records accepted architectural decisions and exposes unresolved matters without silently converting them into commitments. Detailed implementation choices—including chunking parameters, model selection, hardware sizing, exact integration protocols, and backup/recovery targets—remain subject to their respective decision and validation processes.

### 8.2 Architecture Context, Scope & Assumptions

The platform is intended for enterprise environments where documents and model artifacts are processed locally, operational data remains under organizational control, and runtime Internet connectivity is unavailable or prohibited. It provides document ingestion, processing, indexing, secure retrieval, context construction, and LLM-assisted response generation.

The initial architecture scope is limited to a single-node, Docker Compose-based, CPU-only production baseline. Multi-node deployment, Kubernetes, GPU acceleration, distributed data processing, lakehouse technologies, and entity-centric document models are not assumed to be part of the initial production release. They may be explored through explicitly scoped spikes or proofs of concept.

The architecture assumes that users and administrators are authenticated through an approved identity mechanism. The exact identity integration is not finalized in this HLA. Source-system integration patterns, user population, corpus size, performance targets, and hardware sizing require confirmation.

### 8.3 Architecture Principles

1. **Offline-first operation:** runtime capabilities must not depend on public cloud or Internet services.
2. **Explicit source of truth:** each data class has one authoritative owner.
3. **Canonical data before indexes:** derived indexes can be rebuilt from canonical artifacts.
4. **Separation of responsibilities:** logical responsibilities are explicit even if deployed together.
5. **Secure by default:** authorization is enforced before content disclosure; uncertainty fails closed.
6. **Controlled activation:** new document/index versions are validated before becoming active.
7. **Traceable decisions:** accepted decisions, open decisions, assumptions, and implementation candidates remain distinguishable.
8. **Incremental extensibility:** future capabilities require evidence and explicit scope approval.

### 8.4 System Context

The platform interacts with end users, administrators/operators, enterprise document sources, and an approved identity mechanism. Model and software artifacts are introduced through a controlled offline transfer and validation process. The platform returns generated responses grounded in authorized document evidence and records relevant operational and audit events.

No public cloud service is a required runtime dependency. Any external integration must be assessed for air-gap compatibility, identity propagation, security, and operational ownership before it is accepted.

### 8.5 Logical Architecture

The architecture is organized into three logical planes over a foundational data layer.

- The **Experience & Access Plane** provides user and administrative access, request admission, identity context propagation, and response delivery.
- The **Application & Intelligence Plane** performs document ingestion, parsing, NLP processing, embedding, index construction, query processing, hybrid retrieval, fusion, RAG orchestration, and LLM interaction.
- The **Governance & Operations Plane** provides authorization enforcement, lifecycle and activation control, audit, observability, and operational administration.
- The **Foundational Data Layer** contains authoritative object and metadata stores and the derived retrieval indexes.

These planes describe responsibility boundaries. They do not prescribe a one-to-one mapping to containers, processes, or hosts.

### 8.6 Core Components

The principal logical components are Document Intake, Ingestion Controller, Parser/OCR Adapter, Normalization Engine, Chunking Engine, Embedding Service, Canonical Artifact Manager, Metadata & Lifecycle Service, Index Builder, Index Activation Controller, Query Processing, Hybrid Retrieval, RRF Fusion, Authorization Enforcement, RAG Orchestrator, LLM Runtime, and Audit & Observability.

Each component must have a documented responsibility, input/output contract, ownership boundary, and failure behavior in Detailed Architecture. The HLA does not determine the final service decomposition.

### 8.7 Data Architecture & Ownership

MinIO is authoritative for original documents, canonical chunks, and content artifacts. Original documents are retained in the Raw Zone. Canonical processed chunks and related content artifacts are retained in the Processed Zone. The role and necessity of a Curated Zone remain subject to the approved data architecture decisions.

PostgreSQL is authoritative for document metadata, ACL information, lifecycle state, operational state, lineage, and active-version pointers. Canonical chunk bodies are not stored in PostgreSQL.

Qdrant stores derived vector representations and retrieval metadata. OpenSearch stores derived lexical/BM25 index structures. Neither index is a source of truth for canonical content. Both must be rebuildable from authoritative data and metadata.

### 8.8 Document Ingestion & Processing

A document enters through a controlled intake boundary. The platform validates the submission, registers metadata, and processes the document through parsing and OCR where applicable. Text-processing stages apply distinct contracts for normalization, chunking, and embedding preparation. Canonical chunks and content artifacts are persisted in MinIO, while metadata and lineage are recorded in PostgreSQL.

The platform then builds the required vector and lexical indexes and validates their readiness. A document version is not considered active merely because processing or one index build has completed. Detailed retry, idempotency, parser selection, OCR selection, normalization behavior, and chunking parameters are deferred to Detailed Architecture and their respective OPEN decisions.

### 8.9 Hybrid Search & Retrieval

The retrieval architecture uses separate vector and lexical search paths. Qdrant serves vector retrieval; OpenSearch serves lexical/BM25 retrieval. Candidate sets are combined through Reciprocal Rank Fusion (RRF). Index-side ACL filtering reduces unauthorized candidates, but it is not the final authorization authority.

The platform must preserve document and version references through retrieval so that candidates can be revalidated against authoritative PostgreSQL ACL and lifecycle data before canonical content is fetched.

### 8.10 RAG & LLM Integration

The RAG Orchestrator coordinates authorized evidence retrieval, context construction, and LLM invocation. Only content that has passed authoritative authorization revalidation may be included in the generation context. Responses should remain grounded in retrieved evidence, and the system must support an appropriate not-found or insufficient-evidence behavior.

The initial production baseline is CPU-only. The exact LLM, embedding model, reranker, quantization format, token budget, and evaluation thresholds remain open until benchmarked and accepted.

### 8.11 Security & Authorization

PostgreSQL is the authoritative source for document-level ACL and lifecycle authorization. Qdrant and OpenSearch ACL pre-filters are defense-in-depth optimizations; they do not replace authoritative authorization checks. The baseline permission is document-level `READ`, default-deny, with no assumed ACL inheritance.

The mandatory retrieval authorization sequence is:

1. Authenticate the user or service principal.
2. Resolve the authorization context from the trusted identity source.
3. Resolve and pin the compatible active retrieval reference from PostgreSQL.
4. Apply ACL pre-filters in Qdrant and OpenSearch.
5. Execute vector and lexical retrieval in parallel.
6. Fuse candidates using RRF.
7. **Authorization Checkpoint 1:** revalidate each candidate against current PostgreSQL ACL, document lifecycle, and active document version.
8. Fetch canonical chunks from MinIO only for candidates that pass Checkpoint 1.
9. Validate document version, chunk manifest, and content integrity/hash.
10. Rerank using authorized canonical content only.
11. **Authorization Checkpoint 2:** revalidate authorization and lifecycle/version state before context assembly.
12. Assemble context from authorized canonical chunks only.
13. Invoke the local LLM.
14. Validate and deliver the response.

The user identity key is the stable pair `issuer + subject`; service principals are distinct from human principals, and group IDs come from the trusted identity source. ACL pre-filters are not authoritative. Direct ACL revocation in PostgreSQL must be enforced at the authoritative checkpoint even if derived index metadata is stale. ACL synchronization is complete only after both indexes are updated; delayed grants may cause false negatives but must never cause disclosure.

Unavailable or indeterminate authorization authority, invalid identity context, or inability to enforce a required pre-filter must fail closed. No unauthorized content may be fetched into the reranker, assembled into context, or sent to the LLM. Identity-context freshness and advanced authorization extensions remain under OPEN-003 and Detailed Security Design.

### 8.12 Document Lifecycle, Versioning & Index Activation

The high-level document lifecycle is:

`RECEIVED → PROCESSING → PROCESSED → INDEXING → VALIDATING → ACTIVE`

Exceptional document states include `FAILED`, `REJECTED`, `SUPERSEDED`, `DELETING`, and `DELETED`.

Document identity and index membership are separate concepts. The baseline maintains `document_id`, `document_version_id`, `chunk_manifest_id`, `index_set_id`, `index_version_id`, `index_member_id`, and `pipeline_run_id`. Canonical chunks remain in MinIO Processed Zone; PostgreSQL stores their metadata/references and the authoritative operational registry.

#### ADR-008 — coordinated Index Set and incremental membership contract

Qdrant and OpenSearch are paired representations of one logical Index Set. Membership is incremental per document version, not a full-corpus snapshot. Each paired membership must reference the same `document_version_id` and `chunk_manifest_id` in both engines. Adding or updating one document affects only its corresponding membership and does not require a full-corpus rebuild.

PostgreSQL is the authoritative registry for membership, build/validation status, compatibility, and the active retrieval reference. Membership states are `BUILDING → VALIDATING → READY → ACTIVE → RETIRED`; failed build or validation enters `FAILED`. A membership can become ACTIVE only after both Qdrant and OpenSearch representations are complete and validated.

The active retrieval reference is switched in a PostgreSQL transaction. This is atomic only within PostgreSQL; no distributed transaction is assumed. A failed new build preserves the prior active membership. Query Planner consumes only ACTIVE and compatible memberships and pins the selected active reference for each query. Retired memberships are excluded from new queries and cleaned up only after in-flight references and retention requirements permit it.

Update, reindex, deletion, idempotency, and reconciliation must preserve the distinction between index-membership retirement and logical-document deletion. Detailed API/schema, physical index naming, retry/reconciliation implementation, retention values, and in-flight query draining remain Detailed Architecture work. This is an architecture-level contract, not runtime verification evidence.

## 9. خلاصه اجرایی

زیرساخت هوش مصنوعی سازمانی، بستری On-Premises و Air-Gapped برای هوشمندسازی اسناد سازمانی و تولید پاسخ مبتنی بر بازیابی (RAG) است که پشتیبانی از پردازش اسناد فارسی را نیز در نظر می‌گیرد. خط مبنای اولیه Production برای استقرار تک‌گره‌ای (Single-Node)، مبتنی بر CPU و Docker Compose طراحی شده و به سرویس‌های عمومی Cloud یا اتصال اینترنت در زمان اجرا وابسته نیست.

معماری از سه Plane منطقی شامل «تجربه و دسترسی»، «کاربرد و هوشمندی» و «حاکمیت و عملیات» بر فراز یک لایه بنیادین داده تشکیل می‌شود. MinIO مرجع نگهداری اسناد اصلی، قطعات متنی Canonical و مصنوعات محتوایی است. PostgreSQL مرجع معتبر Metadata، اطلاعات ACL، وضعیت چرخه عمر، Lineage، وضعیت عملیاتی و اشاره‌گرهای نسخه فعال است. Qdrant و OpenSearch به‌ترتیب ایندکس‌های مشتق‌شده Vector و Lexical را فراهم می‌کنند. این ایندکس‌ها منبع حقیقت محتوا نیستند و باید از داده‌های Canonical قابل بازسازی باشند.

بستر از چرخه عمر کنترل‌شده اسناد و مسیر بازیابی امن پشتیبانی می‌کند. بازیابی ترکیبی از مسیرهای مستقل Vector و Lexical استفاده می‌کند و نتایج با RRF ادغام می‌شوند. مجوزدهی در سطح سند هم به‌صورت Pre-filter در ایندکس‌ها و هم از طریق اعتبارسنجی مجدد و authoritative در PostgreSQL، پیش از Fetch محتوای Canonical و ورود آن به Context مدل، اعمال می‌شود. هرگونه ابهام یا ناتوانی در اعتبارسنجی مجوز باید به Fail-closed منجر شود.

این سند تصمیمات پذیرفته‌شده را از تصمیمات باز، فرضیات و گزینه‌های نیازمند ارزیابی جدا می‌کند. جزئیات پیاده‌سازی مانند اندازه و هم‌پوشانی Chunk، انتخاب مدل‌ها، ظرفیت سخت‌افزار، پروتکل‌های دقیق ارتباطی و اهداف Backup/Recovery در این مرحله قطعی نیستند.

## 10. زمینه، دامنه و فرضیات معماری

این بستر برای سازمان‌هایی طراحی می‌شود که پردازش اسناد و مدل‌ها باید در محدوده زیرساخت داخلی انجام شود، داده‌های عملیاتی تحت کنترل سازمان باقی بمانند و اتصال اینترنت در زمان اجرا وجود نداشته یا ممنوع باشد. قابلیت‌های اصلی شامل دریافت سند، پردازش، ایندکس‌سازی، بازیابی امن، ساخت Context و تولید پاسخ با کمک LLM است.

دامنه اولیه معماری به استقرار Production تک‌گره‌ای، مبتنی بر Docker Compose و CPU-only محدود است. استقرار چندگره‌ای، Kubernetes، شتاب‌دهی GPU، پردازش توزیع‌شده داده، فناوری‌های Lakehouse و مدل Entity-Centric جزو مفروضات انتشار اولیه نیستند و در صورت نیاز باید از مسیر Spike یا PoC و تأیید صریح عبور کنند.

فرض می‌شود کاربران و مدیران از طریق سازوکار هویت مورد تأیید سازمان احراز هویت شوند؛ اما فناوری و نحوه اتصال دقیق Identity Provider هنوز در این HLA نهایی نشده است. الگوهای اتصال به سامانه‌های مبدأ، جمعیت کاربران، حجم اسناد، اهداف عملکردی و ظرفیت سخت‌افزار نیازمند تعیین تکلیف هستند.

## 11. اصول معماری

1. **عملیات Offline-first:** قابلیت‌های زمان اجرا نباید به اینترنت یا سرویس Cloud عمومی وابسته باشند.
2. **مرجع حقیقت صریح:** هر دسته داده باید مالک و مرجع معتبر مشخص داشته باشد.
3. **داده Canonical مقدم بر Index:** ایندکس‌های مشتق‌شده باید از مصنوعات Canonical قابل بازسازی باشند.
4. **تفکیک مسئولیت‌ها:** مرز مسئولیت‌های منطقی حتی در صورت استقرار مشترک اجزا روشن باشد.
5. **امنیت به‌صورت پیش‌فرض:** مجوزدهی پیش از افشای محتوا اعمال شود و ابهام به Fail-closed منجر شود.
6. **فعال‌سازی کنترل‌شده:** نسخه جدید سند و ایندکس پیش از فعال‌شدن اعتبارسنجی شود.
7. **ردیابی‌پذیری تصمیمات:** Accepted، OPEN، Assumption و Implementation Candidate از یکدیگر متمایز بمانند.
8. **توسعه‌پذیری تدریجی:** قابلیت‌های آینده تنها با شواهد و تصویب دامنه وارد Baseline شوند.

## 12. زمینه و تعاملات سیستم

سامانه با کاربران نهایی، مدیران و اپراتورها، منابع اسناد سازمانی و سازوکار هویت مورد تأیید تعامل دارد. مدل‌ها و مصنوعات نرم‌افزاری از مسیر انتقال آفلاین کنترل‌شده و اعتبارسنجی‌شده وارد می‌شوند. سامانه پاسخ‌هایی مبتنی بر شواهد اسنادی مجاز تولید می‌کند و رویدادهای عملیاتی و امنیتی مرتبط را ثبت می‌نماید.

هیچ سرویس عمومی Cloud وابستگی ضروری زمان اجرای سامانه نیست. هر اتصال بیرونی باید از نظر سازگاری با Air-Gap، انتقال هویت، امنیت و مالکیت عملیاتی بررسی شود.

## 13. معماری منطقی

معماری از سه Plane منطقی بر فراز لایه بنیادین داده تشکیل شده است:

- **Plane تجربه و دسترسی:** دسترسی کاربران و مدیران، پذیرش درخواست، انتقال Context هویت و تحویل پاسخ.
- **Plane کاربرد و هوشمندی:** دریافت و پردازش اسناد، Parsing، NLP، Embedding، ساخت ایندکس، پردازش Query، بازیابی ترکیبی، Fusion، ارکستراسیون RAG و تعامل با LLM.
- **Plane حاکمیت و عملیات:** اعمال مجوزدهی، کنترل چرخه عمر و فعال‌سازی، Audit، Observability و مدیریت عملیات.
- **لایه بنیادین داده:** مخازن معتبر Object و Metadata و ایندکس‌های مشتق‌شده بازیابی.

این Planeها مرز مسئولیت منطقی را نشان می‌دهند و به‌معنی نگاشت یک‌به‌یک به Container، Process یا Host نیستند.

## 14. اجزای اصلی و مسئولیت‌ها

اجزای منطقی اصلی عبارت‌اند از: Document Intake، Ingestion Controller، Parser/OCR Adapter، Normalization Engine، Chunking Engine، Embedding Service، Canonical Artifact Manager، Metadata & Lifecycle Service، Index Builder، Index Activation Controller، Query Processing، Hybrid Retrieval، RRF Fusion، Authorization Enforcement، RAG Orchestrator، LLM Runtime و Audit & Observability.

مسئولیت، ورودی و خروجی، مرز مالکیت و رفتار شکست هر جزء باید در Detailed Architecture مشخص شود. این HLA درباره تفکیک نهایی Serviceها یا Containerها تصمیم نمی‌گیرد.

## 15. معماری داده و مالکیت

MinIO مرجع معتبر اسناد اصلی، قطعات Canonical و مصنوعات محتوایی است. اسناد اصلی در Raw Zone و قطعات Canonical و مصنوعات پردازش‌شده در Processed Zone نگهداری می‌شوند. نقش و ضرورت Curated Zone تا زمان تصویب معماری داده نباید به‌عنوان الزام قطعی انتشار اولیه فرض شود.

PostgreSQL مرجع Metadata، اطلاعات ACL، وضعیت چرخه عمر، وضعیت عملیاتی، Lineage و اشاره‌گر نسخه فعال است. متن Canonical قطعات در PostgreSQL ذخیره نمی‌شود.

Qdrant ساختارهای مشتق‌شده Vector و Metadata بازیابی را نگهداری می‌کند و OpenSearch ساختارهای مشتق‌شده Lexical/BM25 را. هیچ‌یک مرجع محتوای Canonical نیستند و هر دو باید از داده و Metadata معتبر قابل بازسازی باشند.

## 16. معماری دریافت و پردازش اسناد

سند از مرز Intake کنترل‌شده وارد می‌شود. سامانه ابتدا ورودی را اعتبارسنجی و Metadata آن را ثبت می‌کند؛ سپس در صورت نیاز مراحل Parsing و OCR انجام می‌شود. قراردادهای پردازش متن شامل Normalization، Chunking و آماده‌سازی Embedding باید مستقل از یکدیگر تعریف شوند. قطعات Canonical و مصنوعات محتوایی در MinIO ذخیره و Metadata و Lineage در PostgreSQL ثبت می‌شوند.

در ادامه، ایندکس‌های Vector و Lexical ساخته و آمادگی آن‌ها اعتبارسنجی می‌شود. صرف تکمیل پردازش یا ساخت یکی از ایندکس‌ها به‌معنی فعال‌شدن نسخه سند نیست. جزئیات Retry، Idempotency، انتخاب Parser/OCR، رفتار دقیق Normalization و پارامترهای Chunking به Detailed Architecture و تصمیمات باز مربوط واگذار می‌شود.

## 17. معماری بازیابی ترکیبی

معماری بازیابی از دو مسیر مستقل Vector و Lexical استفاده می‌کند. Qdrant مسئول بازیابی Vector و OpenSearch مسئول بازیابی Lexical/BM25 است. مجموعه نتایج با RRF ادغام می‌شوند. فیلتر ACL در سطح ایندکس تعداد کاندیداهای غیرمجاز را کاهش می‌دهد، اما مرجع نهایی مجوزدهی نیست.

سامانه باید شناسه سند و نسخه را در طول مسیر بازیابی حفظ کند تا پیش از Fetch محتوای Canonical، هر کاندیدا با داده معتبر ACL و Lifecycle در PostgreSQL اعتبارسنجی شود.

## 18. معماری RAG و یکپارچه‌سازی LLM

RAG Orchestrator بازیابی شواهد مجاز، ساخت Context و فراخوانی LLM را هماهنگ می‌کند. تنها محتوایی مجاز است وارد Context تولید شود که اعتبارسنجی مجدد مجوز را با موفقیت گذرانده باشد. پاسخ‌ها باید مبتنی بر شواهد بازیابی‌شده باشند و رفتار مناسب برای نبود شواهد یا ناکافی‌بودن آن‌ها تعریف شود.

خط مبنای اولیه Production مبتنی بر CPU-only است. انتخاب دقیق LLM، مدل Embedding، Reranker، فرمت Quantization، بودجه Token و آستانه‌های ارزیابی تا زمان Benchmark و تصویب باز می‌مانند.

## 19. امنیت و مجوزدهی

PostgreSQL مرجع معتبر ACL در سطح سند و وضعیت مجوزدهی چرخه عمر است. ACL Pre-filter در Qdrant و OpenSearch صرفاً به‌عنوان Defense-in-Depth و برای محدودکردن کاندیداها استفاده می‌شود و جایگزین کنترل معتبر مجوزدهی نیست. مجوز پایه `READ` در سطح سند، با سیاست Default Deny و بدون فرض وراثت ACL است.

ترتیب الزامی کنترل دسترسی در مسیر Retrieval:

1. احراز هویت کاربر یا Service Principal.
2. استخراج Authorization Context از منبع هویت معتبر.
3. دریافت و تثبیت مرجع بازیابی فعال و سازگار از PostgreSQL.
4. اعمال ACL Pre-filter در Qdrant و OpenSearch.
5. اجرای موازی Vector Retrieval و Lexical Retrieval.
6. ادغام کاندیداها با RRF.
7. **نقطه کنترل دسترسی اول:** اعتبارسنجی مجدد ACL جاری، چرخه عمر سند و نسخه فعال هر کاندیدا در PostgreSQL.
8. Fetch کردن Chunkهای Canonical از MinIO فقط برای کاندیداهای مجاز.
9. اعتبارسنجی نسخه سند، Chunk Manifest و یکپارچگی/Hash محتوا.
10. اجرای Reranking فقط بر محتوای Canonical مجاز.
11. **نقطه کنترل دسترسی دوم:** اعتبارسنجی مجدد مجوز و وضعیت چرخه عمر/نسخه پیش از ساخت Context.
12. ساخت Context فقط از Chunkهای Canonical مجاز.
13. فراخوانی Local LLM.
14. اعتبارسنجی و تحویل پاسخ.

شناسه هویت پایدار کاربر زوج `issuer + subject` است؛ Service Principal از کاربر انسانی مستقل بوده و Group ID از منبع هویت معتبر دریافت می‌شود. Pre-filter مرجع نهایی مجوزدهی نیست. لغو مستقیم ACL در PostgreSQL باید در نقطه اعتبارسنجی معتبر اعمال شود، حتی اگر Metadata ایندکس‌ها قدیمی باشد. همگام‌سازی ACL فقط پس از به‌روزرسانی هر دو Index کامل است؛ تأخیر Grant می‌تواند False Negative ایجاد کند، اما نباید موجب افشای اطلاعات شود.

در صورت عدم دسترسی یا ابهام مرجع مجوزدهی، نامعتبر بودن Identity Context یا ناتوانی در اعمال Pre-filter الزامی، سامانه باید Fail-closed عمل کند. محتوای غیرمجاز نباید Fetch شود، وارد Reranker یا Context شود یا به LLM ارسال گردد. تازگی Identity Context و توسعه‌های پیشرفته مجوزدهی ذیل OPEN-003 و Detailed Security Design باقی می‌مانند.

## 20. چرخه عمر سند، نسخه‌بندی و فعال‌سازی Index

چرخه عمر سطح بالای سند به شکل زیر است:

`RECEIVED → PROCESSING → PROCESSED → INDEXING → VALIDATING → ACTIVE`

وضعیت‌های استثنایی سند شامل `FAILED`، `REJECTED`، `SUPERSEDED`، `DELETING` و `DELETED` هستند.

هویت سند و Membership ایندکس دو مفهوم مستقل‌اند. شناسه‌های پایه شامل `document_id`، `document_version_id`، `chunk_manifest_id`، `index_set_id`، `index_version_id`، `index_member_id` و `pipeline_run_id` هستند. Chunkهای Canonical در Processed Zone از MinIO نگهداری می‌شوند؛ PostgreSQL Metadata/Reference و Registry عملیاتی معتبر را ثبت می‌کند.

#### قرارداد ADR-008 — Index Set هماهنگ و Membership افزایشی

Qdrant و OpenSearch دو نمایش جفت‌شده از یک Index Set منطقی هستند. Membership به‌صورت افزایشی و در سطح هر نسخه سند مدیریت می‌شود، نه بر مبنای Snapshot کل Corpus. هر Membership جفت‌شده در هر دو موتور باید به همان `document_version_id` و `chunk_manifest_id` ارجاع دهد. افزودن یا به‌روزرسانی یک سند فقط Membership متناظر را تحت تأثیر قرار می‌دهد و به بازسازی کل Corpus نیاز ندارد.

PostgreSQL مرجع معتبر Registry برای Membership، وضعیت Build/Validation، سازگاری و مرجع بازیابی فعال است. وضعیت‌های Membership عبارت‌اند از `BUILDING → VALIDATING → READY → ACTIVE → RETIRED` و شکست Build یا Validation به `FAILED` منتهی می‌شود. Membership فقط پس از تکمیل و اعتبارسنجی نمایش‌های هر دو موتور ACTIVE می‌شود.

مرجع بازیابی فعال در یک تراکنش PostgreSQL تغییر می‌کند. اتمیک بودن فقط در محدوده PostgreSQL است و تراکنش توزیع‌شده فرض نمی‌شود. شکست Build جدید Membership فعال قبلی را حفظ می‌کند. Query Planner فقط Membershipهای ACTIVE و سازگار را مصرف کرده و مرجع فعال انتخاب‌شده را برای هر Query تثبیت می‌کند. Membershipهای RETIRED از Queryهای جدید خارج شده و پس از پایان ارجاع Queryهای در حال اجرا و رعایت Retention پاک‌سازی می‌شوند.

فرآیندهای Update، Reindex، Deletion، Idempotency و Reconciliation باید بازنشسته‌کردن Membership را از حذف سند منطقی متمایز نگه دارند. طراحی تفصیلی API/Schema، نام‌گذاری فیزیکی Indexها، پیاده‌سازی Retry/Reconciliation، مقادیر Retention و تخلیه Queryهای در حال اجرا به Detailed Architecture واگذار می‌شود. این قرارداد در سطح معماری است و شواهد آزمون Runtime محسوب نمی‌شود.

## 21. معماری یکپارچه‌سازی و ارتباطات

HLA تعاملات منطقی را مشخص می‌کند اما Message Broker یا Protocol تأییدنشده‌ای را انتخاب نمی‌کند. در Detailed Architecture باید تعاملات به هم‌زمان و غیرهم‌زمان طبقه‌بندی و انتظارات Correlation، Idempotency، خطا و Retry تعیین شوند. فناوری ارتباطی انتخاب‌شده باید با عملیات آفلاین و Baseline تک‌گره‌ای سازگار باشد.

## 22. معماری استقرار و زیرساخت

استقرار اولیه Production به‌صورت On-Premises، Air-Gapped، Single-Node و CPU-only و بر پایه Docker Compose است. سرویس‌های داده پایدار و مصنوعات مدل به‌صورت محلی میزبانی می‌شوند. نمای استقرار مفهومی است؛ ظرفیت دقیق سرور، فضای ذخیره‌سازی، تفکیک شبکه، Portها، Baseline سخت‌سازی سیستم‌عامل و مقصد Backup هنوز تعیین نشده‌اند.

ظرفیت‌سنجی و ارزیابی عملکرد باید بر مبنای الگوی بار واقعی انجام شود، نه بر اساس فرض‌های ناشی از تک‌گره‌ای بودن استقرار. پیش از پذیرش Production، پروفایل بار نماینده باید حجم و رشد اسناد، ترکیب اندازه و قالب اسناد (از جمله سهم OCR در صورت کاربرد)، نرخ دریافت و پردازش، بار Indexing، تعداد کاربران و درخواست‌های هم‌زمان، اهداف تأخیر Retrieval و تقاضای Inference محلی مبتنی بر CPU را پوشش دهد. Benchmark باید اثر ترکیبی مصرف منابع MinIO، PostgreSQL، Qdrant، OpenSearch، بار Application و Inference محلی را روی همان Host ارزیابی کند. مقادیر عددی Latency، Throughput، Concurrency و ظرفیت سخت‌افزار تا زمان توافق بر پروفایل بار و دستیابی به شواهد Benchmark باز می‌مانند؛ این HLA هیچ SLO یا ظرفیت عددی را تصویب نمی‌کند.

Kubernetes، کلاستر چندگره‌ای و شتاب‌دهی GPU بخشی از معماری اولیه Production نیستند.

## 23. معماری عملیات و Observability

سامانه باید در حوزه‌های دریافت، پردازش، بازیابی، مجوزدهی، تولید پاسخ، فعال‌سازی Index، چرخه عمر مدل و ذخیره‌سازی دید عملیاتی فراهم کند. Metrics، Logs، Traces، رویدادهای Audit، وضعیت Health/Readiness و وضعیت Backup/Restore باید به‌اندازه کافی قابل هم‌بستگی باشند تا عیب‌یابی و پاسخ به رخداد پشتیبانی شود. برای ارزیابی عملکرد و ظرفیت، شاخص‌های سطح بار (نرخ دریافت، پردازش و ایندکس‌سازی؛ تأخیر Retrieval؛ هم‌زمانی؛ و Throughput استنتاج) باید با شاخص‌های مصرف منابع Host و Componentها (CPU، حافظه، ظرفیت و I/O ذخیره‌سازی) در MinIO، PostgreSQL، Qdrant، OpenSearch، اجزای Application و Runtime استنتاج محلی قابل هم‌بستگی باشند.

اسناد عملیاتی SINA ورودی تاریخی طراحی هستند؛ آستانه‌ها، دوره‌های نگهداری و اهداف خدماتی آن‌ها بدون تطبیق و تصویب نباید تعهد قطعی پروژه تلقی شوند.

## 24. قابلیت اطمینان، Backup و Recovery

طراحی Backup و Recovery باید داده‌های معتبر را از ایندکس‌های قابل بازسازی تفکیک کند. اسناد اصلی، مصنوعات Canonical، Metadata/ACL/Lifecycle/Lineage در PostgreSQL، مصنوعات مدل و Configuration/Secrets ویژگی‌های بازیابی و اولویت‌های متفاوتی دارند. ایندکس‌های مشتق‌شده قابل بازسازی‌اند، مشروط بر آنکه زمان بازیابی و دسترسی به منابع مبدأ قابل قبول باشد.

اهداف RPO/RTO، تناوب Backup، تفکیک رسانه، راهبرد Off-site، ترتیب Restore و تناوب آزمون بازیابی همچنان OPEN هستند و به تصویب صریح نیاز دارند. مقادیر موجود در اسناد تاریخی SINA هدف پذیرفته‌شده محسوب نمی‌شوند.

## 25. محدودیت‌ها، ریسک‌ها و تصمیمات باز

این HLA تصمیمات باز باقی‌مانده را حفظ می‌کند؛ ADR-008 با ADR-008 بسته شده است. ADR-007 خط مبنای مجوزدهی در سطح سند برای انتشار اولیه عملیاتی را تعیین می‌کند و OPEN-003 به توسعه‌های پیشرفته مجوزدهی و جزئیات محصول/تازگی Identity Provider محدود می‌شود. وابستگی‌های مهم معماری شامل مجوزدهی پیشرفته، مرزبندی مسئولیت‌های لایه Application، قراردادهای مستقل Normalization/Search Analysis/Chunking فارسی، انتخاب Embedding و LLM، ظرفیت سخت‌افزار، اهداف Backup/Recovery، انتخاب Catalog/Lakehouse، مدل Entity-Centric و فعال‌سازی کنترل‌شده Index است.

فناوری‌ها و قابلیت‌های آینده باید خارج از Baseline اولیه باقی بمانند، مگر آنکه از مسیر Spike، PoC و تصویب صریح عبور کنند. این HLA زمانی برای بازبینی معماری آماده تلقی می‌شود که شکاف‌های Traceability و Cross-Reference مخزن رفع شده یا به‌عنوان محدودیت بازبینی صریحاً پذیرفته شده باشند.

---

# Part VI — Consistency & Quality Review

## 26. Consistency checks performed in this draft

| Check | Result | Notes / action |
|---|---|---|
| Accepted decisions vs HLA narrative | PASS with controlled caveats | Single-node, CPU-only, MinIO, PostgreSQL, Qdrant + OpenSearch, RRF are reflected consistently |
| Canonical data ownership | PASS | Canonical chunks are in MinIO Processed; PostgreSQL is metadata/control only |
| Derived index semantics | PASS | Qdrant/OpenSearch are rebuildable derived indexes |
| Authorization sequence | PASS | Index pre-filter followed by PostgreSQL authoritative revalidation before fetch/context |
| Fail-closed behavior | PASS | Explicitly stated in read path and security chapter |
| Index activation atomicity | PASS | HLA explicitly avoids claiming cross-system atomic transaction |
| Document lifecycle | PASS | Uses accepted lifecycle states and exceptional states |
| Normalization vs Analyzer vs Chunking | PASS with OPENs | Distinct contracts; detailed behavior deferred |
| CPU-only / Single-node | PASS | No GPU, Kubernetes, or multi-node assumed in initial production |
| Future technology scope | PASS | Iceberg, Trino, DuckDB, Spark, Entity-Centric, Kubernetes, GPU remain future/experimental |
| Requirements traceability | PROVISIONAL | Exact requirement statements/IDs not available in full for this authoring pass |
| Repository recursive inventory | OPEN | Recursive Tree Audit incomplete |
| Cross-reference integrity | OPEN | Cross-Reference Inventory incomplete |
| INT requirement category | OPEN | Category exists but no INT-xxx IDs were defined |
| Legacy SINA operational targets | CONTROLLED / NOT ACCEPTED | Must not become project commitments by repetition |
| Bilingual semantic parity | PROVISIONAL PASS | Persian draft mirrors the English baseline; detailed terminology review remains |

## 27. Material risks and review points

1. **Requirement traceability risk:** without the full requirement statements, the chapter mapping is thematic. Do not treat it as a verified requirement compliance matrix.
2. **Authorization design risk:** the high-level sequence is fixed, but identity source, group/role mapping, ACL schema, revocation latency, and cache policy remain OPEN.
3. **Index activation risk:** failure between separate index systems can leave candidate versions partially built. Activation controller and recovery state machine require detailed design.
4. **Canonical artifact integrity risk:** object versioning, checksum policy, immutability, retention, and deletion semantics require definition.
5. **Offline supply-chain risk:** model, container image, OS package, and security-feed imports need a controlled provenance and integrity workflow.
6. **Capacity risk:** CPU-only single-node operation may constrain throughput and latency. No sizing or performance commitment should be inferred until workload characterization and benchmarks are complete.
7. **Persian quality risk:** normalization, search analyzer, chunking, OCR, and embedding quality must be evaluated independently and end-to-end.
8. **Operational target risk:** historical SINA RPO/RTO, latency, recall, and concurrency values conflict or lack approval; they must not be copied into HLA as commitments.
9. **Scope creep risk:** advanced lakehouse, entity-centric, distributed, and GPU capabilities can expand scope materially; keep them outside the initial baseline.
10. **Documentation governance risk:** README, AGENTS, PROJECT_RULES, ADR-SSOT-001, and Archive index responsibilities need reconciliation before repository publication.

## 28. Decisions deliberately not made

- No closure of OPEN-001…ADR-008.
- No acceptance of norm-v1.0; it remains an Implementation Candidate under OPEN-007.
- No acceptance of exact chunk size/overlap.
- No selection of a final Persian search analyzer.
- No selection of embedding, LLM, or reranker models.
- No hardware sizing or performance SLO commitment.
- No RPO/RTO or backup cadence commitment.
- No message broker or API protocol selection.
- No detailed IAM/identity provider selection.
- No acceptance of Iceberg, Trino, DuckDB, Spark, Entity-Centric, Kubernetes, multi-node, or GPU for initial production.
- No creation of standalone ADR-001…ADR-006 documents.
- No GitHub changes of any kind.

## 29. Recommended review sequence for tomorrow

1. Confirm the logical architecture model (three planes + foundational data layer).
2. Review source-of-truth and ownership boundaries, especially MinIO vs PostgreSQL vs derived indexes.
3. Review the secure retrieval sequence and fail-closed enforcement.
4. Review index activation and rollback semantics.
5. Confirm whether the HLA scope and exclusions accurately reflect the intended initial production release.
6. Review the 12 diagram specifications and approve or amend the diagram catalogue.
7. Review requirement traceability after the complete Requirements text is available.
8. Decide which OPEN items should be resolved during Detailed Architecture and which require separate spikes/PoCs.
9. Only after content approval, prepare repository-specific file proposals for separate authorization.

---

# Part VII — Remaining Work / Readiness Statement

## 30. Work package status

| Workstream | Status | Evidence / limitation |
|---|---|---|
| HLA-01 Architecture Input Baseline | Drafted | Based on accepted baseline and read-only repository context |
| HLA-02 Chapter Traceability Matrix | Drafted — provisional | Exact requirement-level mapping awaits full Requirements text |
| HLA-03 Diagram Specifications | Drafted | 12 diagrams specified at logical level |
| HLA-04 English HLA v0.1 | Drafted | Complete controlled narrative across 17 chapters |
| HLA-05 Persian HLA v0.1 | Drafted | Parallel Persian narrative; terminology review still needed |
| HLA-06 Consistency Review | Initial pass complete | Findings and unresolved gaps recorded |
| Recursive Tree Audit | Not complete | Must be completed before repository documentation changes |
| Cross-Reference Inventory | Not complete | Must be completed before renames/archive/index updates |
| Final bilingual editorial review | Pending | To be done with user review |
| HLA stabilization candidate | Pending user review | No content approval is implied by this draft |

## 31. Readiness statement

**Current status: HLA v0.1 review package revision 0.3 — Draft for Architecture Review, with GAP-006 and GAP-007 architecture contracts incorporated.**

This is a complete first-pass architecture narrative and diagram specification package, not a final approved architecture. The principal architecture invariants have been carried through consistently. The remaining blockers are not hidden: exact requirement-level traceability, repository-wide recursive inventory, cross-reference validation, bilingual editorial review, and user decisions on scope and unresolved OPEN items.

This revision is committed only to `docs/architecture-baseline-v0.1`. It does not approve the HLA baseline, establish implementation/test evidence, or authorize a merge to `main`.