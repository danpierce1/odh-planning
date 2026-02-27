# Story: Spike — Investigate Llamastack Responses for Invalid Vector Store Configurations

## Jira Fields

| Field          | Value                                                                                    |
|----------------|------------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                    |
| Summary        | Spike — Investigate Llamastack Responses for Invalid Vector Store Configurations         |
| Priority       | Normal                                                                                   |
| Project        | RHOAIENG                                                                                 |
| Components     | AI Core Dashboard, Gen AI Studio                                                         |
| Labels         | dashboard-crimson-scrum                                                                  |
| Activity Type  | New Features                                                                             |
| Target Version | rhoai-3.4                                                                                |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                                      |

---

## Jira Description

As a developer implementing vector store validation in the BFF, I need to understand how llamastack behaves and what errors it returns when given invalid or misconfigured vector store definitions, so that validation logic in Stories 3, 4, and 5 is grounded in actual llamastack behaviour rather than assumptions.

This is a time-boxed spike. Findings should be documented and used to inform the implementation of vector store validation (Stories 3–5).

## Acceptance Criteria

- [ ] Tested and documented llamastack behaviour when given an unreachable vector store endpoint.
- [ ] Tested and documented llamastack behaviour when given a vector store whose required embedding model is not available.
- [ ] Tested and documented llamastack behaviour when given a malformed or incomplete vector store configuration.
- [ ] Tested and documented llamastack behaviour when invalid credentials are supplied in Platform Engineer defined ConfigMap for an external vector store.
- [ ] Findings written up and shared with the team (e.g. as a comment on this ticket or linked doc) before Stories 3–5 begin implementation.

## Notes

- Key questions to answer: Does llamastack perform connectivity checks at startup? What error format does it return? Are errors per-store or does one bad store fail the entire install?
- This spike should be completed before Stories 3, 4, and 5 are implemented.
