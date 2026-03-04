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
| Parent Link    | RHAISTRAT-545                                                                      |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51779                                    |

---

## Jira Description

Similar to the existing Add a model flow in AAE, add support for viewing external vector stores in an AAE page (under a vector stores tab) and selecting one or more to be included when configuring a Playground via an Add to Playground link.

Engineers can view external vector stores in more detail and select them for inclusion when installing a Playground (can be fresh install or replacing an existing playground, either way both go through the install endpoint, but the latter first runs a delete playground request).

The gen-ai-aa-vector-stores ConfigMap is used as a read-only reference for what external vector stores can be viewed and selected (in future we will likely support the User to Register a vector store from the UI). The ConfigMap is not modified by this flow — only Platform Engineers manage its contents.

**Note**: This epic is a late addition to this strat and has not yet been reviewed or approved by PM/Design. Details may change pending that review.

**Needs clarification**: confirm if external vector store selection should also be included in the Update Configuration modal flow in playground.

**Outcomes by Persona**

AI Engineer:
- Can view all external vector stores defined in the gen-ai-aa-vector-stores ConfigMap in the AAE page under vector stores tab. The primary column for the table should be Collection, and we should also have columns for Provider, Embedding Model ID/Name, Domain, Status, Dimensions, Distance, metric, Created, Endpoint, Playground.
- For 3.4, a user can click Add to Playground for a vector store collection if the associated embedding model is added as an AI asset endpoint (check the list of AI Asset endpoints, if it's not present we show the row as greyed out, with details on how to add the embedding model as an endpoint). In the modal that opens, show the list of vector stores and associated embedding model next to each store. If an embedding model is not added to AI Asset endpoints we also here show the entire row as greyed out. If it is added but not registered, indicate that the we will auto register it with llamastack. The user may then proceed with installing the vector store and embedding model in the playground by clicking the Configure button.

## Acceptance Criteria

- [ ] External vector stores defined in the gen-ai-aa-vector-stores ConfigMap are visible in the AAE page under a Vector Stores tab, with columns for Collection, Provider, Embedding Model ID/Name, Domain, Status, Dimensions, Distance, metric, Created, Endpoint, Playground. The existing BFF VectorStoresAAHandler endpoint in aaa_vectorstores_handler.go can be used to retrieve this list.
- [ ] For 3.4, in order to add Vector store collections to a playground, the user must use the Vector Stores tab under AAE page to select the vector stores for the playground.
- [ ] A user can click Add to Playground for a vector store collection if the associated embedding model is added as an AI asset endpoint. If it's not, the row is shown as greyed out with details on how to add the embedding model as an endpoint.
- [ ] In the modal that opens, show the list of vector store collections and associated embedding model next to each store. If an embedding model is not added to AI Asset endpoints again show the entire row as greyed out. If it is added but not registered, indicate that the backend will auto register it with llamastack.
- [ ] The user can proceed with installing the vector store collections and associated embedding models in the Playground by clicking the Configure button, which triggers Playground install (note we already have logic whereby if playground already exists, the UI first makes request to delete the existing playground) by making a request using the existing installLSD method call in ChatbotConfigurationModal.tsx, passing the vector store(s) and models in the request (to the LlamaStackDistributionInstallHandler endpoint).
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by this flow.

## Stories

- Story 1: Display External Vector Stores in the AAE Vector Stores Tab
- Story 2: Add to Playground Flow — Select and Launch a Playground with Vector Stores from AAE
