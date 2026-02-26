# Feature Specification: External Model Endpoints Support

**Feature Branch**: `005-name-external-model-endpoints`
**Created**: 2026-02-17
**Status**: Draft
**Strategic Input**: JIRA RHAISTRAT-172 - AI Asset Endpoints - External Model Endpoints Support

## Overview

Enable users to register and utilize external model endpoints directly within the AI Available Assets page in Gen AI Studio. Two connection options are supported: (1) **Models in another namespace**—users connect to a model deployed elsewhere in the cluster using the internal API (e.g. URL pattern `svc.cluster.local`) and an authorized token, keeping traffic in-cluster; (2) **External third-party providers** (e.g., Anthropic, OpenAI, AWS, Gemini)—users provide the provider’s URL and API key for authentication. This extends the current capability beyond namespace-hosted LLS servers to include both in-cluster cross-namespace models and, when allowed by the platform, arbitrary external endpoints hosting OpenAI-compliant APIs.

**Scope**: External endpoints will be visible in the AI Available Assets Models page and selectable in the AI Playground for inference.

**Out of Scope**:
- **API key or token rotation / lifecycle management**: Users must manually remove and re-add credentials (e.g. update the stored secret) when keys rotate; there is no in-product rotation or automatic refresh.
- **Use outside AI Assets and Playground**: External model endpoints are only registerable and usable from the AI Available Assets page and the AI Playground. Integration with MaaS, AI Hub, pipelines, model serving deployment flows, or other Gen AI Studio surfaces is not in scope for this feature.

---

## Epics & User Stories

### Epic 1: Feature Flags for External Model Visibility and Provider Access (Priority: P1, Owner: gen-ai)

Provide two feature flags in the DataScienceCluster (DSC) custom resource so platform engineers can control whether the external model feature is visible and whether connecting to external model providers is allowed. Feature flag state is loaded from the BFF to drive UI and registration behavior.

**User Value**: Platform Engineers can safely roll out external model support (starting with in-cluster, cross-namespace only) and optionally allow external provider connections only when they accept the security implications. AI Engineers see the external model feature only when the platform has enabled it.

**Connection options (when feature is visible)**:
- **Option A — External model provider**: Connect using an API key from that provider. Data sent in the responses API (e.g., RAG context, MCP server results, user questions) is sent outside the cluster to the provider.
- **Option B — Model in another namespace**: Connect to a model deployed in another namespace using the internal API (URL pattern such as `svc.cluster.local`) and an authorized token. Traffic stays in-cluster.

When only the visibility flag is enabled, **only Option B** (model in another namespace) is supported. Option A is available only when the second flag (allow external model provider) is enabled.

**Technical Considerations**:
- **Visibility flag (Flag 1)**: Added to the DataScienceCluster CR. Controls whether the external model feature is shown in the UI. Default: flag is **missing** → feature is **hidden**.
- **External provider flag (Flag 2)**: Added to the DataScienceCluster CR. Controls whether users can add an external model provider (Option A). When disabled, only Option B (in-cluster, cross-namespace) is allowed. Rationale: enabling external providers means data from the responses API is sent outside the cluster.
- Feature flag values are consumed by the BFF and drive UI visibility and registration options.

**Documentation requirement**: Documentation MUST clearly describe (1) what each feature flag does, (2) how platform engineers enable them, and (3) the security risks of enabling the external model provider flag (data leaving the cluster).

**Outcomes by Persona**:

_Platform Engineer_:
- Can enable the visibility flag so the external model feature appears in the UI; when disabled or missing, the feature is hidden
- Can enable the external provider flag to allow API-key-based connections to external providers; when disabled, only in-cluster cross-namespace (Option B) is available
- Can find documentation that explains both flags, how to enable them, and the security risks of enabling external model providers

_AI Engineer_:
- Sees the external model registration flow only when the platform has enabled the visibility flag
- Can register only in-cluster cross-namespace models when the external provider flag is off; can register external providers when it is on

**Success Criteria**:
- **SC-001**: When the visibility flag is missing or disabled, the external model feature is hidden in the UI; when enabled, only in-cluster cross-namespace (Option B) registration is available unless the external provider flag is enabled
- **SC-002**: Platform engineers can enable or disable each feature flag via the DataScienceCluster CR and have documentation that explains what each flag does, how to enable it, and (for the external provider flag) the security risks of enabling it

---

### Epic 2: External Endpoint Registration - MVP (Priority: P1, Owner: gen-ai)

Provide a minimal user interface for registering external model endpoints with secure credential storage, backed by BFF endpoints that create and verify the configuration. When the feature is visible (Epic 1 visibility flag enabled), only "connect to model in another namespace" (internal API, e.g. `svc.cluster.local`, plus authorized token) is supported until the external provider flag is enabled.

