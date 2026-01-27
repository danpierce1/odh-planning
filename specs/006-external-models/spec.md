# Feature Specification: External Models

**Feature Branch**: `006-external-models`
**Created**: 2026-01-27
**Status**: Draft
**Input**: User description: "Enable users to register and utilize external, third-party models (outside the namespace) directly within the AI Available Assets page"

## Epics *(mandatory)*

### Epic 1: External Model Registration (Priority: P1, Owner: Dashboard/gen-ai)

Enable users to register external model endpoints via a wizard in the AI Available Assets page, capturing model details, endpoint URL, and authentication credentials using the existing Connection Types pattern.

**User Value**: Users can bring their own external models (OpenAI, Anthropic, Gemini, etc.) into the platform without manual Kubernetes configuration, centralizing all AI assets in one place.

**Technical Considerations**:
- Registration must leverage existing Connection Types pattern for credential storage
- Validation must test API connectivity before allowing registration to complete
- Must support OpenAI-compatible APIs as baseline, plus built-in adapters for Anthropic and Gemini

**Outcomes by Persona**:

_AI Engineer_:
- Register an external model by providing model name, endpoint URL, and API key
- Select from supported provider types (OpenAI-compatible, Anthropic, Google Gemini)
- Receive immediate feedback if connection validation fails

_ML Ops Engineer_:
- API keys stored securely as Kubernetes Secrets via Connection Types
- Manage external model credentials at namespace level (shared by project members)

_Platform Engineer_:
- External model registration follows existing platform security patterns
- No new secret management infrastructure required

---

### Epic 2: Model Registry Integration (Priority: P1, Owner: Model Registry Team)

Integrate external models with the Model Registry so they appear alongside internally-hosted models, enabling unified model discovery and management.

**User Value**: Users have a single, consistent view of all available models—internal and external—without needing to track external models separately.

**Technical Considerations**:
- Model Registry must support a new model type/source indicator for external models
- External model metadata (provider, endpoint) must be stored without exposing secrets
- Registry queries must include/filter external models appropriately

**Outcomes by Persona**:

_AI Engineer_:
- See external models listed in Model Registry alongside internal models
- Filter/search models by source (internal vs. external)
- View external model metadata (provider type, endpoint, registration date)

_Data Scientist_:
- Discover available external models through familiar Model Registry interface

> **Dependency**: Model Registry team must support external model type before full integration.

---

### Epic 3: Provider Adapters (Priority: P1, Owner: Dashboard/gen-ai)

Implement built-in adapters that translate between internal API format and provider-specific formats for Anthropic and Google Gemini, enabling seamless use of non-OpenAI providers.

**User Value**: Users can leverage Anthropic Claude and Google Gemini models without worrying about API format differences—the platform handles translation automatically.

**Technical Considerations**:
- OpenAI-compatible format is the internal standard; adapters translate to/from this
- Adapters must handle streaming responses where provider supports it
- Adapters must pass through provider errors directly to user

**Outcomes by Persona**:

_AI Engineer_:
- Use Anthropic Claude models with the same interface as OpenAI models
- Use Google Gemini models with the same interface as OpenAI models
- Streaming works automatically if the provider supports it

_Platform Engineer_:
- Adapter architecture is extensible for future providers

**Supported Providers (v1)**:
- OpenAI-compatible APIs (baseline, no adapter needed)
- Anthropic Claude API (built-in adapter)
- Google Gemini API (built-in adapter)

---

### Epic 4: External Model Proxy (Priority: P1, Owner: Platform/Backend)

Implement backend proxy that forwards inference requests to external model endpoints, injecting authentication credentials securely without involving KServe/ModelMesh runtimes.

**User Value**: Users can invoke external models through a consistent platform API, with credentials injected securely server-side rather than exposed to client applications.

**Technical Considerations**:
- Direct proxy architecture (no KServe/ModelMesh involvement)
- Credentials retrieved from Connection Types at request time
- Must support streaming responses for providers that offer it
- Pass through provider errors directly (no normalization)

**Outcomes by Persona**:

_AI Engineer_:
- Call external models through platform API endpoint
- Receive responses (including streaming) from external providers
- See provider error messages directly when issues occur

_ML Ops Engineer_:
- API keys never exposed to client-side code
- Request routing handled transparently by platform

---

### Epic 5: AI Available Assets UI (Priority: P1, Owner: Dashboard/gen-ai)

Update the AI Available Assets page to support external model registration and display, with clear visual distinction between internal and external models.

**User Value**: Users have a unified view of all models with clear indication of which are internal vs. external, and can register new external models alongside deploying internal ones.

**Technical Considerations**:
- "Register External Model" action at same level as "Deploy Model"
- Visual indicators (icon, badge, or label) distinguishing external models
- Registration wizard must guide user through provider selection and credential input

**Outcomes by Persona**:

_AI Engineer_:
- Click "Register External Model" from AI Available Assets page
- Complete wizard with provider type, endpoint URL, model name, and API key
- See registered external model appear in assets list with "External" indicator

_Platform User_:
- Clearly distinguish internal models from external models in the list
- Understand which models use namespace credentials vs. platform-hosted

---

### Epic 6: Playground Integration (Priority: P1, Owner: Dashboard/gen-ai)

Enable external models to be selected and used within the AI Playground, with feature availability dependent on provider capabilities.

**User Value**: Users can experiment with external models in the familiar Playground interface, with the same chat experience they use for internal models.

**Technical Considerations**:
- External models appear in Playground model selector
- Feature availability (streaming, etc.) depends on provider capabilities
- Chat history and conversation work identically to internal models

