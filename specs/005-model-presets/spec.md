# Feature Specification: Model Presets

**Feature Branch**: `005-model-presets`
**Created**: 2026-01-21
**Status**: Draft
**Input**: User description: "Preset catalog for deploying models with validated configurations, reducing deployment from hours of research to minutes"

## Epics *(mandatory)*

### Epic 1: Preset Catalog Backend (Priority: P1, Owner: Platform/Backend)

Provide a versioned API service that delivers platform-curated model presets, enabling the Dashboard to fetch validated deployment configurations in real-time.

**User Value**: Users gain access to pre-validated model configurations without manual research, with confidence that presets are current and version-tracked.

**Technical Considerations**:
- Presets must be decoupled from Dashboard releases (backend service or external catalog)
- Each preset must be versioned for traceability and auditability
- API must support real-time fetch with appropriate caching strategy

**Outcomes by Persona**:

_AI Engineer_:
- Access a catalog of platform-curated model presets via API
- Receive versioned presets with clear metadata about what changed between versions

_Platform Engineer_:
- Presets are managed independently of Dashboard deployment cycles
- Preset updates are available to users without requiring Dashboard upgrades

> **Dependency**: Platform/Backend team must provide preset catalog API before UI integration can begin.

---

### Epic 2: Preset Selection UI (Priority: P1, Owner: Dashboard/gen-ai)

Integrate a preset selection step within the existing model deployment flow, allowing users to browse, search, and select from the preset catalog.

**User Value**: Users can discover and select validated model configurations directly within the deployment flow, eliminating context switching to external documentation.

**Technical Considerations**:
- Must integrate within existing "Deploy model" flow, not a separate page
- Search/filter must be performant even as catalog grows to 20-30 presets
- UI must clearly indicate preset version being applied

**Outcomes by Persona**:

_AI Engineer_:
- Browse a searchable catalog of model presets during deployment
- Filter presets by model name, category, or resource requirements
- See preset metadata (model name, registry image, resource tiers) before selection

_ML Ops Engineer_:
- Quickly identify which presets are available for specific hardware configurations
- View resource requirements upfront to assess cluster fit

---

### Epic 3: Configuration Auto-Population (Priority: P1, Owner: Dashboard/gen-ai)

When a preset is selected, automatically populate the deployment form with validated configuration values while allowing user modifications before final deployment.

**User Value**: Users skip manual configuration research and start from a known-good baseline, with full control to adjust values for their specific needs.

**Technical Considerations**:
- Form state must update atomically when preset is applied
- User modifications must be tracked separately from preset defaults (for potential future "reset to preset" functionality)
- Tiered options (optimal/recommended/minimum) require tier selection UI before auto-population

**Outcomes by Persona**:

_AI Engineer_:
- Select a preset and see deployment form auto-populated with validated values
- Choose between resource tiers (optimal, recommended, minimum) based on available hardware
- Modify any preset value before deploying (preset is a starting point, not a constraint)

_Platform Engineer_:
- Review auto-suggested naming conventions before deployment
- See clear indication of which values came from preset vs. manual entry

---

### Epic 4: Resource Validation & Warnings (Priority: P1, Owner: Dashboard/gen-ai)

Validate preset resource requirements against available cluster resources and provide clear warnings when mismatches occur, without blocking deployment.

**User Value**: Users are informed of potential issues before deployment, reducing failed deployments while preserving flexibility to proceed if they understand the risks.

**Technical Considerations**:
- Validation must check GPU type/count, memory, and CPU against cluster availability
- Warnings must be non-blocking (user can proceed at their own risk)
- Unavailable presets (missing quay.io image) must be visually distinct in catalog

**Outcomes by Persona**:

_AI Engineer_:
- Receive clear warning if selected tier requires resources unavailable in the cluster
- See alternative tier suggestions when primary tier cannot be satisfied
- Proceed with deployment despite warnings when intentional (e.g., testing on smaller hardware)

_ML Ops Engineer_:
- Quickly identify which presets are viable for the current cluster configuration
- Understand resource gaps before deployment attempts

---

### Epic 5: Preset Catalog Expansion (Priority: P2, Owner: Platform/Backend)

Expand the initial MVP catalog (2-3 presets) to a medium-sized catalog (20-30 presets) covering popular models across categories (chat, code, embedding, etc.).

**User Value**: Users have validated presets for a broad range of common use cases, reducing the need for manual configuration research across model categories.

**Outcomes by Persona**:

_AI Engineer_:
- Access presets for popular chat models (Llama 3, Mistral, Granite)
- Access presets for code generation models
- Access presets for embedding models

_Data Scientist_:
- Find presets organized by use case category for easier discovery

---

### Edge Cases

