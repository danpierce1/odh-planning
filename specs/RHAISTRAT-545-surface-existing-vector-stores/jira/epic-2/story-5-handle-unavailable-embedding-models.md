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

For this story, just ensure that the backend surfaces to the UI which external vector stores cannot be enabled due to not having an active associated embedding model (this may be covered by the embedding_model_available field recently added, so verify that it will be derived correctly based on the params passed to the Install LSD (Create Playground) endpoint).

During LSD install, the BFF can cross-check each vector store's required embedding model against the models supplied to the install step. If the embedding model for a store is not available, that store needs to be excluded from LSD registration but LSD install is allowed to complete.

Implementation approach may be informed by the spike findings (Story 2).

## Acceptance Criteria

- [ ] During LSD install, the BFF cross-checks each vector store's required embedding model against the models available in the current install.
- [ ] If a store's embedding model is unavailable we should fail the Playground creation entirely in this scenario with explanation.
- [ ] The backend response should enable the UI to know which vector stores cannot be enabled due to embedding model not being available (see if embedding_model_available field from Eder POC is useful here)
