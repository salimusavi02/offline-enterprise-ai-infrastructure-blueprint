# End-to-End Data Flow — Document Intelligence & RAG

**Version:** 2.0 (Proposed)  
**Status:** Proposed — Pending Document Approval  
**Architecture:** Single-Node / Offline-First  
**Persistence:** MinIO + PostgreSQL + Qdrant + OpenSearch

---

## 1. Purpose

This document defines the end-to-end data lifecycle for the enterprise Document Intelligence and RAG platform.

It describes:

- Document ingestion and processing
- Canonical artifact persistence
- Metadata and lineage management
- Embedding and hybrid indexing
- Version activation
- Query and retrieval
- ACL enforcement
- Document update and deletion
- Failure recovery and reconciliation

The central architectural rule is:

**MinIO stores canonical document artifacts and chunks; PostgreSQL coordinates operational state and metadata; Qdrant and OpenSearch maintain derived search indexes.**

## 2. Data Ownership

| Data | Authoritative System |
|---|---|
| Original binary document | MinIO / Raw |
| Extracted content | MinIO / Processed |
| Normalized content | MinIO / Processed |
| Canonical chunks | MinIO / Processed |
| Document metadata | PostgreSQL |
| Workflow state | PostgreSQL |
| ACL metadata | PostgreSQL |
| Vector index | Qdrant |
| Lexical index | OpenSearch |
| Index lifecycle registry | PostgreSQL |

Search indexes must be reproducible from canonical artifacts and versioned processing metadata.

## 3. Write Path — Document Ingestion

### Step 1: Document Submission

A document enters through an authorized upload API or an approved ingestion mechanism.

Initial checks:

- Authentication and authorization
- File size limit
- Allowed MIME types
- File extension and content consistency
- Malware scanning using locally maintained definitions
- Filename and metadata validation

Rejected documents must not enter the normal processing pipeline.

### Step 2: Content Identification

The system calculates a SHA-256 hash over the original binary content.

The hash is used for exact duplicate detection and traceability.

The ingestion process must be idempotent. Repeated submission of the same content must not create uncontrolled duplicate processing records.

A document identity and document version identity must be distinguished:

- `document_id`: logical document identity
- `document_version_id`: immutable content version identity

### Step 3: Raw Artifact Persistence

The original binary file is persisted in MinIO Raw Zone.

The object reference, checksum, source metadata, and ingestion information are recorded in PostgreSQL.

The original artifact must remain available according to the retention policy.

### Step 4: Document Registration

PostgreSQL creates or updates the relevant records:

- Document
- Document version
- Source reference
- Initial lifecycle state
- ACL reference
- Ingestion metadata
- Pipeline execution record

The document version enters `RECEIVED` or `PROCESSING`.

### Step 5: Parsing and Extraction

The processing pipeline extracts document content and structural information.

Possible input formats include:

- PDF
- DOCX
- TXT
- Scanned documents and images, where OCR is enabled

Extracted content should preserve useful structural information:

- Page boundaries
- Section hierarchy
- Tables and captions
- Document metadata
- Source locations

Parser failures must be recorded against the relevant document version.

### Step 6: Normalization

The extracted content passes through a versioned normalization pipeline.

The normalization contract must define:

- Unicode normalization form
- Persian/Arabic character mapping
- ZWNJ policy
- Whitespace handling
- Control character handling
- RTL and logical text ordering
- Language detection or classification

The exact NFC/NFKC policy remains an open architecture decision until formally approved.

Normalization must not silently discard meaningful content or change logical text order.

Each processed artifact must record the normalization implementation and version.

### Step 7: Chunking

The normalized content is divided into canonical chunks.

Chunking should consider:

- Document structure
- Section and paragraph boundaries
- Sentence boundaries
- Token limits
- Context preservation
- Persian and multilingual text characteristics

The initial candidate parameters may be benchmarked around 512 tokens and 50-token overlap, but these values are not final until validated against representative documents and the selected tokenizer.

Every chunk must have a stable identifier within its document version and retain source-location metadata.

### Step 8: Processed Artifact Persistence

The following artifacts are stored in MinIO Processed Zone:

- Extracted document
- Normalized document
- Canonical chunk files
- Chunk manifest
- Processing metadata

The chunk manifest provides the authoritative mapping between:

- `chunk_id`
- `document_id`
- `document_version_id`
- `artifact_id`
- Chunk order
- Content hash
- Source location
- Processing and chunking versions

PostgreSQL stores references and metadata for these artifacts, not the canonical chunk text.

### Step 9: Metadata and Lineage Commit

PostgreSQL records:

- Artifact references
- Processing run
- Processing version
- Chunk manifest reference
- Document lifecycle state
- Source lineage
- ACL information

A successful processing run must be distinguishable from a partially completed or failed run.

### Step 10: Embedding Generation

The embedding worker reads canonical chunks from MinIO and generates vectors using a local embedding runtime.

Each embedding operation must record:

- Model identifier
- Model version
- Model artifact hash
- Embedding dimension
- Preprocessing version
- Processing run
- Target index version

