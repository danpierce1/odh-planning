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

As an AI Engineer, I want to click Add to Playground from the AAE Vector Stores tab and configure a Playground with the selected vector store(s) and associated embedding model(s), so that I can quickly launch a RAG-enabled Playground without manual configuration.

When a user clicks Add to Playground for a vector store row, a modal opens showing the list of vector store collections and associated embedding model next to each store. If an embedding model is not added to AI Asset endpoints show the entire row as greyed out. If it is added but not registered, indicate that the backend will auto register it with llamastack. The user then clicks Configure to proceed, which triggers the LSD install with the vector store(s) and models passed.

The gen-ai-aa-vector-stores ConfigMap is not modified by this flow — it is used only as a read-only reference.

## Acceptance Criteria

- [ ] Clicking Add to Playground from the AAE Vector Stores tab opens a modal that shows the list of vector store collections and associated embedding model next to each store. If an embedding model is not added to AI Asset endpoints show the entire row as greyed out. If it is added but not registered, indicate that the backend will auto register it with llamastack. The user then clicks Configure to proceed, which triggers the LSD install with the vector store collection(s) and models passed.
- [ ] If the embedding model is not yet available in llamastack, the modal indicates it will be automatically registered.
- [ ] The user can proceed by clicking Configure, which triggers the LSD install, using the existing installLSD method call in ChatbotConfigurationModal.tsx, passing the vector store(s) and models in the request (to the LlamaStackDistributionInstallHandler endpoint).
- [ ] The gen-ai-aa-vector-stores ConfigMap is not modified by this flow.

## Notes

- This story is pending PM/Design review — implementation details may change.
- **Needs clarification**: confirm if external vector store selection should also be included in the Update Configuration modal flow in playground (perhaps a stretch goal)
- Depends on Story 1 (RHOAIENG-51780, AAE Vector Stores tab) being in place.
- Depends on Epic 2 (RHOAIENG-51472, BFF LSD install) for the backend registration of the selected stores.
