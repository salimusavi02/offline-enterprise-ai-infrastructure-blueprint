# Enterprise AI Infrastructure: Document Intelligence & RAG Ecosystem

**Version:** 2.0 (Proposed)  
**Status:** Proposed Architecture — Pending Document Approval  
**Deployment Model:** Single-Node  
**Environment:** Air-Gapped / Offline-First  
**Primary Languages:** Persian / English

---

## 1. Executive Summary

This document defines the target architecture for an offline enterprise Document Intelligence and Retrieval-Augmented Generation (RAG) platform.

The platform ingests enterprise documents, extracts and processes their content, generates canonical chunks and embeddings, maintains independent lexical and vector indexes, and provides permission-aware retrieval for local LLM inference.

The initial deployment is intentionally Single-Node to reduce operational complexity. The architecture nevertheless establishes clear logical boundaries and technology-independent interfaces to support future evolution.

The architecture is based on the following approved decisions:

| Area | Approved Decision |
|---|---|
| Deployment | Single-Node |
| Object Storage / Data Lake foundation | MinIO |
| Operational Metadata | PostgreSQL |
| Vector Search | Qdrant |
| Lexical Search (BM25) | OpenSearch |
| Canonical Chunk Storage | MinIO — Processed Zone |
| Hybrid Fusion | Reciprocal Rank Fusion (RRF) |
| LLM and Embedding | Local runtimes |
| Deployment Orchestration | Docker Engine / Docker Compose |

MinIO, PostgreSQL, Qdrant, and OpenSearch have distinct responsibilities. Search indexes are derived artifacts and must be reproducible from canonical data and versioned processing artifacts.

## 2. Architecture Principles

1. **Offline-first:** Runtime operation must not depend on public internet connectivity.
2. **Single source of truth:** Canonical source files and processed artifacts are maintained in MinIO.
3. **Separation of responsibilities:** Object storage, operational metadata, and search indexes are separate architectural concerns.
4. **Rebuildability:** Vector and lexical indexes must be rebuildable from canonical processed artifacts.
5. **Versioned processing:** Document, processing, chunking, embedding, and index versions must be traceable.
6. **Permission-aware retrieval:** No document content may enter an LLM context without authorization validation.
7. **Idempotent workflows:** Retried jobs must not create inconsistent duplicate artifacts or records.
8. **Controlled activation:** An incomplete document or index version must never become active.
9. **Observable operations:** Processing, retrieval, failures, and administrative actions must be auditable.
10. **Incremental evolution:** The initial deployment must avoid unnecessary distributed-system complexity.

## 3. Logical Architecture

The platform is divided into the following logical layers.

### 3.1 Client and Access Layer

Responsibilities:
- User-facing application and API clients
- Authentication
- Request validation
- Session and identity context propagation

### 3.2 API and Application Layer

Responsibilities:
- Document upload and management APIs
- Query and response APIs
- Administrative endpoints
- API-level authorization and rate control

### 3.3 Domain and Orchestration Layer

Responsibilities:
- Document lifecycle management
- Pipeline orchestration
- Workflow state transitions
- Retry and recovery coordination
- Version activation and deactivation

### 3.4 Processing and Retrieval Layer

Logical service boundaries:

- Ingestion Service
- Document Processing Service
- Normalization and Chunking Service
- Embedding Service
- Indexing Service
- Query Planning Service
- Hybrid Retrieval Service
- Reranking Service
- Context Assembly Service
- Local LLM Service

These are logical responsibilities, not a requirement to deploy every component as a separate microservice or container.

### 3.5 Data and Persistence Layer

This layer contains three distinct persistence responsibilities:

**A. Data Lake / Object Storage — MinIO**

Stores:
- Original source documents
- Extracted content
- Normalized documents
- Canonical chunks
- Processing manifests
- Curated business artifacts

**B. Operational Storage — PostgreSQL**

Stores:
- Document and version metadata
- Processing workflow state
- ACL and authorization metadata
- Data lineage
- Index registry
- Pipeline execution records
- Outbox events
- Audit metadata

PostgreSQL does not own the canonical chunk text.

**C. Search Index Storage**

- Qdrant: vector representations and retrieval metadata
- OpenSearch: lexical searchable content and BM25 indexes

Both search systems are derived stores. They are not authoritative repositories for document lifecycle or canonical content.

### 3.6 Cross-Cutting Capabilities

- Identity and access management
- Audit logging
- Monitoring and alerting
- Configuration and secrets management
- Backup and recovery
- Model and artifact version management

## 4. Data Lake and Operational Storage Architecture

### 4.1 MinIO — Data Lake Foundation

MinIO provides S3-compatible object storage and is the persistence foundation for the Data Lake.

