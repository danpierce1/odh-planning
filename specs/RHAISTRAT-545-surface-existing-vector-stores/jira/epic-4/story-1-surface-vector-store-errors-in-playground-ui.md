# Story: Surface Vector Store Errors in the Playground UI

## Jira Fields

| Field         | Value                                                                                      |
|---------------|--------------------------------------------------------------------------------------------|
| Issue Type    | Story                                                                                      |
| Summary       | Surface Vector Store Errors in the Playground UI                                           |
| Priority      | Normal                                                                                     |
| Project       | RHOAIENG                                                                                   |
| Components    | AI Core Dashboard, Gen AI Studio                                                           |
| Labels        | dashboard-crimson-scrum                                                                    |
| Activity Type  | New Features                                                                              |
| Target Version | rhoai-3.4                                                                                 |
| Epic Link      | RHOAIENG-51777                                                                            |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51778                                           |

---

## Jira Description

As an AI Engineer, I want to see clear and actionable error messages when something goes wrong with external vector stores — whether at Playground creation time or during a chat session — so that I understand what happened and can take action or report the issue to my Platform Engineer.

Two error scenarios are in scope:

1. Playground creation failure: if a vector store misconfiguration causes the LSD install to fail (Epic 2, Story 3), the Playground UI should surface that error clearly rather than showing a generic failure.

2. Unreachable vector store during chat: if a vector store that was successfully registered becomes unreachable during an active chat session, the user should see a clear error identifying the issue as a vector store connectivity problem, distinct from a model or general query error.

## Acceptance Criteria

- [ ] If LSD install fails due to a vector store misconfiguration, the Playground creation error message identifies the vector store issue specifically (not a generic failure).
- [ ] If a vector store is unreachable during an active chat session, the user sees a clear error message indicating the vector store is unreachable.
- [ ] Error messages are actionable — they give the user enough information to relay to a Platform Engineer (e.g. "Vector store 'product-docs' is unreachable. Contact your Platform Engineer to verify the connection.").
- [ ] Vector store errors are visually distinguishable from model errors or general query failures.

## Notes

- Epic 2 should deal with handling and surfacing errors on BFF side, so this story is for ensuring the UI renders errors appropriately.