- What happens when a preset requires resources unavailable in the cluster? Warn the user but allow deployment to proceed (user accepts risk).
- What happens when a preset references a quay.io image that no longer exists? Show preset in catalog but mark as "unavailable"; display error only if user attempts to deploy.
- What happens if the preset catalog API is unreachable? Show cached presets if available; display error state if no cache exists; allow manual deployment flow as fallback.
- What happens when multiple tiers are available but all exceed cluster resources? Show all tiers with warnings; suggest contacting admin to provision additional resources.
- What happens if a user modifies preset values then selects a different preset? New preset values overwrite all form fields; consider confirmation dialog for destructive change.

## Performance & Scaling

| Concern | Impact | Consideration |
|---------|--------|---------------|
| **Catalog Fetch Latency** | Preset catalog loads during deployment flow | Real-time fetch should complete quickly; consider caching strategy |
| **Catalog Growth** | MVP: 2-3 presets → Target: 20-30 presets | Search/filter must remain performant; pagination may be needed for larger catalogs |
| **Resource Validation** | Cluster resource check on preset selection | Validation should be fast; consider background fetch of cluster state |
| **Concurrent Users** | Multiple users fetching presets simultaneously | Backend service must handle expected user concurrency |

## System Constraints

- Presets are platform-provided only; no user-created presets in v1
- Quay.io registry only; no multi-registry support in v1
- Presets are versioned but no UI to select older versions in v1
- Each preset includes three resource tiers: optimal, recommended, minimum
- Preset catalog is fetched in real-time; presets are versioned for traceability

## Key Entities

- **Model Preset**: A platform-curated configuration template containing model metadata, resource requirements, and serving parameters for a specific model
- **Preset Version**: A specific revision of a preset, enabling traceability and auditability of configuration changes
- **Resource Tier**: One of three configuration levels (optimal, recommended, minimum) within a preset, representing different resource/performance tradeoffs
- **Preset Catalog**: The complete collection of available model presets, served via API
- **Serving Parameters**: Model-specific settings within a preset including context length, quantization, and batch size

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can browse and search model presets within the deployment flow
- **SC-002**: Selecting a preset auto-populates the deployment configuration form with validated values
- **SC-003**: Users can choose between resource tiers (optimal, recommended, minimum) for each preset
- **SC-004**: Users can modify any preset value before deploying
- **SC-005**: Users receive clear warnings when preset requirements exceed available cluster resources
- **SC-006**: Presets with unavailable registry images are visibly marked in the catalog
- **SC-007**: Deployment success rate improves for users using presets compared to manual configuration (primary success metric)
- **SC-008**: Adoption rate of preset-based deployment increases over time (primary success metric)

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **Model Serving** | Presets must map to compatible ServingRuntime | Integration | Presets are runtime-agnostic; system matches to available runtimes |
| **Platform/Backend** | Preset catalog API service | Integration | New API endpoint(s) to serve versioned presets; blocking dependency |
| **UX Team** | Preset selection UI within deployment flow | Design | Search, tier selection, warnings display; blocking dependency |
| **QE Team** | E2E test coverage for preset-based deployment | Testing | Happy path + edge cases (warnings, unavailable presets) |

### Blocking Dependencies

- **Platform/Backend**: Preset catalog API must be available before Dashboard UI work can complete
- **UX Team**: Designs for preset selection, tier selection, and warning states needed before UI implementation

### Informational (Capacity/Awareness)

- **Model Serving**: FYI on runtime-agnostic preset approach; system will match presets to available ServingRuntimes

## Clarifications

### Session 2026-01-21

- Q: Which registries are supported? → A: Quay.io only for initial scope
- Q: Who manages presets? → A: Platform-provided only; no user-created presets in v1
- Q: How do presets specify serving runtime? → A: Runtime-agnostic; presets specify requirements, system matches to available runtime
- Q: How are GPU requirements handled? → A: Tiered options (optimal, recommended, minimum); user chooses tier
- Q: Where does preset UI appear? → A: Embedded within existing deployment flow, not a standalone page
- Q: How many presets initially? → A: MVP with 2-3 presets, roadmap to 20-30
- Q: Where are presets stored? → A: Backend service or external catalog (decoupled from Dashboard releases)
- Q: How are presets updated? → A: Real-time fetch, but versioned for traceability
- Q: What happens when resources are unavailable? → A: Warn but allow deployment (user accepts risk)
- Q: What happens when registry image is missing? → A: Show preset but mark as "unavailable"
- Q: What's out of scope for v1? → A: User-created presets, multi-registry, admin/org presets, version selection UI, offline bundles

## Assumptions

- A backend service or external catalog for presets will be developed by the Platform/Backend team
- The existing deployment flow can accommodate an additional preset selection step without major refactoring
- Cluster resource information (GPU types, memory, etc.) is available to the Dashboard for validation
- Preset versioning will be transparent to users (they get the latest version; version history is for auditability, not selection)
- The Model Serving team's runtime matching logic can accept requirement-based specifications from presets
- Quay.io availability and image existence can be verified at catalog display time or deployment time
