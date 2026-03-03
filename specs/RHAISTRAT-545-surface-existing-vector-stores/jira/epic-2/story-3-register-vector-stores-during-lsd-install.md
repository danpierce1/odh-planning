# Story: Register External Vector Stores During BFF LSD Install

## Jira Fields

| Field          | Value                                                                                |
|----------------|--------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                |
| Summary        | Register External Vector Stores During BFF LSD Install                               |
| Priority       | Normal                                                                               |
| Project        | RHOAIENG                                                                             |
| Components     | AI Core Dashboard, Gen AI Studio                                                     |
| Labels         | dashboard-crimson-scrum                                                              |
| Activity Type  | New Features                                                                         |
| Target Version | rhoai-3.4                                                                            |
| Epic Link      | RHOAIENG-51472                                                                       |

---

## Jira Description

As an AI Engineer, I want the external vector stores defined in a ConfigMap by my Platform Engineer to be automatically included when a new Playground is created, with clear errors surfaced if any stores are misconfigured, unreachable, or missing a required embedding model.

This story updates the BFF Install phase (InstallLlamaStackDistribution and generateLlamaStackConfig functions) to check for the presence of the gen-ai-aa-vector-stores ConfigMap in the namespace. If the ConfigMap is found, all valid vector store entries are parsed from stores.yaml, validated, and included in the llamastack distribution configuration.

Implementation approach should be informed by the spike findings (Story 2).

## Acceptance Criteria

- [ ] During LSD install, the BFF checks for the gen-ai-aa-vector-stores ConfigMap in the target namespace.
- [ ] If the ConfigMap is absent, LSD install proceeds as normal with no vector stores — no error is raised.
- [ ] If a vector store has a Secret reference, the credentials are resolved from the namespace-scoped Secret before use.
- [ ] If a vector store is unreachable or its credentials are invalid, Playground creation fails with a clear, actionable error identifying the affected store.
- [ ] If a store's embedding model is unavailable, Playground creation fails with a clear explanation.

## Notes

- Depends on Story 1 (ConfigMap schema) and Story 2 (spike findings) — although use discretion w.r.t timing of work (may be able to do in parallel).
- Spike findings (Story 2) should inform whether llamastack handles connectivity checks internally or whether the BFF must do this explicitly.