**Scope**: This epic covers (1) the **UI wizard** to add an external model (form/wizard from the AI Available Assets page) and (2) the **BFF endpoints** that support it: an endpoint to **create** the external model and an endpoint to **verify** the model connection before saving.

**User Value**: AI Engineers and Platform Engineers can integrate in-cluster cross-namespace models and, when allowed, external AI services (Anthropic, OpenAI, etc.) into their Gen AI Studio workflow.

**Technical Considerations**:
- **Create endpoint**: Creating an external model is persisted by adding an entry to a **single ConfigMap per namespace** that stores entries for every created external model in that namespace. The ConfigMap follows the same schema as an inference model entry and a provider in a LlamaStack config. User-provided API keys (external provider) or authorized tokens (cross-namespace) must be stored as Kubernetes Secrets, not plain text.
- **Verify endpoint**: Model verification sends a short sample request to the endpoint’s chat completions API, including `max_output_tokens` to reduce token consumption. The response is validated to be OpenAI compliant. Verification must return distinct errors so the UI can show whether the **connection failed** (e.g. unreachable, auth failure) or the **API is not OpenAI compliant** (e.g. invalid response shape).
- Endpoint URL is a passthrough to a service hosting OpenAI-compliant APIs (internal URL pattern e.g. `svc.cluster.local` for Option B; external URL for Option A when allowed).
- Model ID is a **text input** for MVP; Model Alias is an **optional text input** for user-friendly naming.
- External endpoints are namespace-scoped.
- Secrets are namespace-scoped.
- **UX requirement**: The UI MUST show a warning when the user is connecting to an **external model provider** (Option A), alerting them that data from the responses API will be sent outside the cluster. This applies to the registration flow and can be noted in design for the minimal registration wizard.
- There is a 1-1 mapping between provider and model.


**Outcomes by Persona**:

_AI Engineer_:
- Can access a registration form/wizard from the AI Available Assets page (when visibility flag is enabled)
- Can input model ID, endpoint URL, and API key or token (depending on which connection option is available)
- Can optionally input a model alias for user-friendly naming
- Can click a "Verify Model" button to test the connection before saving
- Receives immediate feedback on verification: success, or a clear error indicating either connection failure (e.g. unreachable, auth failure) or that the API is not OpenAI compliant
- Sees a clear warning when registering an external model provider that data will be sent outside the cluster
- Can save the external endpoint configuration and see it immediately appear in the AI Assets Endpoint model list
- Receives clear validation feedback if required fields are missing

_Platform Engineer_:
- Can control visibility and external provider access via DataScienceCluster feature flags (see Epic 1)
- Understands that API keys/tokens are securely stored as Kubernetes Secrets
- Understands the security implications of allowing external model connections and can mitigate risks via feature flags

**Success Criteria**:
- **SC-003**: AI Engineers see a warning in the UI when connecting to an external model provider (data sent outside the cluster)
- **SC-004**: AI Engineers can successfully register external model endpoints (per allowed options) and see them appear in AI Available Assets without errors
- **SC-007**: API keys/tokens are stored securely as Kubernetes Secrets and never exposed in plain text in logs or UI
- **SC-010**: Registration form provides clear validation feedback and prevents submission with missing required fields
- **SC-012**: Model verification returns distinct, actionable errors for connection failure (e.g. unreachable, auth failure) vs API not OpenAI compliant

> **⚠️ Dependency**: UX Team must provide design for minimal registration wizard. Core fields: Model ID (required text), Endpoint URL, API Key/Token, Model Alias (optional text), "Verify Model" button, and **warning when connecting to an external model provider**.


---

### Epic 3: External Endpoint Display and Management (Priority: P1, Owner: gen-ai)

Display registered external endpoints in the AI Available Assets page with clear visual distinction by model type. Users can view endpoint details, try a model in Playground or register it in LlamaStack if not already there, and delete external models. Never retrieve the token and pass to the client side.

**Scope**: This epic covers (1) the **UI** for listing external models, (2) the **BFF endpoint** that lists external models (reads the ConfigMap and returns the list), (3) the **BFF endpoint** that registers a model in LlamaStack when it is not already registered, and (4) the **BFF endpoint** that deletes an external model (removes its entry from the ConfigMap).

**User Value**: AI Engineers and Platform Engineers have a centralized view of all available AI assets—both internal and external—and can manage external endpoints as needed.

