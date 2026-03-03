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

Findings should be documented and used to inform the implementation of vector store validation (Stories 3–5).

## Acceptance Criteria
- [ ] Does llamastack perform connectivity checks at startup?
- [ ] if multiple vector stores have misconfiguration/issue, does llamastack raise each error in response or one at a time
- [ ] What error format does llamastack return?
- [ ] Test llamastack behaviour when given a vector store whose required embedding model is not available.
- [ ] Test llamastack behaviour when given an unreachable vector store endpoint.
- [ ] Test llamastack behaviour when given a malformed or incomplete vector store configuration.
- [ ] Test llamastack behaviour when invalid credentials for provided vector store.
- [ ] Test if one bad store configuration causes llamastack install to fail
- [ ] document findings and add as a comment on this ticket or linked doc