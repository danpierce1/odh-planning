# Feature Specification: RAG Knowledge Sources

**Feature Branch**: `007-rag-knowledge-sources`
**Created**: 2026-01-27
**Status**: Draft
**Input**: User description: "Surface existing vector stores as first-class RAG knowledge sources in Gen AI Studio Playground via declarative ConfigMap configuration"

## Epics *(mandatory)*

### Epic 1: Vector Store ConfigMap Schema (Priority: P1, Owner: Platform/Backend)

Define and implement the ConfigMap schema for declaring vector store configurations, enabling platform engineers to expose existing vector stores in a GitOps-friendly, auditable manner.

**User Value**: Platform engineers can declaratively expose approved vector stores to their teams using familiar Kubernetes ConfigMap patterns, enabling governed RAG access without custom code.

**Technical Considerations**:
- Single ConfigMap per namespace containing multiple vector store entries
- Each entry references a Connection Type for credentials (consistent with External Models pattern)
- Schema must support PGVector, Qdrant, and Milvus connection parameters

**Outcomes by Persona**:

_Platform Engineer_:
- Define vector stores in a single ConfigMap per namespace
- Specify vector DB type, connection reference, collection/index name, and metadata
- Manage vector store exposure through GitOps workflows (declarative, auditable)

_ML Ops Engineer_:
- Review available vector stores defined in namespace ConfigMap
- Understand connection details and metadata for each vector store

**ConfigMap Schema (v1)**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: rag-knowledge-sources
  namespace: <user-namespace>
  labels:
    rhoai.io/component: rag-knowledge-sources
data:
  sources: |
    - name: product-docs
      type: pgvector  # pgvector | qdrant | milvus
      connectionRef: product-docs-connection  # Connection Type name
      collection: product_documentation
      description: "Product documentation knowledge base"
      owner: platform-team
    - name: internal-runbooks
      type: qdrant
      connectionRef: runbooks-connection
      collection: runbooks_v2
      description: "Internal operations runbooks"
      owner: sre-team
