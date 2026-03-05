# Story: Update View Code Output to Include Vector Store and Embedding Model Details

## Jira Fields

| Field          | Value                                                                                          |
|----------------|------------------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                          |
| Summary        | Update View Code Output to Include Vector Store and Embedding Model Details                    |
| Priority       | Normal                                                                                         |
| Project        | RHOAIENG                                                                                       |
| Components     | AI Core Dashboard, Gen AI Studio                                                               |
| Labels         | dashboard-crimson-scrum                                                                        |
| Activity Type  | New Features                                                                                   |
| Target Version | rhoai-3.4                                                                                      |
| Epic Link      | RHOAIENG-51774                                                                                 |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-52021                                                |

---

## Jira Description

As an AI Engineer, I want the View Code output in the Playground to include the selected vector store collection and embedding model details, so that I can reproduce the RAG query outside the Playground without having to manually look up the configuration.

When a vector store is active in a chat session, the View Code panel should be updated to reflect the selected collection, the embedding model used, and any relevant configuration needed to reproduce the retrieval query externally.

A spike may be needed to determine the exact format of the View Code output for vector store and embedding model details.

## Acceptance Criteria

- [ ] When a vector store is selected and active in a chat pane, the View Code output includes the selected collection/vector store details.
- [ ] The View Code output includes the embedding model used for the active vector store.
- [ ] The View Code output includes relevant configuration for reproducing the query outside the Playground.
- [ ] When no vector store is active, View Code output is unchanged from current behaviour.

## Notes

- A spike may be needed to determine the exact format of the View Code output for vector store + embedding model details.
- Depends on Story 1 (RHOAIENG-51775, Knowledge Tab) being in place.
- Depends on Epic 1 (RHOAIENG-51445, feature flag) and Epic 2 (RHOAIENG-51472, BFF ConfigMap loading) being in place.
