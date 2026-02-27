# Epic: Add External Vector Stores to AI Asset Endpoints (AAE) Page

## Jira Fields

| Field         | Value                                                                              |
|---------------|------------------------------------------------------------------------------------|
| Issue Type    | Epic                                                                               |
| Summary       | Add External Vector Stores to AI Asset Endpoints (AAE) Page                        |
| Epic Name     | Add External Vector Stores to AI Asset Endpoints (AAE) Page                        |
| Priority      | Major                                                                              |
| Project       | RHOAIENG                                                                           |
| Components    | AI Core Dashboard, Gen AI Studio                                                   |
| Labels        | dashboard-crimson-scrum                                                            |
| Activity Type | New Features                                                                       |

---

## Jira Description

Similar to the existing Add a model flow in AAE, add support for viewing external vector stores in the AAE page and selecting one or more to be included when creating a new Playground via an Add to Playground link.

Engineers can view external vector stores in more detail and select them for inclusion when creating a new Playground.

The gen-ai-aa-vector-stores ConfigMap is used as a read-only reference for what external vector stores can be viewed and selected. The ConfigMap is not modified by this flow — only Platform Engineers manage its contents.

**Note**: This epic is a late addition to this strat and has not yet been reviewed or approved by PM/Design. Details may change pending that review.

**Outcomes by Persona**

AI Engineer:
- Can view all registered external vector stores in the AAE page with key metadata (provider, embedding model, domain, status).
- Can select one or more vector stores when creating a new Playground from AAE, using the same Configure Playground modal used for model selection.

## Acceptance Criteria

- [ ] External vector stores are visible in a Vector Stores tab in the AAE page (feature flag gated).
- [ ] The Configure Playground modal is updated to include a Knowledge Sources section with selectable vector stores.
- [ ] Selected vector stores are passed to the create Playground backend and registered via the Epic 2 LSD install logic.
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by this flow.

## Stories

- Story 1: Display External Vector Stores in the AAE Vector Stores Tab
- Story 2: Add to Playground Flow — Select and Launch a Playground with Vector Stores from AAE
