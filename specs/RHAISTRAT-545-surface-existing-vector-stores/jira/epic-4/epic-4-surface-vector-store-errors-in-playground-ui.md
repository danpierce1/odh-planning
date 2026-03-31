# Epic: Surface Vector Store Errors in the Playground UI

## Jira Fields

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Issue Type     | Epic                                                                               |
| Summary        | Surface Vector Store Errors in the Playground UI                                   |
| Epic Name      | Surface Vector Store Errors in the Playground UI                                   |
| Priority       | Major                                                                              |
| Project        | RHOAIENG                                                                           |
| Components     | AI Core Dashboard, Gen AI Studio                                                   |
| Labels         | dashboard-crimson-scrum                                                            |
| Activity Type  | New Features                                                                       |
| Target Version | rhoai-3.4                                                                          |
| Parent Link    | RHAISTRAT-545                                                                      |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51777                                    |

---

## Jira Description

Ensure AI engineers see clear, actionable error messages when we encounter failure to configure external vector stores, or other problems with vector stores — whether at Playground creation time due to misconfiguration, or during an active chat session when a vector store becomes unreachable. Errors should be distinguishable from model errors or general query failures so users can identify and report the issue to their Platform Engineer.

**Outcomes by Persona**

AI Engineer:
- Sees a specific error message identifying the vector store issue when Playground creation fails due to a misconfigured store.
- Sees a clear, actionable error during chat when a vector store is unreachable, distinct from model or query errors.

## Acceptance Criteria

- [ ] Playground creation failures caused by vector store misconfiguration are surfaced with a specific, identifiable error message (not a generic failure).
- [ ] Vector store unreachability during an active chat session is surfaced as a distinct, actionable error message.
- [ ] Error messages are visually distinguishable from model errors or general query failures.

## Stories

- Story 1 (RHOAIENG-51778): Surface Vector Store Errors in the Playground UI
