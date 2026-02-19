# Feature Specification: Surface Existing Vector Stores

**Feature Branch**: `RHAISTRAT-545-surface-existing-vector-stores`
**Created**: 2026-02-19
**Status**: Draft
**Input**: User description: "Provide a lightweight, governed RAG-as-a-Service experience in Gen AI Studio Playground by surfacing existing vector stores through declarative configuration."

## Overview

This feature enables platform engineers to register pre-existing vector databases (PGVector, Qdrant, Milvus) via ConfigMaps, making them discoverable and usable by AI engineers in the Gen AI Studio Playground for RAG experimentation.

## Epics

### Epic 1: External Vector Store configuration/loading (Priority: P1, Owner: Dashboard/gen-ai)

Enable platform engineers to register existing vector stores via ConfigMaps, making them discoverable in the Playground.

**User Value**: Platform teams control which vector stores are available to AI engineers, enabling self-service RAG experimentation with enterprise-approved knowledge sources.

**Technical Considerations**:
- Vector store definitions stored in Kubernetes ConfigMaps (namespace-scoped visibility)
- Single vector store per chat session in 3.4
- OpenAI-compatible vector database APIs through llama-stack interface
- ConfigMap changes require llamastack distribution restart to take effect
- ConfigMap schema must be GitOps friendly for declarative management
- Optional Metadata fields: description, owner, domain, name
- mocked bff backend?
- feature flag?

**Needs clarification**:
- Use yaml fields or JSON blob for the ConfigMap vector store configuration
- All external vector stores defined under single ConfigMap?
- ConfigMap is scoped to a single namespace
- All users in namespace can select and use any vector store.
- Vector store credentials storage: How are credentials managed?
  - platform engineer creates Secret, and references in ConfigMap (namespace-scoped)?
- When should the vector store connections be validated? (e.g. on load of playground, on attempt to enable vector store, ...)
- What providers to test/support for MVP? (Milvus, stretch for PGVector, Qdrant)

**Outcomes by Persona**:

_Platform Engineer_:
- Register an existing PGVector database via ConfigMap
- Provide metadata (description, owner, domain) to help users understand data provenance
- Manage vector store access at namespace level
- Understand that ConfigMap updates require llamastack distribution restart

_AI Engineer_:
- View all available vector stores in namespace from "Knowledge Sources" section
- See vector store metadata (name, type, description) without accessing external docs
- Use vector stores without needing database credentials or connection details
- Discover approved knowledge sources for experiments

---

### Epic 2: External vector stores usable in playground (Priority: P1, Owner: Dashboard/gen-ai)

Enable AI engineers in the GenAI playground to select an external vector store and chat against it.

**User Value**: Engineers can rapidly prototype and evaluate RAG applications using enterprise-approved knowledge sources without writing code, managing connections, or understanding retrieval implementation details.

**Technical Considerations**:
- Retrieval implementation details (chunk size, embedding model, similarity thresholds) hidden from users
- Read-only operations (query and retrieval only, no write access to vector stores)
- Chat session state includes enabled vector store reference

**Needs Clarification**:
- Playground currently supports one inline vector store across all panes. For external vector stores MVP, will we support one or multiple vector stores across panes?
- Whats the behaviour when opening multiple chat comparisons?
  - If a user selects an external vector store, and opens another chat comparison pane, should the new pane point to the same knowledge source(s), have no vector store selected, or something else.

**Outcomes by Persona**:

_AI Engineer_:
- Enable a vector store for chat session with single click/selection
- Compare model responses with and without vector stores to evaluate RAG quality
- Disable vector store mid-session to test non-RAG behavior
- Chat against pre-ingested product documentation to test customer support scenarios

---

### Epic 3: Error Handling and Observability (Priority: P1, Owner: Dashboard/gen-ai)

Provide clear, actionable error messages when vector stores are unreachable or misconfigured, guiding users toward resolution without requiring deep technical knowledge.

**User Value**: Engineers can quickly identify and escalate configuration issues instead of debugging connection problems, while platform engineers receive clear signals about infrastructure health before users are impacted.

**Technical Considerations**:
- Distinguish between connection errors, misconfiguration errors, and empty result sets
- Connection validation timing determines UX (eager vs. lazy validation)
- Error messages must guide users to appropriate next steps (retry, contact platform team, choose different store)
- Health status indicators for vector stores (available, unreachable, misconfigured)

**Outcomes by Persona**:

_AI Engineer_:
- See clear error message when vector store is unreachable
- See which vector stores are currently available vs. unavailable
- Receive user-friendly error message (not stack trace) when queries fail
- Distinguish infrastructure issues from query problems

