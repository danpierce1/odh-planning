# Story: Load External Vector Stores from ConfigMap During BFF LSD Install

## Jira Fields

| Field          | Value                                                                                |
|----------------|--------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                |
| Summary        | Load External Vector Stores from ConfigMap During BFF LSD Install                    |
| Priority       | Normal                                                                               |
| Project        | RHOAIENG                                                                             |
| Components     | AI Core Dashboard, Gen AI Studio                                                     |
| Labels         | dashboard-crimson-scrum                                                              |
| Activity Type  | New Features                                                                         |
| Target Version | rhoai-3.4                                                                            |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                                  |

---

## Jira Description

As an AI Engineer, I want the external vector stores defined in a ConfigMap by my Platform Engineer to be automatically included when a new Playground is created, so that I can select them in the UI.

This story updates the BFF Install phase (InstallLlamaStackDistribution and generateLlamaStackConfig functions) to check for the presence of the gen-ai-aa-vector-stores ConfigMap in the namespace. If the ConfigMap is found, all valid vector store entries are parsed from stores.yaml and included in the llamastack distribution configuration.

## Acceptance Criteria

- [ ] During LSD install, the BFF checks for the gen-ai-aa-vector-stores ConfigMap in the target namespace.
- [ ] If the ConfigMap is present, all parseable vector store entries from stores.yaml are passed to the llamastack configuration.
- [ ] If the ConfigMap is absent, LSD install proceeds as normal with no vector stores — no error is raised.
- [ ] Invalid entries should cause a failure (with useful error information for UI)

## Notes

- Depends on Story 1 (ConfigMap schema) and Story 2 (spike findings) - although use discretion w.r.t timing of work (may be able to do in parallel)
- Validation of connectivity and embedding model availability is handled in Stories 4 and 5 respectively.
