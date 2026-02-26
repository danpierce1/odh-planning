# Tasks: External Model Endpoints Support

**Input**: Design documents from `specs/005-name-external-model-endpoints/`
**Prerequisites**: spec.md (required). plan.md not yet present—paths use logical areas (DSC/operator, BFF, frontend); refine with plan.md when available.

**Tests**: Not requested in the feature specification; no test tasks included.

**Organization**: Tasks are grouped by epic to enable independent implementation and testing.

## Format: `[ID] [P?] [Epic?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Epic]**: Which epic this task belongs to (E1–E4)
- Include exact file paths in descriptions; path prefixes (e.g. `bff/`, `frontend/`) assume repo structure from plan.md when created.

## Path Conventions

- **DataScienceCluster CR**: Operator/API codebase that owns the DSC CR (path per plan.md).
- **BFF**: Backend-for-frontend services (e.g. `bff/` or backend package).
- **Frontend**: Gen AI Studio UI (e.g. `frontend/` or `ui/`).
- Adjust paths to match actual repository layout from plan.md.

---

## Phase 1: Epic 1 – Feature Flags for External Model Visibility and Provider Access (Priority: P1)

**Goal**: Platform engineers control visibility and external provider access via DSC CR; AI Engineers see the feature only when visibility is enabled. Includes setup, schema, DSC flags, BFF flag API, UI gating, and documentation.

**Independent Test**: With visibility flag off, external model feature is hidden in UI; with visibility on and provider flag off, only "model in another namespace" is available; with both on, external provider option is available. Documentation explains both flags and security risks.

### Setup, schema, and feature flags (Epic 1)

- [ ] T001 [P] [E1] Define and document external model ConfigMap schema (single ConfigMap per namespace, LlamaStack-aligned) and shared types location for BFF create/list/delete in specs or design docs
- [ ] T002 [E1] Add visibility feature flag (Flag 1) to DataScienceCluster CR—controls whether external model feature is shown in UI; default missing = hidden
- [ ] T003 [E1] Add external model provider feature flag (Flag 2) to DataScienceCluster CR—when disabled, only in-cluster cross-namespace (Option B) allowed
- [ ] T004 [P] [E1] Implement BFF endpoint or config loader to read both feature flags from DataScienceCluster and expose to frontend (e.g. `bff/feature-flags/` or equivalent)

### UI and documentation (Epic 1)

- [ ] T005 [E1] Wire UI to BFF feature-flag API in `frontend/`: hide external model entry point (e.g. AI Available Assets) when visibility flag is missing or disabled; in registration flow show only "model in another namespace" (Option B) when external provider flag is disabled, both options when enabled
- [ ] T006 [E1] Add or update documentation in docs/ (path per plan.md or docs team): (1) what each feature flag does, how to enable via DataScienceCluster CR, and security risks of enabling external model provider; (2) registration workflow and external provider security/airgap note

**Checkpoint**: Epic 1 complete—feature flags in DSC, schema defined, BFF serves flag state, UI visibility and registration options respect flags; docs in place (SC-001, SC-002).

---

## Phase 2: Epic 2 – External Endpoint Registration MVP (Priority: P1)

**Goal**: Users can register external model endpoints (in-cluster cross-namespace and, when allowed, external provider) via a wizard with secure credential storage and verify-before-save.

**Independent Test**: From AI Available Assets, open add flow; enter model ID, URL, API key/token; optionally alias; verify model; see success or distinct errors (connection failure vs not OpenAI compliant); save and see endpoint in list. Warning shown for external provider. Credentials stored in Secrets only.

### Implementation for Epic 2