_Platform Engineer_:
- Be notified when vector store ConfigMap has invalid connection details
- Fix configuration issues before users encounter errors
- Understand validation failures during ConfigMap registration

**Needs clarification**
- Input on error messages/scenarios to handle

---

## Key Entities

- **Vector Store Configuration**: A ConfigMap defining a registered vector database, including:
  - Type (PGVector, Qdrant, or Milvus)
  - Connection reference (endpoint URL or Kubernetes service name)
  - Collection or index name
  - Secret reference for authentication credentials
  - Optional metadata (description, owner, domain tags)
  - Validation status (valid, unreachable, misconfigured)

- **Chat Session**: An active Playground conversation, including:
  - Selected model
  - Enabled vector store (zero or one)
  - Chat history
  - Session configuration state

- **RAG Query**: A user message that triggers retrieval, including:
  - User input text
  - Retrieved context chunks (hidden from user in 3.4)
  - Model response with grounding
  - Retrieval metadata (hidden from user in 3.4)

## Success Criteria

### Measurable Outcomes

- **SC-001**: Platform engineers can register a vector store via ConfigMap and see it appear in Playground after llamastack restart
- **SC-002**: AI engineers can select a vector store and receive grounded responses in their first chat query without configuration steps
- **SC-003**: System successfully queries at least one external vector store per supported backend (PGVector, Qdrant, Milvus) in validation testing
- **SC-004**: Users receive clear, actionable error messages when vector stores are unreachable, distinguishing infrastructure issues from query problems
- **SC-005**: ConfigMaps are portable and auditable, allowing platform teams to manage vector store access through GitOps workflows
- **SC-006**: Engineers can compare RAG-enabled vs. non-RAG responses by toggling vector store enablement within the same chat session
- **SC-007**: Total response time for RAG queries is under 3 seconds for typical queries
- **SC-008**: Vector store selection behaves similarly to MCP tool selection (RHOAI 3.0), maintaining UI consistency

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **Llama-stack Team** | RAG primitives for PGVector, Qdrant, Milvus | Integration | Blocking dependency for multi-backend support |
| **Platform/Backend** | ConfigMap and llamastack distribution lifecycle | Integration | Restart mechanism for ConfigMap updates |
| **UX Team** | RAG knowledge sources UI patterns | Design | Visual design for vector store list, enablement, error states |
| **QE Team** | E2E test coverage | Testing | ConfigMap registration, RAG queries, error handling, multi-backend validation |

### Blocking Dependencies

- **Llama-stack Team**: RAG primitives must support all three vector database types (PGVector, Qdrant, Milvus)
- **Platform/Backend**: ConfigMap and llamastack distribution restart mechanism must be available

### Informational (Capacity/Awareness)

- **Security Team**: FYI on Secret reference pattern for vector database credentials (namespace-scoped)

## Assumptions

- Organizations already have existing vector databases populated through external pipelines, CLIs, or CI/CD systems
- Platform engineers have Kubernetes permissions to create and manage ConfigMaps and Secrets in relevant namespaces
- Vector databases are network-reachable from the Playground backend services with reasonable latency (<500ms)
- llama-stack RAG primitives provide a consistent interface across PGVector, Qdrant, and Milvus backends
- Vector stores contain pre-computed embeddings and do not require re-indexing or chunking during query time
- Single vector store per chat session is sufficient for initial release (multi-store retrieval deferred to post-3.4)
- Authentication to vector databases (if required) is handled via Kubernetes Secrets referenced in ConfigMaps, not user-provided credentials
- Platform engineers can restart llamastack distribution when adding or updating vector store ConfigMaps
- Retrieval latency from vector databases is reasonable (contributes <1 second to total query time)

## Out of Scope

The following capabilities are explicitly excluded from RHOAI 3.4 and documented for future consideration:

- **Document upload or ingestion workflows**: Users cannot upload files to create new vector stores
- **Chunking, embedding, or indexing configuration**: All data transformation happens outside the Playground
- **Data transformation pipelines**: No ETL or preprocessing capabilities
- **Vector store creation or lifecycle management**: Platform engineers manage vector databases through external tools
- **UI-driven ingestion workflows**: All ingestion remains in external systems
- **Retrieval parameter tuning**: Users cannot adjust chunk size, similarity thresholds, or re-ranking in 3.4
- **Per-project or per-user access controls**: Vector store visibility is namespace-scoped
- **Observability and retrieval diagnostics**: No detailed metrics or tracing for retrieval operations
- **Automatic ConfigMap reload**: ConfigMap changes require manual llamastack distribution restart
- **Hot-reload of vector store configurations**: No dynamic configuration updates without restart
