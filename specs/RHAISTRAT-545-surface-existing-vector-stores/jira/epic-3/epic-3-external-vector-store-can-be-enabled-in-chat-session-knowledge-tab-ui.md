# Epic: External Vector Store Can Be Enabled in Chat Session Knowledge Tab UI

## Jira Fields

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Issue Type     | Epic                                                                               |
| Summary        | External Vector Store Can Be Enabled in Chat Session Knowledge Tab UI              |
| Epic Name      | External Vector Store Can Be Enabled in Chat Session Knowledge Tab UI              |
| Priority       | Major                                                                              |
| Project        | RHOAIENG                                                                           |
| Components     | AI Core Dashboard, Gen AI Studio                                                   |
| Labels         | dashboard-crimson-scrum                                                            |
| Activity Type  | New Features                                                                       |
| Target Version | rhoai-3.4                                                                          |
| Parent Link    | RHAISTRAT-545                                                                      |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51774                                    |

---

## Jira Description

Enable AI engineers in the GenAI Playground to view and enable an external vector store for a chat session from the Knowledge tab. Engineers can rapidly prototype and evaluate RAG applications using enterprise-approved knowledge sources without writing code, managing connections, or understanding retrieval implementation details.

External vector stores are only visible when the feature flag is enabled (Epic 1, RHOAIENG-51445) and have been registered via the ConfigMap (Epic 2, RHOAIENG-51472) and provided/installed in LSD.

**Outcomes by Persona**

AI Engineer:
- Can see all available/installed external vector stores in the Knowledge tab
- Can enable a vector store for a chat session with a single click and disable it mid-session.
- Can compare model responses with and without a vector store by toggling enablement.
- When a new comparison pane is opened, the current vector store selection is replicated to it by default; each pane can then be managed independently.

## Acceptance Criteria

- [ ] Available/registered external vector stores are listed in the Playground Knowledge tab when the feature flag is enabled.
- [ ] For MVP, the user can select one collection at a time via a radio select based table (not multi-select checkboxes - that may be the flow post 3.4), and the existing inline vector store option is listed as one row in this table.
- [ ] User can enable and disable a single vector store per chat pane (they can only have either an inline or external active at one point, both are treated the same)
- [ ] Vector store selection is replicated to new comparison panes by default; each pane can then be managed independently.
- [ ] Update the View Code output to include the selected collection/vector store details, the embedding model used, relevant configuration for reproducing the query outside the playground (A spike may be needed to determine the exact format of the view code output for vector store + embedding model details.)

## Stories

- Story 1 (RHOAIENG-51775): Enable External Vector Stores in the Playground Knowledge Tab
- Story 2 (RHOAIENG-52021): Update View Code Output to Include Vector Store and Embedding Model Details
