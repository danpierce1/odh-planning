# Story: Enable External Vector Stores in the Playground Knowledge Tab

## Jira Fields

| Field          | Value                                                                                          |
|----------------|------------------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                          |
| Summary        | Enable External Vector Stores in the Playground Knowledge Tab                                  |
| Priority       | Normal                                                                                         |
| Project        | RHOAIENG                                                                                       |
| Components     | AI Core Dashboard, Gen AI Studio                                                               |
| Labels         | dashboard-crimson-scrum                                                                        |
| Activity Type  | New Features                                                                                   |
| Target Version | rhoai-3.4                                                                                      |
| Epic Link      | RHOAIENG-51774                                                                                 |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51775                                                |

---

## Jira Description

As an AI Engineer, I want to see all external vector stores available in my namespace listed in the Playground Knowledge tab, be able to enable or disable one for my chat session, and have my selection carried over when I open additional comparison panes, so that I can easily experiment with RAG and compare model responses across panes using the same knowledge source.

When the feature flag is enabled (Epic 1, RHOAIENG-51445) and vector stores have been registered via the gen-ai-aa-vector-stores ConfigMap (Epic 2, RHOAIENG-51472), the Knowledge tab surfaces those stores with their name, provider type, and description. Only one vector store can be active at a time per chat pane (external or inline — both are treated the same).

When a new comparison pane is opened, the current vector store selection is replicated to it by default. After that, each pane is independent — the user can enable or disable the vector store on any pane without affecting others.

## Acceptance Criteria

- [ ] Available/registered external vector stores are listed in the Playground Knowledge tab only when the feature flag is enabled.
- [ ] Each store entry displays its name, provider type (PGVector, Qdrant, Milvus), and description where provided.
- [ ] For MVP, the user can select one collection at a time via radio select (not multi-select checkboxes - that may be the flow post 3.4)
- [ ] User can enable an external vector store with a single click; only one vector store can be active at a time per chat pane (external or inline).
- [ ] User can disable the active vector store mid-session, reverting to non-RAG behaviour.
- [ ] The enabled vector store is stored in the chat session state and used for retrieval on subsequent messages.
- [ ] When a new comparison pane is opened, the current vector store selection is replicated to that pane by default.
- [ ] After a pane is opened, its vector store selection is independent — the user can enable or disable on one pane without affecting others.
- [ ] Retrieval implementation details (chunk size, embedding model name, similarity thresholds) are not shown to the user.
- [ ] For 3.4, we can just show the vector stores which are registered/available in LSD in the dropdown (and not show any greyed out stores in this dropdown)

## Notes

- Depends on Epic 1 (RHOAIENG-51445, feature flag) and Epic 2 (RHOAIENG-51472, BFF ConfigMap loading) being in place.
- Read-only operations only — no write access to vector store data.
- Reference designs for visual treatment of the Knowledge tab and store list.
