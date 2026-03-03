# Story: Add to Playground Flow — Select and Launch a Playground with Vector Stores from AAE

## Jira Fields

| Field          | Value                                                                                          |
|----------------|------------------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                          |
| Summary        | Add to Playground Flow — Select and Launch a Playground with Vector Stores from AAE            |
| Priority       | Normal                                                                                         |
| Project        | RHOAIENG                                                                                       |
| Components     | AI Core Dashboard, Gen AI Studio                                                               |
| Labels         | dashboard-crimson-scrum                                                                        |
| Activity Type  | New Features                                                                                   |
| Target Version | rhoai-3.4                                                                                      |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                                            |

---

## Jira Description

As an AI Engineer, I want to click Add to Playground from the AAE Vector Stores tab and configure a Playground with the selected vector store and its associated embedding model, so that I can quickly launch a RAG-enabled Playground without manual configuration.

When a user clicks Add to Playground for a vector store row, a modal opens showing: the embedding model ID (and name if available) tied to the vector store, whether that embedding model is already available in llamastack (if not, indicate it will be automatically registered), and the selected vector store collection. The user then clicks Configure to proceed, which triggers the LSD install with the vector store and embedding model registered.

The gen-ai-aa-vector-stores ConfigMap is not modified by this flow — it is used only as a read-only reference.

## Acceptance Criteria

- [ ] Clicking Add to Playground from the AAE Vector Stores tab opens a modal showing: the selected vector store collection, the associated embedding model ID and name (if available), and whether the embedding model is already available in llamastack.
- [ ] If the embedding model is not yet available in llamastack, the modal indicates it will be automatically registered.
- [ ] The user can proceed by clicking Configure, which triggers Playground creation with the vector store and embedding model registered via the Epic 2 LSD install logic.
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by this flow.

## Notes

- This epic is pending PM/Design review — implementation details may change.
- **Needs clarification**: confirm if external vector store selection should also be included in the Update Configuration modal flow in playground.
- Depends on Story 1 (AAE Vector Stores tab) being in place.
- Depends on Epic 2 (BFF LSD install) for the backend registration of the selected stores.