The Data Lake is organized into three logical zones:

**Raw Zone**
- Original uploaded documents
- Immutable source artifacts
- Original binary content
- Source and ingestion metadata references

**Processed Zone**
- Extracted text
- Normalized text
- Canonical chunks
- Chunk manifests
- Processing outputs
- Versioned intermediate artifacts

**Curated Zone**
- Business-ready entity documents
- Curated datasets
- Structured artifacts prepared for downstream consumption

The zones are logical data-governance boundaries. Their physical implementation may use separate buckets or prefixes according to operational and security requirements.

### 4.2 PostgreSQL — Operational Storage

PostgreSQL is responsible for operational consistency and lifecycle coordination.

Core logical entities include:

- `sources`
- `documents`
- `document_versions`
- `artifacts`
- `pipeline_runs`
- `pipeline_steps`
- `chunk_manifests`
- `access_policies`
- `groups`
- `document_acl`
- `index_registry`
- `outbox_events`
- `audit_events`

PostgreSQL stores references to MinIO objects, not the authoritative document or canonical chunk payloads.

### 4.3 Canonical Data Ownership

| Data | Authoritative Location |
|---|---|
| Original document | MinIO Raw Zone |
| Extracted document | MinIO Processed Zone |
| Normalized document | MinIO Processed Zone |
| Canonical chunks | MinIO Processed Zone |
| Document metadata | PostgreSQL |
| Workflow state | PostgreSQL |
| ACL and policy metadata | PostgreSQL |
| Vector index | Qdrant |
| BM25 index | OpenSearch |
| Index lifecycle metadata | PostgreSQL |

A search result must be traceable to a canonical `chunk_id` and its corresponding MinIO artifact.

## 5. Hybrid Search Architecture

The platform uses two independent search engines.

### 5.1 Vector Search — Qdrant

Responsibilities:
- Store embeddings
- Execute semantic similarity searches
- Support metadata filtering
- Associate vectors with canonical chunk identifiers
- Support index versioning and rebuild operations

Qdrant payload should contain only the metadata required for filtering, ranking, and traceability. Canonical chunk content remains in MinIO.

### 5.2 Lexical Search — OpenSearch

Responsibilities:
- Maintain searchable text indexes
- Execute BM25 queries
- Support language-aware analysis
- Support metadata and ACL filtering
- Maintain independent index versions

Persian analyzer design, including ZWNJ handling, stemming, and tokenization, requires a dedicated benchmark and validation phase.

### 5.3 Hybrid Fusion

The initial fusion strategy is Reciprocal Rank Fusion (RRF).

The retrieval workflow:

1. Execute vector search in Qdrant.
2. Execute lexical search in OpenSearch.
3. Normalize and combine ranked results through RRF.
4. Validate document state and authorization.
5. Apply reranking if enabled.
6. Retrieve canonical chunks from MinIO.
7. Assemble authorized context for LLM inference.

The two indexes must refer to compatible document and processing versions.

## 6. Document and Index Lifecycle

Each document may have multiple versions.

A document version moves through a controlled lifecycle:

`RECEIVED → PROCESSING → PROCESSED → INDEXING → VALIDATING → ACTIVE`

Additional terminal or exceptional states may include:

- `FAILED`
- `REJECTED`
- `SUPERSEDED`
- `DELETED`

### 6.1 Activation Requirements

A new document version must not become active until:

- Original and processed artifacts are persisted.
- Canonical chunk manifest is valid.
- Metadata and lineage are committed.
- Embeddings are generated successfully.
- Qdrant indexing is validated.
- OpenSearch indexing is validated.
- ACL metadata is available and consistent.
- Index and artifact version compatibility is confirmed.

Activation must be a controlled state transition coordinated through PostgreSQL.

### 6.2 Previous Version Preservation

The currently active version remains available while a replacement is being processed and indexed.

After successful validation:
- The new version becomes active.
- The previous version becomes superseded.
- Retrieval is routed to the active version.

If the new version fails validation, the previous active version remains available.

## 7. ACL and Security Architecture

Authorization is enforced at multiple points.

### 7.1 Authorization Flow

1. Authenticate the user.
2. Resolve user identity and group memberships.
3. Build the authorization context.
4. Apply ACL constraints to Qdrant and OpenSearch queries where supported.
5. Fuse candidate results.
6. Revalidate authorization against authoritative PostgreSQL metadata.
7. Verify document lifecycle state.
8. Fetch canonical chunks from MinIO only after authorization succeeds.
9. Include only authorized content in the LLM context.

### 7.2 Fail-Closed Requirement

If authorization metadata is missing, unavailable, inconsistent, or cannot be validated, access must be denied.