**Technical Considerations**:
- **UI — List view**: The list shows at minimum model ID, alias, and URL for each external model. Each entry has a button to **try the model in Playground** or **add it (register) in LlamaStack** if not already registered. There must be a **visible distinction** between: (a) models deployed in the user’s namespace, (b) external models pointing inside the cluster (cross-namespace), and (c) external models pointing to a provider (outside the cluster).
- **BFF — List external models**: The BFF reads the ConfigMap (same schema as in Epic 2) and returns the list of external models for the UI.
- **BFF — Register in LlamaStack**: A BFF endpoint allows registering an external model in LlamaStack when it is not already registered, so the user can then use it (e.g. from Playground).
- **BFF — Delete external model**: A BFF endpoint deletes an external model by removing its entry from the ConfigMap. Because all external models are stored as entries in a single ConfigMap, the Kubernetes Secret that holds the model’s API key or token is **not** removed automatically. The BFF will need to delete the secret itself.

**Outcomes by Persona**:

_AI Engineer_:
- Can view external models in the AI Available Assets page alongside internal and in-namespace models
- Can identify at a glance: models in own namespace vs external in-cluster vs external provider
- Can see at minimum model ID, alias, and URL for each external endpoint
- Can use a button to try the model in Playground or add (register) it in LlamaStack if not already there
- Can delete external endpoints

_Platform Engineer_:
- Understands that deletion removes the model entry from the ConfigMap and the associated Secret

**Success Criteria**:
- **SC-005**: AI Engineers can distinguish between models in own namespace, external in-cluster, and external provider in the Assets list at a glance
- **SC-008**: External endpoint deletion removes the endpoint record (ConfigMap entry) and secret via the BFF
- **SC-009**: Users with namespace access can view and use all registered external endpoints (namespace-scoped visibility)
- **SC-013**: List view shows at minimum ID, alias, and URL, with an action to try in Playground or register in LlamaStack if not already there

> **⚠️ Dependency**: UX Team must provide design for the three-way visual distinction (in-namespace / external in-cluster / external provider),


---

### Epic 4: External Endpoint Integration with AI Playground (Priority: P1, Owner: gen-ai)

Make external endpoints selectable in the AI Playground for inference. A BFF endpoint will be created to register the model in the LlamaStack instance used by Playground. The model must appear in the Playground’s models dropdown and be usable for chat. Inference uses the responses API with the credential (API key or token) from the Kubernetes Secret.

**Scope**: This epic covers (1) ensuring the **model can be selected from the models dropdown** in the AI Playground, and (2) the **BFF endpoint** that registers the model in the **LlamaStack instance used by Playground** (so the model is available for inference there).

**User Value**: AI Engineers can use external models in the Playground to test prompts, compare responses, and validate model behavior without leaving Gen AI Studio.

**Technical Considerations**:
- External models must appear in the Playground’s **models dropdown** and be selectable like internal models.
- **BFF — Register in LlamaStack for Playground**: A BFF endpoint registers the external model in the LlamaStack instance used by AI Assets Endpoints page so it can be used for inference (aligned with Epic 3’s register-in-LlamaStack capability; Playground uses its own LlamaStack instance).
- Inference calls the responses API for the external model via the LlamaStack responses API; the API key or token from the Kubernetes Secret must be available when invoking the responses API (e.g. fetched and passed as a parameter).
- OpenAI-compliant API format expected by Playground must be maintained end-to-end. Streaming responses and error handling must work consistently with internal models.

**Outcomes by Persona**:

_AI Engineer_:
- Can select a registered external endpoint from the **models dropdown** in the AI Playground
- Can send inference requests to the external endpoint and receive responses in the Playground UI
- Experiences consistent behavior whether using internal or external models (streaming, error states, response formatting)
- Receives clear error messages if the external service is unreachable or returns an error
- [If scoping mismatch exists] Receives clear error message if attempting to use an endpoint without access to the associated secret

_Platform Engineer_:
- Understands that external requests are authenticated using the stored Kubernetes Secret
- Can troubleshoot failed external requests using standard logging and monitoring tools
- Understanding that document content from RAG and other info in the responses API request is being sent externally
- Understands that this cannot be enabled in an airgap Env

**Success Criteria**:
- **SC-006**: AI Engineers can select external endpoints in the AI Playground and receive inference responses consistently
- **SC-011**: External model inference requests fail gracefully with actionable error messages when external services are unreachable or return errors

> **⚠️ Dependency**: Architecture must clarify authentication mechanism if endpoints and secrets have different scoping (namespace vs user).

---

## Edge Cases

