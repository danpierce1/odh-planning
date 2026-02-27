# Story: Document the External Vector Stores Feature Flag

## Jira Fields

| Field          | Value                                                                 |
|----------------|-----------------------------------------------------------------------|
| Issue Type     | Story                                                                 |
| Summary        | Document the External Vector Stores Feature Flag                      |
| Priority       | Normal                                                                |
| Project        | RHOAIENG                                                              |
| Components     | AI Core Dashboard, Gen AI Studio                                      |
| Labels         | dashboard-crimson-scrum                                               |
| Activity Type  | New Features                                                          |
| Target Version | rhoai-3.4                                                             |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                   |

---

## Jira Description

As a Platform Engineer, I can find clear documentation explaining the external vector stores feature flag — what it controls, how to enable or disable it, and what pre-conditions are needed — so that I can make informed decisions about when and how to roll out this feature in my environment.

A new feature flag without documentation risks platform engineers either never discovering the capability or enabling it without understanding the impact. Documentation must ship alongside the flag.

Target audience: Platform Engineers comfortable with Kubernetes/OpenShift but not necessarily familiar with RHOAI internals.

## Acceptance Criteria

- [ ] Ensure documentation team is informed of the new flag and it's purpose (assumpting they will handle actually documenting the field)

## Notes

- Cross-reference with Epic 2 (ConfigMap setup) and Epic 3 (Playground UI) so engineers understand the full enablement journey.
- Confirm the exact field name with the engineering team once Story 1 is complete before finalising the YAML example.
