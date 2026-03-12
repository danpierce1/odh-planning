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
| Epic Link      | RHOAIENG-51779                                                                                 |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51781                                                |

---

## Jira Description

As an AI Engineer, I want to open the Configure Playground modal from either the Models tab or the Vector Stores tab in the AAE page, so that I can select models and vector store collections and launch a RAG-enabled Playground without manual configuration.

The Configure Playground modal is a two-page wizard. Page 1 shows the available models — the user selects one or more, and for each selected model they can set the Model Type (Inferencing/LLM or Embedding) via a dropdown that appears on row selection. Clicking "Next select collections" advances to Page 2, which shows the vector store collections available based on the embedding model(s) selected on Page 1. The user can multiselect/bulk select collections; previously registered vector stores and models are pre-selected. Rows where the associated embedding model is not in AI Asset endpoints are shown as greyed out; if the embedding model is added but not yet registered with llamastack, the UI indicates it will be auto-registered. Clicking "Configure" on Page 2 triggers the LSD install.

The gen-ai-aa-vector-stores ConfigMap is not modified by this flow — it is used only as a read-only reference.

## Acceptance Criteria

- [ ] The Configure Playground modal can be accessed via an Add to Playground link from either the Models tab or the Vector Stores tab in the AAE page.
- [ ] Page 1 of the modal shows the list of available models. When the user selects a model row, a Model Type dropdown appears allowing them to choose Inferencing/LLM or Embedding. The user can select one or more models. Buttons at the bottom of Page 1 are "Next select collections" and "Cancel".
- [ ] Page 2 of the modal shows the vector store collections available based on the embedding model(s) selected on Page 1. Buttons at the bottom of Page 2 are "Back to models", "Configure", and "Cancel".
- [ ] On Page 2, rows where the associated embedding model is not added to AI Asset endpoints are shown as greyed out. If an embedding model is added but not yet registered with llamastack, the row indicates it will be auto-registered by the backend.
- [ ] Any vector stores or models that have been registered previously in the playground are already pre-selected. The user can multiselect/bulk select collections.
- [ ] Clicking Configure on Page 2 triggers the LSD install using the existing installLSD method call in ChatbotConfigurationModal.tsx, passing the selected vector store(s) and models in the request (to the LlamaStackDistributionInstallHandler endpoint). If a playground already exists, the UI first makes a request to delete it before installing.
- [ ] The Update Configuration modal, accessible from the dropdown within the playground, follows the same two-page flow but pre-selects any models and vector stores currently configured in the playground. The action button on Page 2 shows "Update" instead of "Configure".
- [ ] The Create Playground modal, shown when a user visits the playground page and no playground exists in the namespace, follows the same two-page flow as the Configure Playground modal but with no pre-selected models or vector stores.
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by this flow.

## Notes

- This story is pending PM/Design review — implementation details may change.
- **Needs clarification**: confirm if external vector store selection should also be included in the Update Configuration modal flow in playground (perhaps a stretch goal)
- Depends on Story 1 (RHOAIENG-51780, AAE Vector Stores tab) being in place.
- Depends on Epic 2 (RHOAIENG-51472, BFF LSD install) for the backend registration of the selected stores.