- [ ] T007 [P] [E2] Implement BFF endpoint to create external model: persist entry to namespace ConfigMap (per T001 schema), store API key/token in namespace Secret; accept model ID, endpoint URL, optional alias, credential; enforce feature flags in `bff/external-models/` (or equivalent)
- [ ] T008 [P] [E2] Implement BFF endpoint to verify external model: send short sample request to endpoint chat completions API with `max_output_tokens`; validate OpenAI-compliant response; return distinct errors for connection failure (unreachable, auth failure) vs API not OpenAI compliant in `bff/external-models/`
- [ ] T009 [E2] Implement registration wizard UI from AI Available Assets in `frontend/` (wizard per UX design): form with required model ID, endpoint URL, API key or token (depending on option), optional model alias, Verify Model button; call create and verify BFF endpoints; show validation feedback for missing required fields; show clear warning when user selects external model provider (Option A) that data from the responses API will be sent outside the cluster
- [ ] T010 [E2] Ensure API keys/tokens are never logged or exposed in UI; credential handling only via Kubernetes Secret in BFF and frontend
- [ ] T011 [E2] Handle duplicate model ID or alias in same namespace: either disallow or allow with clear differentiation (e.g. by URL or alias); implement in BFF create and/or UI validation in `bff/external-models/` and `frontend/`
- [ ] T012 [E2] Confirm URL validation approach (e.g. HTTPS or accept any and rely on verify/inference to fail) and apply in BFF and UI if applicable in `bff/external-models/` and `frontend/`

**Checkpoint**: Epic 2 complete—registration and verification work; credentials in Secrets; warning for external provider; validation and distinct verify errors; duplicate and URL edge cases handled; docs in T006 (SC-003, SC-004, SC-007, SC-010, SC-012).

---

## Phase 3: Epic 3 – External Endpoint Display and Management (Priority: P1)

**Goal**: List external endpoints in AI Available Assets with clear distinction (in-namespace vs external in-cluster vs external provider); view details; try in Playground or register in LlamaStack; delete external models. Token never sent to client.

**Independent Test**: External models appear in list with ID, alias, URL; three-way visual distinction; actions “try in Playground” and “add to LlamaStack” (if not already); delete removes entry and Secret.

### Implementation for Epic 3

- [ ] T013 [P] [E3] Implement BFF endpoint to list external models: read namespace ConfigMap (per T001 schema), return list of external models (id, alias, URL, type); do not return token/API key in `bff/external-models/`
- [ ] T014 [P] [E3] Implement BFF endpoint to delete external model: remove entry from ConfigMap and delete associated Kubernetes Secret in `bff/external-models/`
- [ ] T015 [P] [E3] Implement BFF endpoint to register external model in LlamaStack when not already registered (for use from Playground or other consumers) in `bff/external-models/`
- [ ] T016 [E3] Implement list view UI for external models on AI Available Assets in `frontend/` (assets list per UX design): show at least model ID, alias, URL; three-way visual distinction (in-namespace / external in-cluster / external provider); actions try in Playground, add to LlamaStack if not already; wire list to BFF list endpoint, delete to BFF delete endpoint, and add to LlamaStack to BFF register-in-LlamaStack endpoint
- [ ] T017 [E3] Add or update documentation for list and management of external endpoints in docs/ (path per plan.md or docs team)

**Checkpoint**: Epic 3 complete—list, delete, register in LlamaStack; clear visual distinction; no token to client; list/management docs in place (SC-005, SC-008, SC-009, SC-013).

---

## Phase 4: Epic 4 – External Endpoint Integration with AI Playground (Priority: P1)

**Goal**: External endpoints are selectable in the AI Playground models dropdown and usable for inference; BFF registers model in the LlamaStack instance used by Playground; inference uses responses API with credential from Secret.

**Independent Test**: Select a registered external endpoint from Playground models dropdown; send chat request; receive response (or actionable error if unreachable). Behavior consistent with internal models (streaming, errors).

### Implementation for Epic 4

