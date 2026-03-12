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
| Epic Link      | RHOAIENG-51779                                                                     |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51780                                    |

---

## Jira Description

As an AI Engineer, I want to see all registered external vector store collections listed in the AAE page under a Vector Stores tab, so that I can review their details and optionally add one to a Playground.

When the feature flag is enabled (Epic 1, RHOAIENG-51445), a Vector Stores tab is added to the AAE page. It lists all external vector store collections from the gen-ai-aa-vector-stores ConfigMap using the existing BFF VectorStoresAAHandler endpoint in aaa_vectorstores_handler.go. The table shows columns for Vector store collection, Type, Embedding Model, Created, Dimensions, Distance metric, and Playground.

Rows where the associated embedding model is not registered as an AI asset endpoint are shown as greyed out, with details on how to add the embedding model as an endpoint.

## Acceptance Criteria

- [ ] A Vector Stores tab is added to the AAE page, visible only when the feature flag is enabled.
- [ ] The tab lists all external vector stores from the ConfigMap with columns: Vector store collection, Type, Embedding Model, Created, Dimensions, Distance metric, Playground.
- [ ] Rows where the associated embedding model is not added as an AI asset endpoint are shown as greyed out with an explanatory message on how to add the embedding model as an endpoint
- [ ] An Add to Playground action is available for non-greyed-out rows (behaviour implemented in Story 2, RHOAIENG-51781).

## Notes

- This story is pending PM/Design review — implementation details may change.
- Depends on Epic 1 (RHOAIENG-51445, feature flag) and Epic 2 (RHOAIENG-51472, BFF ConfigMap loading) being in place.