**Outcomes by Persona**:

_AI Engineer_:
- Select an external model in AI Playground model dropdown
- Chat with external model using standard Playground interface
- Streaming responses work if provider supports streaming

_Data Scientist_:
- Compare external model responses against internal models in Playground
- Test prompts against external models before production use

---

### Edge Cases

- What happens when external API validation fails during registration? Show clear error message with provider response; do not save registration until validation passes.
- What happens when a registered external model's API key expires or is revoked? Inference requests fail with provider's auth error passed through; user must update credentials.
- What happens if an external provider has an outage? Provider error message passed through to user; platform does not retry.
- What happens when provider rate limits the API key? Rate limit error from provider shown to user directly.
- What happens if user registers same external model twice with different names? Allow it—user may want different configurations or separate tracking.
- What happens when deleting an external model that's referenced in saved prompts/projects? Warn user of references; proceed with deletion if confirmed.
- What happens if provider changes their API format? Built-in adapters may need updates; platform team maintains adapters.

## Performance & Scaling

| Concern | Impact | Consideration |
|---------|--------|---------------|
| **Proxy Latency** | Additional hop through platform backend | Latency added should be minimal; external provider latency dominates |
| **Concurrent Requests** | Multiple users hitting same external API key | Namespace-level keys shared by project members; rate limits are user's responsibility |
| **Streaming** | Backend must support streaming passthrough | Proxy must not buffer entire response; stream chunks as received |
| **Connection Pooling** | Repeated requests to same external endpoint | Consider connection reuse for performance |

## System Constraints

- API keys managed at namespace level (not per-user)
- OpenAI-compatible API format is the internal standard
- Built-in adapters for Anthropic and Gemini only; other providers must be OpenAI-compatible
- No KServe/ModelMesh involvement (direct proxy architecture)
- Provider errors passed through directly (no retry logic, no normalization)
- Connection validation required before registration completes

## Key Entities

- **External Model**: A model endpoint registered in the platform that points to a third-party API (Anthropic, OpenAI, Gemini, etc.)
- **Provider Type**: The API format/provider for an external model (OpenAI-compatible, Anthropic, Gemini)
- **Provider Adapter**: A built-in translator that converts between internal format and provider-specific API format
- **External Model Connection**: A Connection Type instance storing the API key and endpoint URL for an external model
- **Model Source**: An attribute distinguishing internal (platform-hosted) models from external (third-party) models

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can register an external model endpoint via the AI Available Assets UI
- **SC-002**: External models appear in AI Available Assets list with clear visual distinction from internal models
- **SC-003**: External models appear in Model Registry alongside internal models
- **SC-004**: Users can select and use external models in AI Playground
- **SC-005**: Inference requests to external models are successfully proxied and responses returned
- **SC-006**: API keys are stored securely using Connection Types (Kubernetes Secrets)
- **SC-007**: Connection validation occurs during registration; invalid credentials prevent save
- **SC-008**: Streaming responses work for providers that support streaming
- **SC-009**: Provider errors are displayed directly to users without modification

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **Model Registry** | Support external model type in registry | Integration | New model source attribute; blocking dependency |
| **Connections Team** | Connection Type for external model credentials | Integration | Leverage existing pattern; may need new connection type definition |
| **Platform/Backend** | Proxy API for external model inference | Integration | New endpoints for registration CRUD + inference proxy; blocking dependency |
| **UX Team** | Registration wizard and visual distinction | Design | Provider selection, validation feedback, external model indicators |
| **QE Team** | E2E test coverage | Testing | Registration flow, inference, error handling, Playground integration |
| **Playground Team (gen-ai)** | External models in Playground selector | Integration | Model list must include external models |

### Blocking Dependencies

- **Model Registry Team**: Must support external model type before registry integration
- **Platform/Backend**: Proxy API must be available for inference to work
- **Connections Team**: Connection Type definition needed for credential storage
- **UX Team**: Registration wizard designs needed before UI implementation

### Informational (Capacity/Awareness)

- **Security Team**: FYI on external API credential storage approach (uses existing Connection Types pattern)

## Clarifications

### Session 2026-01-27

- Q: Which external providers are supported? → A: OpenAI-compatible APIs (baseline) + Anthropic + Google Gemini (built-in adapters)
- Q: Who manages API keys? → A: Namespace-level keys, shared by project members
- Q: How are credentials stored? → A: Via existing Connection Types pattern (Kubernetes Secrets)
- Q: Is a serving runtime needed? → A: No, direct proxy architecture (no KServe/ModelMesh)
- Q: Where does registration UI appear? → A: Alongside "Deploy model" on AI Available Assets page
- Q: Do external models appear in Model Registry? → A: Yes, full integration
- Q: How does Playground integration work? → A: Feature availability depends on provider capabilities
- Q: How are non-OpenAI formats handled? → A: Built-in adapters translate Anthropic/Gemini to internal format
- Q: How are provider errors handled? → A: Pass through directly to user
- Q: Is connection validated during registration? → A: Yes, validation required before save

## Assumptions

- The existing Connection Types infrastructure can be extended to support external model credentials
- Model Registry schema can be extended to include external model metadata and source indicator
- Backend can implement streaming passthrough for external provider responses
- Anthropic and Google Gemini APIs are stable enough for built-in adapter implementation
- Users understand they are responsible for their own API key costs and rate limits
- External providers' API documentation is sufficient for adapter implementation
- Platform has outbound network access to reach external provider endpoints
