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

As an AI Engineer, I want to select one or more external vector stores from the AAE page and include them when creating a new Playground, so that I can quickly launch a RAG-enabled Playground without having to configure knowledge sources after the fact.

When a user clicks Add to Playground from the AAE Vector Stores tab, the existing Configure Playground modal opens. This modal is updated to include a Knowledge Sources section listing selectable external vector stores alongside the existing Available Models list. The user selects their desired models and vector stores, then confirms to create the Playground.

The selected vector stores are passed to the create Playground backend, which registers them during the LSD install phase (Epic 2). The gen-ai-aa-vector-stores ConfigMap is not modified by this flow — it is used only as a read-only reference for what is available to select.

## Acceptance Criteria

- [ ] Clicking Add to Playground from the AAE Vector Stores tab opens the existing Configure Playground modal.
- [ ] The Configure Playground modal is updated to include a Knowledge Sources section with a list of selectable external vector stores.
- [ ] User can select from both Available Models and Knowledge Sources in the modal before creating the Playground.
- [ ] Selected vector stores are passed to the create Playground backend and registered via the Epic 2 LSD install logic.
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by this flow.

## Notes

- This epic is pending PM/Design review — implementation details may change.
- Depends on Story 1 (AAE Vector Stores tab) being in place.
- Depends on Epic 2 (BFF LSD install) for the backend registration of the selected stores.