Post-retrieval filtering alone is insufficient if unauthorized content can enter reranking, context assembly, logs, or LLM prompts.

ACL changes must be traceable and propagated to both search indexes. Until synchronization is confirmed, authoritative authorization checks must prevent exposure.

## 8. Logical Service Boundaries and Architecture Style

The internal application architecture should follow Hexagonal Architecture (Ports and Adapters).

Examples of ports:

- `ObjectStoragePort`
- `MetadataRepositoryPort`
- `VectorSearchPort`
- `LexicalSearchPort`
- `EmbeddingPort`
- `LLMInferencePort`
- `AuthorizationPort`

Adapters implement these interfaces for MinIO, PostgreSQL, Qdrant, OpenSearch, and local model runtimes.

Hexagonal Architecture is an internal design pattern, not an additional deployment layer.

## 9. Physical Deployment Architecture

The first deployment consists of one Linux host running Docker Engine and Docker Compose.

Expected logical workloads:

- Reverse Proxy / API Entry
- API Application
- Ingestion Worker
- Processing Worker
- Embedding Runtime
- Indexing Worker
- Local LLM Runtime
- PostgreSQL
- MinIO
- Qdrant
- OpenSearch
- Monitoring and Logging
- Backup Jobs

The logical services may be consolidated into fewer containers during the initial implementation.

Redis, RabbitMQ, and a separate distributed workflow engine are not mandatory in the first deployment. Their introduction requires a demonstrated operational need.

### 9.1 Network Segmentation

Logical networks:

- `frontend_net`: controlled client-facing access
- `backend_net`: internal application and data services

Database and search service ports must not be directly exposed to user networks.

Administrative interfaces must be restricted to authorized management access.

### 9.2 Persistent Volumes

Expected persistent data areas:

- MinIO data
- PostgreSQL data
- Qdrant data
- OpenSearch data
- Model Vault
- Application configuration
- Application logs

Secrets must be managed separately from ordinary configuration and protected at rest.

## 10. Backup and Recovery

Backup responsibilities must cover:

- MinIO source and processed artifacts
- PostgreSQL metadata and operational state
- Qdrant and OpenSearch snapshots or rebuild capability
- Model artifacts and model manifests
- Configuration and deployment definitions
- Secrets, through an appropriately protected mechanism

A backup stored only on the primary host is not sufficient.

RPO, RTO, retention periods, and recovery procedures remain operational decisions requiring explicit approval.

Restore tests are mandatory. A backup that has not been successfully restored is not considered validated.

## 11. Observability and Auditability

The platform should capture:

- Document ingestion events
- Processing execution and failures
- State transitions
- Index build and activation events
- Retrieval traces
- Authorization decisions
- Model and embedding versions
- Administrative actions
- Backup and restore outcomes

Logs must avoid unnecessary exposure of sensitive document contents and user queries.

## 12. Single-Node Limitations

The first deployment is a controlled initial architecture, not a highly available cluster.

Known limitations:

- Single point of failure
- Shared CPU, RAM, and storage contention
- Limited independent scaling
- Maintenance may require service interruption
- Host-level failure can affect all platform services

Future evolution may introduce separate worker nodes, replicated storage, and clustered search services. These are outside the initial deployment scope.

## 13. Open Architecture Decisions

The following items require validation before implementation baselines are frozen:

| ID | Topic | Required Outcome |
|---|---|---|
| OPEN-001 | Unicode normalization | Approve NFC/NFKC and explicit Persian mapping contract |
| OPEN-002 | Chunking | Benchmark chunk size, overlap, and boundary strategy |
| OPEN-003 | Tokenizer | Select and version Persian/multilingual tokenizer |
| OPEN-004 | ACL model | Define users, groups, inheritance, and document-level policies |
| OPEN-005 | PostgreSQL schema | Finalize constraints, indexes, and lifecycle model |
| OPEN-006 | OpenSearch analysis | Validate Persian analyzer and BM25 behavior |
| OPEN-007 | Resource sizing | Benchmark CPU, RAM, SSD, and concurrency |
| OPEN-008 | Backup objectives | Approve RPO, RTO, retention, and recovery design |

## 14. Architecture Decision Records

The following decisions are approved for the initial phase:

- ADR-001: Single-Node deployment
- ADR-002: MinIO as Data Lake object storage foundation
- ADR-003: Separate Qdrant and OpenSearch search engines
- ADR-004: Canonical chunks stored in MinIO Processed Zone; PostgreSQL stores metadata and lineage

Additional technology and implementation decisions must be recorded separately and must not be treated as approved merely because they appear as examples in this document.

---

**Document Status:** Proposed for review.  
**No implementation or repository changes are authorized by this document alone.**