A failed embedding operation must not permit document activation.

### Step 11: Vector Indexing

Generated vectors and the required retrieval metadata are written to Qdrant.

The payload should include:

- `chunk_id`
- `document_id`
- `document_version_id`
- `index_version_id`
- Language
- Required ACL filtering attributes
- Relevant source metadata

Canonical chunk text remains in MinIO.

### Step 12: Lexical Indexing

The canonical chunk content and required metadata are indexed in OpenSearch.

OpenSearch maintains the BM25 searchable representation.

Persian analysis must be implemented through a versioned analyzer configuration. Analyzer changes may require a new index build.

### Step 13: Index Validation

Before activation, both search systems must pass validation.

Validation includes:

- Expected chunk count
- Missing and duplicate identifiers
- Document and version consistency
- Embedding dimension compatibility
- Search index availability
- ACL metadata consistency
- Successful retrieval of test records
- Index version compatibility

A successful write response alone is not sufficient evidence of a valid index.

### Step 14: Document Version Activation

The document version becomes active only after all required artifacts, metadata, and indexes are validated.

The activation operation must be coordinated through PostgreSQL.

The system must ensure that the active version references a consistent set of:

- Processed artifacts
- Chunk manifest
- Embedding model
- Qdrant index
- OpenSearch index
- ACL metadata

If validation fails, the new version remains inactive and the previous active version remains available.

## 4. Workflow State Model

A proposed lifecycle:

`RECEIVED → PROCESSING → PROCESSED → INDEXING → VALIDATING → ACTIVE`

Exceptional states:

- `FAILED`
- `REJECTED`
- `SUPERSEDED`
- `DELETING`
- `DELETED`

State transitions must be explicit and validated.

Only authorized workflow operations may change lifecycle state.

A failed step must not cause a document version to be marked active.

## 5. Event Delivery and Consistency

PostgreSQL is the authoritative source for operational workflow state.

Where asynchronous processing is used, state changes and event publication should be coordinated using a Transactional Outbox pattern.

The Outbox provides:

- Durable event records
- Retryable event delivery
- Duplicate-safe processing
- Traceability between state transitions and downstream jobs

Workers must implement idempotent operations.

A distributed message broker is not mandatory for the initial Single-Node deployment. The initial implementation may use a database-backed job mechanism, provided that durability, retry, and recovery requirements are met.

The platform follows controlled eventual consistency for derived indexes.

A document version must not become active merely because one index has completed.

## 6. Read Path — Query and RAG Retrieval

### Step 1: Query Submission

The user submits a query through the application API.

The request includes the authenticated identity and relevant session context.

### Step 2: Authentication

The system validates the user identity and establishes the request security context.

Unauthenticated requests are rejected.

### Step 3: Authorization Context

The system resolves:

- User identity
- Group memberships
- Applicable roles
- Document access policies
- Relevant administrative restrictions

The resulting authorization context is used throughout retrieval.

### Step 4: Query Preparation

The query passes through the approved query normalization process.

Optional operations may include:

- Language detection
- Query rewriting
- Synonym expansion
- Tokenization
- Embedding generation

Query normalization must be compatible with the relevant document and index normalization contracts.

### Step 5: Parallel Search

The Query Planner executes two independent retrieval operations:

**Vector Search**
- Generate query embedding.
- Search Qdrant.
- Apply supported metadata and ACL filters.

**Lexical Search**
- Analyze the query.
- Execute BM25 search in OpenSearch.
- Apply supported metadata and ACL filters.

Both searches must target compatible active index versions.

### Step 6: Hybrid Fusion

The results are merged using Reciprocal Rank Fusion (RRF).

The fusion stage must preserve:

- Canonical chunk identifiers
- Document and version identifiers
- Index version references
- Retrieval provenance

Duplicate candidates are consolidated according to the approved deduplication policy.

### Step 7: Authorization Revalidation

Before any canonical chunk content is retrieved, the system validates each candidate against authoritative PostgreSQL metadata.

Validation includes:

- User and group permissions
- Document ACL
- Document lifecycle state
- Active document version
- Revocation or deletion state

If authorization cannot be verified, the candidate must be excluded.

If the authorization service or required metadata is unavailable, the operation must fail closed.

### Step 8: Reranking

Authorized candidates may be passed to a local reranking model.

Only authorized candidate content may be supplied to the reranker.

Reranking must preserve provenance and document-version references.

### Step 9: Canonical Chunk Retrieval

The system retrieves the canonical chunk artifacts from MinIO Processed Zone using their identifiers and manifest references.

The retrieved content must match the expected content hash and document version.

Missing or mismatched artifacts must not be silently substituted.

### Step 10: Context Assembly

The Context Assembly component prepares the final LLM context.

It must enforce:

- Authorization
- Context length limits
- Duplicate removal
- Source attribution
- Document version consistency
- Relevant metadata preservation

Only validated, authorized chunks may enter the LLM prompt.

### Step 11: Local LLM Inference

The assembled context and user query are sent to the local LLM runtime.

