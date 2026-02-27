# Story: Validate Vector Store Connectivity and Credentials During LSD Install

## Jira Fields

| Field          | Value                                                                                    |
|----------------|------------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                    |
| Summary        | Validate Vector Store Connectivity and Credentials During LSD Install                    |
| Priority       | Normal                                                                                   |
| Project        | RHOAIENG                                                                                 |
| Components     | AI Core Dashboard, Gen AI Studio                                                         |
| Labels         | dashboard-crimson-scrum                                                                  |
| Activity Type  | New Features                                                                             |
| Target Version | rhoai-3.4                                                                                |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                                      |

---

## Jira Description

As an AI Engineer, I want to see a clear error message when a vector store cannot be reached during Playground creation, so that I can report the issue to my Platform Engineer and understand why the Playground failed to start.

During LSD install, the BFF validates that each vector store defined in the ConfigMap is reachable. If a vector store has a Secret reference for credentials, those credentials are resolved before attempting the connection check. If a store is unreachable or misconfigured, a failure error is returned to the UI.

Implementation approach should be informed by the spike findings (Story 2).

## Acceptance Criteria

- [ ] During LSD install, the BFF checks that each vector store endpoint is reachable.
- [ ] If a vector store has a Secret reference, the credentials are resolved from the namespace-scoped Secret before the connectivity check.
- [ ] If a vector store is unreachable or its credentials are invalid, the install returns a failure error to the UI with a clear, actionable message identifying the affected store.
- [ ] Reachable stores are unaffected by the failure of other stores (where possible — subject to spike findings).

## Notes

- Depends on Story 2 (spike) findings to determine whether llamastack handles connectivity checks internally or whether the BFF must do this explicitly.
- Validation happens at Playground load time (LSD install) for this release.
