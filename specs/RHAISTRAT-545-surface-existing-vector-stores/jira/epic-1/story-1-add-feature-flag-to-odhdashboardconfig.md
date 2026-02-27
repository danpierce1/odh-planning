# Story: Add External Vector Stores Feature Flag to OdhDashboardConfig

## Jira Fields

| Field         | Value                                                                 |
|---------------|-----------------------------------------------------------------------|
| Issue Type    | Story                                                                 |
| Summary       | Add External Vector Stores Feature Flag to OdhDashboardConfig         |
| Priority      | Normal                                                                |
| Project       | RHOAIENG                                                              |
| Components    | AI Core Dashboard, Gen AI Studio                                      |
| Labels        | dashboard-crimson-scrum                                               |
| Activity Type | New Features                                                          |
| Epic Link     | [RHOAIENG-XXXXX — set once Epic is created in Jira]                   |

---

## Jira Description

As a Platform Engineer, I want to enable or disable the external vector stores feature via a configuration flag in the OdhDashboardConfig custom resource, so that I can control when this capability is available to users and ensure it is only exposed when the infrastructure is ready.

The external vector stores feature must be gated behind an administrative flag. This allows platform teams to roll out the feature progressively and only after confirming that vector store infrastructure is in place. The flag is added to OdhDashboardConfig, which is the established mechanism for dashboard feature toggles in RHOAI.

This story is configuration/backend only.

## Acceptance Criteria

- [ ] A new feature flag field is added to the OdhDashboardConfig CR spec, following the existing dashboard feature flag pattern.
- [ ] The flag value is exposed via the existing dashboard feature flags API so the frontend can consume it without additional API calls.
- [ ] The CR schema change is backward compatible — existing OdhDashboardConfig resources without the new field behave as if the flag is disabled.

## Notes

- The flag state must be available before any external vector store UI components render.