- **Verification fails or is skipped**: User can save an external model without successful "Verify Model"; verification is optional. If verification runs and the endpoint is unreachable or not OpenAI compliant, the UI shows the appropriate distinct error (connection failure vs API not compliant).
- **Expired or invalid API key or token**: Inference fails; the user sees an actionable error (e.g. authentication failure) and the credential is never exposed in logs or UI.
- **Namespace deletion**: ConfigMap and Secrets are namespace-scoped; when a namespace is deleted, Kubernetes removes them. No extra cleanup contract is required.
- **Duplicate model ID or alias in same namespace**: The system must either disallow duplicates or allow them with a way for users to tell entries apart (e.g. URL or alias).
- **Endpoint URL format**: Whether the system validates URL format (e.g. HTTPS) or accepts any URL and relies on verification/inference to fail is a design choice to be made.

---

## Potential Spikes

### Spike 1: Multi-Provider Support and LlamaStack Remote Providers

**Uncertainty**: (1) Whether external model endpoints work correctly with other major providers (e.g. Gemini, Anthropic, AWS) beyond OpenAI. (2) Why [LlamaStack remote inference providers](https://github.com/llamastack/llama-stack/tree/main/src/llama_stack/providers/remote/inference) use adapters if they are supposed to be OpenAI compliant—and what that implies for the registration wizard and runtime.

**Spike Goal**:
- Test connectivity, verification, and inference with representative providers (e.g. Gemini, Anthropic, AWS) to confirm behavior and surface provider-specific requirements.
- Review LlamaStack remote providers and adapter usage; document why adapters exist despite OpenAI compatibility and what the supported “provider types” are.
- **Outcome**: If adapters correspond to distinct provider types (e.g. OpenAI vs Anthropic vs Gemini), recommend adding an **optional dropdown in the model creation wizard** listing the remote provider types, with **OpenAI as the default**, so users can select the appropriate provider when needed.

**Recommended Timebox**: TBD (suggest 2–3 days to cover testing and adapter review)

---

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **UX Team** | (1) Registration wizard: Model ID, URL, API Key/Token, optional Model Alias, Verify Model button, and warning when connecting to an external provider. (2) Assets list: three-way visual distinction (in-namespace / external in-cluster / external provider) and actions (try in Playground, add to LlamaStack). | Design | Blocking for Epic 2 (wizard), Epic 3 (list) |
| **Playground Team (gen-ai)** | BFF and UI implementation for external model registration, display, and playground integration | 
Integration | |
| **Documentation Team** | Admin and user docs: DataScienceCluster feature flags (what each does, how to enable, security risks of external provider), registration workflow, and use in Gen AI Studio | Documentation | Epic 1, 2, 3 |
| **QE Team (gen-ai)** | E2E coverage for feature flags, registration, list/delete/register in LlamaStack, Playground model dropdown and inference. | Testing | Epic 1–4 |

---

## Performance & Scaling

The feature does not impose or guarantee performance or scaling limits for external model providers. Rate limits, quotas, and latency are between the user and the provider; the platform does not manage or enforce them. The only in-scope requirement is that the UI (e.g. Playground) remains responsive when external requests are slow (non-blocking behavior).

---

## Success Criteria

### Measurable Outcomes

- **SC-001**: When the visibility flag is missing or disabled, the external model feature is hidden in the UI; when enabled, only in-cluster cross-namespace (Option B) registration is available unless the external provider flag is enabled
- **SC-002**: Platform engineers can enable or disable each feature flag via the DataScienceCluster CR and have documentation that explains what each flag does, how to enable it, and (for the external provider flag) the security risks of enabling it
- **SC-003**: AI Engineers see a warning in the UI when connecting to an external model provider (data sent outside the cluster)
- **SC-004**: AI Engineers can successfully register external model endpoints (per allowed options) and see them appear in AI Available Assets without errors
- **SC-005**: AI Engineers can distinguish between internal and external models in the Assets list at a glance
- **SC-006**: AI Engineers can select external endpoints in the AI Playground and receive inference responses consistently
- **SC-007**: API keys/tokens are stored securely as Kubernetes Secrets and never exposed in plain text in logs or UI
- **SC-008**: External endpoint deletion removes all associated resources (endpoint record and Secret) from the namespace
- **SC-009**: Users with namespace access can view and use all registered external endpoints (namespace-scoped visibility)
- **SC-010**: Registration form provides clear validation feedback and prevents submission with missing required fields
- **SC-011**: External model inference requests fail gracefully with actionable error messages when external services are unreachable or return errors
- **SC-012**: Model verification returns distinct, actionable errors for connection failure (e.g. unreachable, auth failure) vs API not OpenAI compliant
- **SC-013**: List view shows at minimum ID, alias, and URL, with an action to try in Playground or register in LlamaStack if not already there

---
