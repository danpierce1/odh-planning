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
| Epic Link      | RHOAIENG-51472                                                                           |

---

## Jira Description

As a developer implementing vector store validation in the BFF, I need to understand how llamastack behaves and what errors it returns when given invalid or misconfigured vector store definitions, so that validation logic in Story 3 is grounded in actual llamastack behaviour rather than assumptions.

Findings should be documented and used to inform the implementation of vector store validation (Story 3).

## Acceptance Criteria
- [ ] Test if llamastack perform connectivity checks at startup?
- [ ] Test if multiple vector stores have misconfiguration/issue, does llamastack raise each error in response or one at a time
- [ ] Test what error format does llamastack return?
- [ ] Test llamastack behaviour when given a vector store whose required embedding model is not available.
- [ ] Test llamastack behaviour when given an unreachable vector store endpoint.
- [ ] Test llamastack behaviour when given a malformed or incomplete vector store configuration.
- [ ] Test llamastack behaviour when invalid credentials for provided vector store.
- [ ] Test if one bad store configuration causes llamastack install to fail
- [ ] document findings and add as a comment on this ticket or linked doc