# Feature Specification: Surface Existing Vector Stores

**Feature Branch**: `RHAISTRAT-545-surface-existing-vector-stores`
**Created**: 2026-02-19
**Status**: Draft
**Input**: User description: "Provide a lightweight, governed RAG-as-a-Service experience in Gen AI Studio Playground by surfacing existing vector stores through declarative configuration."

## Overview

This feature enables platform engineers to register pre-existing vector databases (PGVector, Qdrant, Milvus) via ConfigMaps, making them discoverable and usable by AI engineers in the Gen AI Studio Playground for RAG experimentation.

## Epics

### Epic 1: Feature Flag for External Vector Stores (Priority: P1, Owner: Dashboard/gen-ai)

Control availability of the external vector store feature through a feature flag in the platform configuration, enabling safe progressive rollout and administrative control over vector store access.

**User Value**: Platform administrators can progressively enable external vector store capabilities, ensuring the feature is only exposed to users when infrastructure is ready and approved, reducing rollout risk.

**Technical Considerations**:
- The feature flag can be added to the OdhDashboardConfig CR. It will control whether the external vector stores are visible in the UI. If flag is set to true, show the feature in UI, otherwise (flag set to false or not present) it should be hidden.
- Flag state must be evaluated before rendering any external vector store UI components
- Documentation must explain the purpose of the feature flag and enablement steps.

**Outcomes by Persona**:

_Platform Engineer_:
- Can enable the feature flag so the external vector stores appear in the UI. The feature is hidden if flag is missing or false.
- Can find documentation that explains the flag and how to enable/disable it

_AI Engineer_:
- Sees the external vector stores in UI only when the platform has enabled the flag

---

### Epic 2: External Vector Stores registration during BFF LSD install (Priority: P1, Owner: Dashboard/gen-ai)

Platform engineers will be responsible for creating the "gen-ai-aa-vector-stores" ConfigMap, and this epic will update the Install phase of LSD in our gen-ai BFF in order to check for the presence of the "gen-ai-aa-vector-stores" ConfigMap - if present, we should configure any valid external vector stores to be included in the LSD that gets created (see InstallLlamaStackDistribution and generateLlamaStackConfig functions).

**User Value**: Enabling the loading of external vector stores from a platform engineer created ConfigMap is the required step for surfacing the external vector stores for selection/enablement in the gen ai Playground.

**Technical Considerations**:
- **Include validation of supplied external vector stores**: 
- For the install logic we must include some validation steps to check that valid external vector stores have been supplied.
  Spike what errors llamastack currently responds with for various invalid configurations.
  Some validations the code should carry out:
  - Ensure each vector db can be connected to (TBD if llamastack does this when starting up, and what error if so). Some may be supplied with credentials. Return a failure error to UI if invalid configuration.
  - Ensure the embedding model that each supplied vector store uses will be available (the models are supplied to the Install LSD/Playground step, so cross check what embedding model has been supplied). TBD if llamastack does this when starting up, and what error if so. If the embedding model isn't available, we can skip registering that vector store in LSD install, and allow the install of LSD to complete, then just display the external vector store as greyed out in the UI (with a message to indicate why it can't yet be enabled).

- Vector store definitions stored in namespace-scoped Kubernetes ConfigMaps
- Single vector store per chat session in 3.4
- OpenAI-compatible vector database APIs through llama-stack interface
- ConfigMap changes require llamastack distribution restart to take effect
- ConfigMap schema must be GitOps friendly for declarative management
- Optional Metadata fields: description, owner, domain, name

**Needs clarification**:
- Q: Use yaml fields or JSON blob for the ConfigMap vector store configuration?
  A: The configmap created by platform engineer will use a yaml configuration, listing the vector stores under a "stores.yaml", see https://gist.github.com/ederign/edf4edf4f3aff8b0092c0799eac72bb8
- Q: All external vector stores defined under single ConfigMap?
  A: Yes
- Q: ConfigMap is scoped to a single namespace?
  A: Yes
- Q: All users in namespace can select and use any vector store.
- Q: Vector store credentials storage: How are credentials managed?
  A: platform engineer creates Secret, and references in ConfigMap (namespace-scoped)?
- Q: When should the vector store connections be validated? On load of playground for this strat. (in future likely also when user adds from the AAE page)
- Q: What providers to test/support for MVP?
  A: Aim for Milvus, PGVector, Qdrant

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

### Epic 3: An external vector store can be enabled for chat session in UI (Priority: P1, Owner: Dashboard/gen-ai)

Enable AI engineers in the GenAI playground to enable an external vector store for a chat session from the Knowledge tab.

**User Value**: Engineers can rapidly prototype and evaluate RAG applications using enterprise-approved knowledge sources without writing code, managing connections, or understanding retrieval implementation details.

**Technical Considerations**:
- User can enable only one vector store at a time per chat pane (whether inline/external)
- User can see, but cannot enable, a vector store which does not have the associated embedding model available
- If a user opens multiple chat comparison panes, replicate the vector store settings across the panes (vector store id)
- Show error if external vector stores related misconfiguration issue arises on creation of playground (Install of LSD)
- See clear error message if vector store is unreachable
- User can only see and enable external vector stores if feature flag enabled
- Retrieval implementation details (chunk size, embedding model, similarity thresholds) hidden from users
- Read-only operations (query and retrieval only, no write access to vector stores)
- Chat session state includes enabled vector store reference

**Outcomes by Persona**:

_AI Engineer_:
- Enable a vector store for chat session with single click/selection
- Compare model responses with and without vector stores to evaluate RAG quality
- Disable vector store mid-session to test non-RAG behavior
- Chat against pre-ingested product documentation to test customer support scenarios

---

### Epic 4: Add external vector stores flow to AI Asset Endpoints (AAE) page in UI (Priority: P1, Owner: Dashboard/gen-ai)

Similar to the existing Add a model flow in AAE, we want to add support for viewing external vector stores in AAE and being able to select one or more external vector stores to be included when creating a new playground via an Add to Playground link (again similar to Add a model).

**User Value**: Engineers can view external vector stores in more detail, and can select vector stores to be included when creating a new playground.

**Technical Considerations**:
- Feature should only be visible if feature flag enabled
- External vector stores should be visible under a Vector Stores tab in AAE page. For each vector store, we can display columns: Vector store, Provider, Embedding model, Domain, Status (referencing figma at https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=1419-6335&t=u9oLDgOyt8i0NEwO-0)
- If user clicks Add to Playground, the existing Configure Playground modal will open, this modal should be updated to include a Knowledge sources section with a list of selectable vector stores for inclusion in the new playground. User can select from both the Available models and the vector stores lists in this modal, for inclusion in the new Playground to be created.
- Clicking Add to Playground will pass the selection of vector stores to our create playground backend, Epic 2 "External Vector Stores registration during BFF LSD install" should add changes to the install phase of that backend such that the passed vector stores get registered in the new playground.
- Clicking Add to Playground will not register any new external vector stores in the "gen-ai-aa-vector-stores" ConfigMap. Instead that ConfigMap is used as a reference for what external vector stores can be viewed/selected, but we don't change it, only the Platform Engineer is responsible for adding/modifying that ConfigMap manually.

**Dependency on PM/UX for approval**: this Epic is a "late add" to this strat, and the details are being fleshed out after the strat refinement session (due to it not having been included in the initial strat outline/description). It has not yet been reviewed/approved by PM/Design, so details may yet change.

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
- **SC-007**: Vector store selection behaves similarly to MCP tool selection (RHOAI 3.0), maintaining UI consistency

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