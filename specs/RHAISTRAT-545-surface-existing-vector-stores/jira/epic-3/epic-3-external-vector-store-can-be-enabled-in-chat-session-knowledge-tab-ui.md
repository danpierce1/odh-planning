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

---

## Jira Description

Enable AI engineers in the GenAI Playground to view and enable an external vector store for a chat session from the Knowledge tab. Engineers can rapidly prototype and evaluate RAG applications using enterprise-approved knowledge sources without writing code, managing connections, or understanding retrieval implementation details.

External vector stores are only visible when the feature flag is enabled (Epic 1) and have been registered via the ConfigMap (Epic 2).

**Outcomes by Persona**

AI Engineer:
- Can see all available external vector stores in the Knowledge tab, including those that are unavailable due to a missing embedding model (shown greyed out).
- Can enable a vector store for a chat session with a single click and disable it mid-session.
- Can compare model responses with and without a vector store by toggling enablement.
- When a new comparison pane is opened, the current vector store selection is replicated to it by default; each pane can then be managed independently.

## Acceptance Criteria

- [ ] External vector stores are listed in the Playground Knowledge tab when the feature flag is enabled.
- [ ] Stores without an available embedding model are visible but greyed out with an explanatory message.
- [ ] User can enable and disable a single vector store per chat pane.
- [ ] Vector store selection is replicated to new comparison panes by default; each pane can then be managed independently.

## Stories

- Story 1: Enable External Vector Stores in the Playground Knowledge Tab