- [ ] T018 [E4] Implement BFF endpoint (or extend existing) to register external model in the LlamaStack instance used by AI Playground so model appears in Playground’s model list in `bff/` (Playground/LlamaStack integration)
- [ ] T019 [E4] Ensure external models returned from list/registration flow appear in AI Playground models dropdown and are selectable in `frontend/` Playground UI
- [ ] T020 [E4] Ensure inference path uses responses API with API key/token from Kubernetes Secret (fetched server-side, never sent to client); handle streaming and errors consistently with internal models in BFF/Playground backend
- [ ] T021 [E4] Surface actionable error messages in Playground when external service is unreachable or returns errors (no credential exposure) in `frontend/` Playground UI

- [ ] T022 [E4] Add or update documentation for Playground usage of external endpoints and security/airgap note in docs/ (path per plan.md or docs team)

**Checkpoint**: Epic 4 complete—external endpoints selectable in Playground and inference works with graceful errors; Playground docs in place (SC-006, SC-011).

---

## Dependencies & Execution Order

### Phase Dependencies

- **Epic 1 (Phase 1)**: No dependencies—can start immediately (schema, flags, BFF, UI, docs).
- **Epic 2 (Phase 2)**: Depends on Epic 1 (registration only shown when visibility enabled; uses T001 schema).
- **Epic 3 (Phase 3)**: Depends on Epic 2 (list/delete/register operate on created endpoints).
- **Epic 4 (Phase 4)**: Depends on Epic 2 and Epic 3 (Playground uses registered models and list).

### Epic Dependencies

- **Epic 1 (Feature flags)**: No dependencies.
- **Epic 2 (Registration)**: After Epic 1 (UI visibility and option set).
- **Epic 3 (Display & management)**: After Epic 2 (need create + storage).
- **Epic 4 (Playground)**: After Epic 2, Epic 3 (need models registered and list/register-in-LlamaStack).

### Parallel Opportunities

- T001 can run in parallel with T002–T004 within Phase 1 (schema vs flags/BFF).
- T007, T008 can run in parallel (Phase 2).
- T013, T014, T015 can run in parallel (Phase 3).

---

## Parallel Example: Epic 2

```text
# BFF create and verify endpoints (different handlers/files):
T007: Implement BFF create external model in bff/external-models/
T008: Implement BFF verify external model in bff/external-models/
```

## Parallel Example: Epic 3

```text
# BFF list, delete, register-in-LlamaStack (separate endpoints):
T013: BFF list external models
T014: BFF delete external model
T015: BFF register external model in LlamaStack
```

---

## Implementation Strategy

### MVP First (Epics 1 + 2)

1. Complete Phase 1: Epic 1 (schema, DSC flags, BFF flag API, UI visibility, docs).
2. Complete Phase 2: Epic 2 (registration + verify + wizard + warning).
3. **STOP and VALIDATE**: Registration and visibility work end-to-end; credentials in Secrets.
4. Deploy/demo registration and feature-flag behavior.

### Incremental Delivery

1. Complete Epic 1 (Phase 1) → Feature visible and options correct → Demo.
2. Add Epic 2 (Phase 2) → Register and verify (including edge cases and docs) → Demo (MVP).
3. Add Epic 3 (Phase 3) → List, delete, register in LlamaStack (including docs) → Demo.
4. Add Epic 4 (Phase 4) → Playground dropdown and inference (including docs) → Demo.

### Suggested MVP Scope

- **MVP**: Phase 1 (Epic 1) + Phase 2 (Epic 2)—feature flags and registration with verify and secure storage. Delivers SC-001, SC-002, SC-003, SC-004, SC-007, SC-010, SC-012.

---

## Notes

- [P] tasks = different files or endpoints, no dependencies on each other.
- [E1]–[E4] labels map tasks to Epic 1–4 for traceability.
- Each epic is independently testable per checkpoints.
- plan.md: Once created, replace logical paths (DataScienceCluster CR, `bff/`, `frontend/`) with actual repo and file paths.
- UX dependency: Registration wizard (Epic 2) and list visual distinction (Epic 3) depend on UX design; implementation should follow provided designs.