```

---

### Epic 2: Connection Types for Vector Stores (Priority: P1, Owner: Connections Team)

Define Connection Type templates for each supported vector database (PGVector, Qdrant, Milvus), enabling secure credential storage using the existing Connection Types pattern.

**User Value**: Platform engineers can store vector store credentials securely using familiar Connection Types, with consistent patterns across all RHOAI integrations.

**Technical Considerations**:
- Leverage existing Connection Types infrastructure (no new secret management)
- Each vector DB type needs a Connection Type template with appropriate fields
- Credentials stored as Kubernetes Secrets, referenced by Connection Type

**Outcomes by Persona**:

_Platform Engineer_:
- Create Connection Types for PGVector, Qdrant, or Milvus with appropriate credentials
- Reference Connection Types from vector store ConfigMap entries
- Manage credentials through existing Connection Types UI/patterns

_ML Ops Engineer_:
- Credentials are never exposed in ConfigMaps (only references)

**Connection Type Fields by Vector DB**:

| Vector DB | Required Fields |
|-----------|-----------------|
| PGVector | host, port, database, username, password, ssl_mode |
| Qdrant | url, api_key (optional), collection_name |
| Milvus | host, port, username, password, database |

> **Dependency**: Connections Team must provide Connection Type templates for vector databases.

---

### Epic 3: ConfigMap Discovery & Validation (Priority: P1, Owner: Platform/Backend)

Implement backend logic to discover vector store ConfigMaps in user namespaces, validate configurations, and expose available vector stores via API.

**User Value**: Users see only valid, properly configured vector stores in the Playground, with clear feedback when configurations are invalid.

**Technical Considerations**:
- Discovery scoped to user's namespace (per-namespace ConfigMap)
- Validate ConfigMap schema, Connection Type references, and vector DB type
- Surface validation errors clearly (which vector store, what's wrong)

**Outcomes by Persona**:

_AI Engineer_:
- See only valid, properly configured vector stores in Playground
- Invalid configurations don't appear (or appear with clear error indicator)

_Platform Engineer_:
- Receive clear feedback when ConfigMap has validation errors
- Understand which entries are valid vs. invalid and why

---

### Epic 4: Knowledge Sources UI in Playground (Priority: P1, Owner: Dashboard/gen-ai)

Add a "RAG / Knowledge Sources" section to the Playground UI, allowing users to browse available vector stores and enable them for chat sessions.

**User Value**: Users can discover and enable pre-approved vector stores directly in the Playground, enabling RAG experimentation without writing code.

**Technical Considerations**:
- UI pattern similar to MCP server selection in RHOAI 3.0
- Support multi-selection (enable multiple vector stores simultaneously)
- Display metadata (name, type, description) for each vector store

**Outcomes by Persona**:

_AI Engineer_:
- See "RAG / Knowledge Sources" section in Playground configuration
- Browse available vector stores with name, type, and description
- Enable one or more vector stores for the current chat session
- Disable vector stores without losing chat history

_Data Scientist_:
- Quickly identify available knowledge sources by description
- Compare model outputs with and without RAG enabled

---

### Epic 5: RAG Query Integration (Priority: P1, Owner: Dashboard/gen-ai)

Integrate Playground chat with llama-stack RAG primitives, routing queries through enabled vector stores and displaying retrieved sources alongside responses.

**User Value**: Users can chat against selected vector stores and see what sources informed the response, enabling transparent RAG experimentation.

**Technical Considerations**:
- Queries route through existing llama-stack RAG primitives (already deployed in 3.4)
- Multi-vector-store queries combine retrieval from all enabled stores
- Retrieved sources displayed alongside/below the response
- Query fails with clear error if any enabled vector store is unreachable

**Outcomes by Persona**:

_AI Engineer_:
- Chat messages are augmented with context from enabled vector stores
- See retrieved chunks/sources displayed with each response
- Receive clear error message if vector store is unreachable or misconfigured

_Data Scientist_:
- Understand which sources informed a given response
- Evaluate grounding quality by reviewing retrieved context

---

### Epic 6: Error Handling & Feedback (Priority: P1, Owner: Dashboard/gen-ai)

Implement clear error handling for vector store connectivity issues, misconfigurations, and retrieval failures.

**User Value**: Users understand exactly what went wrong when RAG queries fail, enabling quick troubleshooting or escalation to platform engineers.

**Technical Considerations**:
- Query fails entirely if any enabled vector store is unreachable (no graceful degradation in 3.4)
- Error messages should identify which vector store failed and why
- ConfigMap validation errors surfaced at discovery time, not query time

**Outcomes by Persona**:

_AI Engineer_:
- See clear error message identifying which vector store is unreachable
- Understand whether issue is connection, auth, or configuration
- Can disable problematic vector store and continue with others

_Platform Engineer_:
- Receive actionable error details for troubleshooting
- Understand connection vs. configuration issues

---

### Edge Cases

- What happens when a vector store becomes unreachable mid-session? The query fails with clear error; user can disable that vector store and continue.
- What happens if ConfigMap is malformed or missing required fields? Validation fails at discovery; invalid entries not shown in Playground (or shown with error indicator).
- What happens when Connection Type reference is invalid? Validation fails; entry not usable until Connection Type exists and is valid.
- What happens if no vector stores are configured in the namespace? "RAG / Knowledge Sources" section shows empty state with guidance for platform engineers.
- What happens if retrieval returns no results? Response generated without RAG context; consider indicating "no relevant sources found."
- What happens with very large retrieval results? Limit/truncate retrieved context per llama-stack defaults; not configurable in 3.4.
- What happens if multiple enabled vector stores have conflicting/duplicate content? Combined retrieval; deduplication handled by llama-stack (if at all).

## Performance & Scaling

| Concern | Impact | Consideration |
|---------|--------|---------------|
| **Multi-store retrieval** | Queries hit multiple vector stores simultaneously | Latency is max of individual stores; consider parallel retrieval |
| **Large collections** | Some vector stores may have millions of vectors | Retrieval performance depends on vector store configuration (out of scope for 3.4) |
| **Connection pooling** | Repeated queries to same vector stores | Consider connection reuse to reduce latency |
| **ConfigMap size** | Many vector stores in one ConfigMap | YAML parsing overhead; unlikely to be significant for expected scale |

## System Constraints

- Read-only usage (query + retrieval) in 3.4; no ingestion or lifecycle management
- Only PGVector, Qdrant, and Milvus supported in 3.4
- Single ConfigMap per namespace containing all vector store definitions
- Credentials managed via Connection Types (not inline in ConfigMap)
- Retrieval behavior is transparent but not configurable in 3.4
- llama-stack already deployed; this feature surfaces vector stores to it
- Query fails entirely if any enabled vector store is unreachable

## Key Entities

- **Vector Store**: A pre-existing vector database collection/index surfaced as a RAG knowledge source in Playground
- **Knowledge Sources ConfigMap**: A per-namespace ConfigMap containing vector store definitions
- **Vector Store Entry**: A single vector store definition within the ConfigMap, including type, connection reference, collection name, and metadata
- **Vector Store Connection**: A Connection Type instance storing credentials for accessing a vector store
- **Retrieved Source**: A chunk of text retrieved from a vector store during RAG query, displayed alongside the response

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Existing vector stores can be declared via ConfigMap and discovered in Playground
- **SC-002**: Only llama-stack-compatible vector DBs (PGVector, Qdrant, Milvus) are selectable
- **SC-003**: Users can chat against selected vector stores without writing code
- **SC-004**: Users can enable multiple vector stores simultaneously for combined retrieval
- **SC-005**: Retrieved sources are displayed alongside chat responses
- **SC-006**: Configuration is portable, GitOps-friendly, and auditable (standard ConfigMap)
- **SC-007**: Clear error messages surface when vector store is unreachable or misconfigured
- **SC-008**: Feature is validated by successfully querying at least one external vector store per supported backend (PGVector, Qdrant, Milvus)

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **llama-stack Team** | RAG query integration | Integration | Queries route through llama-stack primitives; llama-stack already deployed |
| **Connections Team** | Connection Type templates for vector DBs | Integration | PGVector, Qdrant, Milvus connection templates needed; blocking dependency |
| **Platform/Backend** | ConfigMap discovery and validation API | Integration | Backend endpoints for vector store discovery |
| **UX Team** | Knowledge Sources UI design | Design | Selection UI, source display, error states |
| **QE Team** | E2E test coverage | Testing | Tests against real vector store backends (PGVector, Qdrant, Milvus) |
| **Playground Team (gen-ai)** | UI integration and RAG query routing | Integration | Main implementation team |

### Blocking Dependencies

- **Connections Team**: Connection Type templates for PGVector, Qdrant, Milvus must exist before full flow works
- **llama-stack Team**: Confirm llama-stack RAG primitives support the three vector DB types
- **UX Team**: Designs for Knowledge Sources section and source display needed before UI implementation

### Informational (Capacity/Awareness)

- **Docs Team**: New ConfigMap schema needs documentation for platform engineers
- **llama-stack Team**: FYI on expected query patterns and multi-store retrieval

## Clarifications

### Session 2026-01-27

- Q: How are vector stores configured? → A: Single ConfigMap per namespace with multiple entries
- Q: What namespace scope? → A: Per-namespace; users see only their namespace's vector stores
- Q: How are credentials handled? → A: Connection Types (consistent with External Models pattern)
- Q: Is llama-stack already deployed? → A: Yes, this feature surfaces vector stores to existing llama-stack
- Q: Can users select multiple vector stores? → A: Yes, multi-selection for combined retrieval
- Q: How are errors handled? → A: Query fails entirely with clear error message
- Q: Are retrieved sources shown? → A: Yes, displayed alongside/below the response
- Q: Which vector DBs supported? → A: Exactly PGVector, Qdrant, Milvus in 3.4

## Assumptions

- llama-stack is deployed and operational in RHOAI 3.4 with RAG primitive support
- llama-stack supports PGVector, Qdrant, and Milvus as retrieval backends
- The existing Connection Types infrastructure can be extended for vector store credentials
- Platform engineers have sufficient Kubernetes access to create ConfigMaps in user namespaces
- Vector stores referenced in ConfigMaps are already populated (ingestion is out of scope)
- Users understand RAG concepts and the value of grounding responses in external knowledge
- Retrieved source display follows existing patterns for metadata/context in Playground responses

## Future Considerations (Post-3.4)

- UI-driven ingestion pipelines
- Vector store lifecycle management (create, delete, update)
- Metadata editing and tagging
- Per-project or per-user access controls
- Observability and retrieval diagnostics
- Unified "Knowledge Registry" abstraction
- Configurable retrieval parameters (top-k, similarity threshold)
- Graceful degradation (skip failing stores, continue with others)
