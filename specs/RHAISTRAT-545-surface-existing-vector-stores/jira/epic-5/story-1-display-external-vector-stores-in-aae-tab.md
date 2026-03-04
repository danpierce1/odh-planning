# Story: Display External Vector Stores in the AAE Vector Stores Tab

## Jira Fields

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                              |
| Summary        | Display External Vector Stores in the AAE Vector Stores Tab                        |
| Priority       | Normal                                                                             |
| Project        | RHOAIENG                                                                           |
| Components     | AI Core Dashboard, Gen AI Studio                                                   |
| Labels         | dashboard-crimson-scrum                                                            |
| Activity Type  | New Features                                                                       |
| Target Version | rhoai-3.4                                                                          |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                                |

---

## Jira Description

As an AI Engineer, I want to see all registered external vector store collections listed in the AAE page under a Vector Stores tab, so that I can review their details and optionally add one to a Playground.

When the feature flag is enabled (Epic 1), a Vector Stores tab is added to the AAE page. It lists all external vector store collections from the gen-ai-aa-vector-stores ConfigMap using the existing BFF VectorStoresAAHandler endpoint in aaa_vectorstores_handler.go. The primary column is Collection, with additional columns for Provider, Embedding Model ID/Name, Domain, Status, Dimensions, Distance Metric, Created, Endpoint, and Playground.

Rows where the associated embedding model is not registered as an AI asset endpoint are shown as greyed out, with details on how to add the embedding model as an endpoint.

## Acceptance Criteria

- [ ] A Vector Stores tab is added to the AAE page, visible only when the feature flag is enabled.
- [ ] The tab lists all external vector stores from the ConfigMap with columns: Collection, Provider, Embedding Model ID/Name, Domain, Status, Dimensions, Distance Metric, Created, Endpoint, Playground.
- [ ] Rows where the associated embedding model is not added as an AI asset endpoint are shown as greyed out with an explanatory message on how to add the embedding model as an endpoint
- [ ] An Add to Playground action is available for non-greyed-out rows (behaviour implemented in Story 2).

## Notes

- This story is pending PM/Design review — implementation details may change.
- Depends on Epic 1 (feature flag) and Epic 2 (BFF ConfigMap loading) being in place.
