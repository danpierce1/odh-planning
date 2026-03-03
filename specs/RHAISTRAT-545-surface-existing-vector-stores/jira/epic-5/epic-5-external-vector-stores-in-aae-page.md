# Epic: Add External Vector Stores to AI Asset Endpoints (AAE) Page

## Jira Fields

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Issue Type     | Epic                                                                               |
| Summary        | Add External Vector Stores to AI Asset Endpoints (AAE) Page                        |
| Epic Name      | Add External Vector Stores to AI Asset Endpoints (AAE) Page                        |
| Priority       | Major                                                                              |
| Project        | RHOAIENG                                                                           |
| Components     | AI Core Dashboard, Gen AI Studio                                                   |
| Labels         | dashboard-crimson-scrum                                                            |
| Activity Type  | New Features                                                                       |
| Target Version | rhoai-3.4                                                                          |

---

## Jira Description

Similar to the existing Add a model flow in AAE, add support for viewing external vector stores in an AAE page (under a vector stores tab) and selecting one or more to be included when configuring a Playground via an Add to Playground link.

Engineers can view external vector stores in more detail and select them for inclusion when configuring a Playground.

The gen-ai-aa-vector-stores ConfigMap is used as a read-only reference for what external vector stores can be viewed and selected. The ConfigMap is not modified by this flow — only Platform Engineers manage its contents.

**Note**: This epic is a late addition to this strat and has not yet been reviewed or approved by PM/Design. Details may change pending that review.

**Needs clarification**: confirm if external vector store selection should also be included in the Update Configuration modal flow in playground.

**Outcomes by Persona**

AI Engineer:
- Can view all external vector stores defined in the gen-ai-aa-vector-stores ConfigMap in the AAE page under vector stores tab. The primary column for the table should be Collection, and we should also have columns for Provider, Embedding Model ID/Name, Domain, Status, Dimensions, Distance, metric, Created, Endpoint, Playground.
- For 3.4, a user can click Add to Playground for a vector store collection if the associated embedding model is registered as an AI asset endpoint (if it's not we show the row as greyed out, with details on how to add the embedding model as an endpoint). In the modal that then opens, we show the embedding model id (and name if available) tied to the vector store, and whether it's available yet in llamastack (if not we indicate it will be automatically be registered in llamastack), and we show the vector store collection that the user selected. The user may then proceed with installing the vector store and embedding model in the playground by clicking the Configure button.

## Acceptance Criteria

- [ ] External vector stores defined in the gen-ai-aa-vector-stores ConfigMap are visible in the AAE page under a Vector Stores tab, with columns for Collection, Provider, Embedding Model ID/Name, Domain, Status, Dimensions, Distance, metric, Created, Endpoint, Playground
- [ ] A user can click Add to Playground for a vector store collection if the associated embedding model is registered as an AI asset endpoint. If it's not, the row is shown as greyed out with details on how to add the embedding model as an endpoint.
- [ ] In the modal that opens, the embedding model ID (and name if available) tied to the vector store is shown, along with whether it's available yet in llamastack (if not, indicate it will be automatically registered). The selected vector store collection is also shown.
- [ ] The user can proceed with installing the vector store and embedding model in the Playground by clicking the Configure button.
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by this flow.

## Stories

- Story 1: Display External Vector Stores in the AAE Vector Stores Tab
- Story 2: Add to Playground Flow — Select and Launch a Playground with Vector Stores from AAE
