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
- [ ] User can enable and disable a single vector store per chat pane (they can only have either an inline or external active at one point, both are treated the same)
- [ ] Vector store selection is replicated to new comparison panes by default; each pane can then be managed independently.

## Stories

- Story 1 (RHOAIENG-51775): Enable External Vector Stores in the Playground Knowledge Tab
