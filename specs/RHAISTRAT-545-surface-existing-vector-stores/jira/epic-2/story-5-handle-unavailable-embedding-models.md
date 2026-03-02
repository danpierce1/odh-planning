# Story: Handle External Vector Stores with Unavailable Embedding Models During LSD Install

## Jira Fields

| Field          | Value                                                                                          |
|----------------|------------------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                          |
| Summary        | Handle External Vector Stores with Unavailable Embedding Models During LSD Install             |
| Priority       | Normal                                                                                         |
| Project        | RHOAIENG                                                                                       |
| Components     | AI Core Dashboard, Gen AI Studio                                                               |
| Labels         | dashboard-crimson-scrum                                                                        |
| Activity Type  | New Features                                                                                   |
| Target Version | rhoai-3.4                                                                                      |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                                            |

---

## Jira Description

As an AI Engineer, I want to see all external vector stores in the UI, even those that do not have an available embedding model currently active (they will show as greyed out - which will be handled in Epic 3 "An external vector store can be enabled for chat session in UI").

For this story, just ensure that the backend surfaces to the UI which external vector stores cannot be enabled due to not having an active associated embedding model (this may be covered by the embedding_model_available field recently added, so verify that it will be derived correctly based on the params passed to the Install LSD (Create Playground) endpoint). 

During LSD install, the BFF can cross-check each vector store's required embedding model against the models supplied to the install step. If the embedding model for a store is not available, that store needs to be excluded from LSD registration but LSD install is allowed to complete.

Implementation approach may be informed by the spike findings (Story 2).

## Acceptance Criteria

- [ ] During LSD install, the BFF cross-checks each vector store's required embedding model against the models available in the current install.
- [ ] If a store's embedding model is unavailable, the store is skipped during LSD registration — it is not registered with llamastack. ⚠️ **Needs confirmation**: an alternative approach of failing Playground creation entirely in this scenario is still under discussion.
- [ ] LSD install completes successfully even when one or more stores are skipped due to a missing embedding model. ⚠️ **Needs confirmation**: depends on resolution of above.
- [ ] The backend response should enable the UI to know which vector stores cannot be enabled due to embedding model not being available (see embedding_model_available field)
