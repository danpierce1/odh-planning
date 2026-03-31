# Epic: Add External Vector Stores to AI Asset Endpoints (AAE) Page and update Configure Playground modal

## Jira Fields

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Issue Type     | Epic                                                                               |
| Summary        | Add External Vector Stores to AI Asset Endpoints (AAE) Page...                     |
| Epic Name      | Add External Vector Stores to AI Asset Endpoints (AAE) Page...                     |
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

Similar to the existing Add a model flow in AAE, add support for viewing external vector stores in an AAE page (under a vector stores tab) and selecting one or more to be included from the Configure Playground modal (which will also be updated).

Engineers can view external vector stores in more detail and select them for inclusion when installing a Playground (can be fresh install or replacing an existing playground, either way both go through the install endpoint, but the latter first runs a delete playground request).

The gen-ai-aa-vector-stores ConfigMap is used as a read-only reference for what external vector stores can be viewed and selected (in future we will likely support the User to Register a vector store from the UI). The ConfigMap is not modified by this flow — only Platform Engineers manage its contents.

The gen-ai-aa-vector-stores ConfigMap schema is at https://github.com/jharan1/odh-spikes/blob/main/spikes/RHOAIENG-51769-external-vector-stores/gen-ai-aa-vector-stores-schema.yaml 

The latest designs are at https://rhoai-promptflow-47b706.pages.redhat.com/gen-ai-studio/asset-endpoints

**Note**: This epic is a late addition to this strat and has not yet been reviewed or approved by PM/Design. Details may change pending that review.

**Outcomes by Persona**

AI Engineer:
- Can view all external vector stores defined in the gen-ai-aa-vector-stores ConfigMap in the AAE page under vector stores tab. The primary column for the table should be Collection, and we should also have columns for Provider, Embedding Model ID/Name, Domain, Status, Dimensions, Distance, metric, Created, Endpoint, Playground.
- For 3.4, a user can click Add to Playground for a vector store collection if the associated embedding model is added as an AI asset endpoint (check the list of AI Asset endpoints, if it's not present we show the row as greyed out, with details on how to add the embedding model as an endpoint). In the modal that opens, show the list of vector stores and associated embedding model next to each store. If an embedding model is not added to AI Asset endpoints we also here show the entire row as greyed out. If it is added but not registered, indicate that the we will auto register it with llamastack. The user may then proceed with installing the vector store and embedding model in the playground by clicking the Configure button.

## Acceptance Criteria

- [ ] External vector stores defined in the gen-ai-aa-vector-stores ConfigMap are visible in the AAE page under a Vector Stores tab table, with columns for Vector store collection, Type, Embedding Model, Created, Dimensions, Distance metric, Playground. The existing BFF VectorStoresAAHandler endpoint in aaa_vectorstores_handler.go can be used to retrieve this list.
- [ ] For 3.4, in order to add Vector store collections to a playground, the user can access the Configure Playground modal (via an Add to Playground link) either from the Models tab or from the Vector Stores tab, under AAE page.
- [ ] Each row in the Vector Stores tab has one of three states based on the associated embedding model: 1) If the embedding model is not added as an AI asset endpoint, the row is shown as greyed out with an explanatory message on how to add it. 2) If the embedding model is added as an AI asset endpoint, the row is active and an "Add to Playground" (or "Try in Playground" if a playground already exists) action is available. 3) If the embedding model is added as an AI asset endpoint and is also registered in the llamastack installed in the playground, a green checkmark is shown in the Embedding Model column next to the model id/name.
- [ ] The Configure Playground modal will contain two pages - the first page of the modal will show the Models that can be selected, and the second page will show the Vector Store collections. In the first page of the modal we must allow the user to select model(s), and for a given model they can select between a Model Type of Inferencing/LLM or Embedding (the selection dropdown shows when they select a row). The user can then select the models they want, and click to open the second page of the modal. When on the first page, there will be buttons at bottom "Next select collections" and "Cancel", and second page the buttons at bottom will include "Back to models", "Configure", and "Cancel".
- [ ] On Page 2 of the modal, we show only vector store collections for which there is an embedding model available (whether just added as an AI asset endpoint, or also registered in llamastack). Any vector stores or models that have been registered previously in the playground are already pre-selected. The user can multiselect/bulk select collections. If the user selects additional vector store collections on Page 2, the associated embedding models are auto-selected on Page 1.
- [ ] The user can proceed with installing the models and vector store collections selected in the Playground by clicking the Configure button on the second page of the modal, which triggers Playground install (note we already have logic whereby if playground already exists, the UI first makes request to delete the existing playground) by making a request using the existing installLSD method call in ChatbotConfigurationModal.tsx, passing the vector store(s) and models in the request (to the LlamaStackDistributionInstallHandler endpoint).
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by any of these flows.
- [ ] For the Update Configuration modal that can be accessed from the dropdown within playground, ensure that when it's opened we pass/preselect any models or vector stores currently in the playground, and at bottom of modal can show Update instead of Configure.
- [ ] For the Create Playground modal that a user can open if they visit the playground page and they haven't yet created a playground in the namespace, this should look the same as the Configure Playground modal accessed from AAE page, only we don't have any preselected models or vector stores.

## Notes

- This epic is pending PM/Design review — implementation details may change.
- **Model ID correlation**: When determining which embedding models have been added as AI Asset endpoints, the UI or API will need to correlate embedding model IDs from the gen-ai-aa-vector-stores ConfigMap against model IDs from the AAE endpoint. These IDs may not include the full provider prefix (e.g. the AAE endpoint may return "qen003b" while the llamastack config map uses "vllm-inference-1/qen003b"). This can cause ambiguity when multiple models with the same base ID exist from different sources (e.g. maas vs namespace models). Adding metadata to models to indicate their source (as discussed with Nick) may provide a cleaner solution for correlating IDs.

## Stories

- Story 1 (RHOAIENG-51780): Display External Vector Stores in the AAE Vector Stores Tab
- Story 2 (RHOAIENG-51781): Add to Playground Flow — Select and Launch a Playground with Vector Stores from AAE