The inference process must not depend on external model APIs or internet connectivity.

### Step 12: Response Validation and Delivery

The response is checked according to configured application policies.

The API returns the answer and appropriate source references.

Audit records should include relevant retrieval and model provenance without unnecessarily storing sensitive document content.

## 7. ACL Enforcement Requirements

ACL must be enforced at multiple stages:

1. User identity resolution
2. Pre-filtering in Qdrant and OpenSearch
3. Authoritative PostgreSQL validation after candidate fusion
4. Authorization before fetching canonical content
5. Authorization before context assembly

The index-level ACL filters are an optimization and defense-in-depth mechanism. PostgreSQL remains authoritative.

A delayed ACL synchronization must never result in unauthorized content exposure.

For restrictive ACL changes, authoritative checks must take effect before index synchronization is considered complete.

## 8. Document Update

When a document changes:

1. Register a new immutable document version.
2. Preserve the existing active version.
3. Store the new original file in Raw Zone.
4. Execute parsing and processing.
5. Store new processed artifacts and chunk manifest.
6. Generate embeddings.
7. Build the corresponding Qdrant and OpenSearch index entries.
8. Validate all artifacts and indexes.
9. Activate the new version.
10. Mark the previous version as superseded.

The old version remains available until the new version passes all activation checks.

Incremental chunk-level updates may be introduced after correctness and consistency are demonstrated. The initial implementation should prioritize deterministic full-version processing.

## 9. Document Deletion

Deletion must be coordinated across operational metadata and derived indexes.

Proposed sequence:

1. Record a deletion request and tombstone in PostgreSQL.
2. Prevent the document from being returned by retrieval.
3. Remove or invalidate corresponding Qdrant records.
4. Remove or invalidate corresponding OpenSearch records.
5. Record completion and failures.
6. Apply MinIO retention and physical deletion policies.
7. Preserve required audit evidence.

Deletion must be retryable and idempotent.

The system must distinguish logical deletion, retrieval exclusion, and physical object deletion.

## 10. Reprocessing and Reindexing

Reprocessing may be triggered by:

- Parser changes
- Normalization changes
- Chunking changes
- Tokenizer changes
- Embedding model changes
- Search analyzer changes
- Index schema changes

Each reprocessing run must have a distinct pipeline version and traceable output artifacts.

A new index version should be built independently and validated before activation.

Reindexing must not corrupt the currently active index.

## 11. Failure Handling

| Failure | Required Behavior |
|---|---|
| MinIO unavailable | Stop dependent processing; retry safely |
| PostgreSQL unavailable | Stop new lifecycle transitions and authorization-dependent operations |
| Parsing failure | Record failure; retain source artifact |
| Normalization failure | Do not create valid processed output |
| Chunking failure | Do not generate embeddings from incomplete chunks |
| Embedding failure | Keep version inactive; retry |
| Qdrant failure | Retry indexing; do not activate |
| OpenSearch failure | Retry indexing; do not activate |
| ACL metadata unavailable | Fail closed |
| Canonical chunk missing | Reject candidate and raise integrity event |
| LLM runtime failure | Return controlled error; preserve retrieval trace |
| Partial deletion | Keep tombstone; retry outstanding cleanup |

All retries must be bounded and observable.

## 12. Reconciliation

A reconciliation process must periodically detect inconsistencies between systems.

Checks should include:

- PostgreSQL references to missing MinIO objects
- Orphaned MinIO artifacts
- Missing chunks in Qdrant
- Missing chunks in OpenSearch
- Stale or incompatible index versions
- Pending Outbox events
- Failed or abandoned pipeline runs
- ACL synchronization inconsistencies
- Tombstones not fully propagated
- Active versions with incomplete artifacts or indexes

Reconciliation must generate actionable records and support safe retry.

## 13. Observability and Audit Events

Important events include:

- `document.received`
- `document.rejected`
- `document.processed`
- `document.indexing_started`
- `document.indexing_completed`
- `document.activation_completed`
- `document.activation_failed`
- `document.superseded`
- `document.deletion_requested`
- `document.deletion_completed`
- `retrieval.completed`
- `authorization.denied`
- `pipeline.retry`
- `reconciliation.detected`

Each event should carry suitable correlation identifiers, such as:

- `document_id`
- `document_version_id`
- `pipeline_run_id`
- `chunk_id`
- `index_version_id`
- `request_id`

Sensitive content must not be logged by default.

## 14. Consistency Guarantees

The platform uses the following consistency model:

- PostgreSQL: authoritative operational state and metadata
- MinIO: authoritative original and processed artifacts
- Qdrant: derived vector index
- OpenSearch: derived lexical index
- Outbox and reconciliation: coordination and recovery mechanisms

The architecture permits eventual consistency between operational metadata and derived indexes, but does not permit an incomplete document version to become active.

Authorization is not eventually consistent from the perspective of content exposure: authoritative permission checks must prevent access while derived ACL indexes are stale.

---

**Document Status:** Proposed for review.  
**Implementation and repository changes require separate explicit approval.**