# Epic: Feature Flag for External Vector Stores

## Jira Fields

| Field         | Value                                          |
|---------------|------------------------------------------------|
| Issue Type    | Epic                                           |
| Summary       | Feature Flag for External Vector Stores        |
| Epic Name     | Feature Flag for External Vector Stores        |
| Priority      | Major                                          |
| Project       | RHOAIENG                                       |
| Components    | AI Core Dashboard, Gen AI Studio               |
| Labels        | dashboard-crimson-scrum                        |
| Activity Type | New Features                                   |

---

## Jira Description

Control availability of the external vector store feature through a feature flag in the platform configuration, enabling safe progressive rollout and administrative control over vector store access.

Platform administrators need the ability to control when the external vector store capability is exposed to users. This epic introduces a feature flag in the OdhDashboardConfig CR that gates all external vector store UI components and behaviours. When the flag is enabled, the external vector store experience surfaces in the Gen AI Studio Playground and related pages. When the flag is absent or disabled (the default), the feature is completely hidden from all users.

This follows the established RHOAI dashboard pattern for feature flags and ensures that platform teams can enable the capability only when the necessary infrastructure (vector store ConfigMap, credentials, llamastack distribution) is ready.

**Outcomes by Persona**

Platform Engineer:
- Can enable or disable the external vector store feature by setting a single flag in OdhDashboardConfig.
- Can find documentation explaining the flag, how to enable/disable it, and what pre-conditions should be met before enabling.

AI Engineer:
- Sees external vector store options in the Playground UI only when the platform engineer has enabled the flag (flag logic to be included later as part of UI epics)
- Is not exposed to the feature in environments where it has not been configured.

## Acceptance Criteria

- [ ] A feature flag is added to the OdhDashboardConfig CR to control external vector store visibility.
- [ ] The flag defaults to disabled (hidden) when absent — the feature is opt-in.
- [ ] All external vector store UI components can later be gated behind this flag.
- [ ] Documentation team informed of new flag.

## Stories

- Story 1: Add External Vector Stores Feature Flag to OdhDashboardConfig
- Story 2: Document the External Vector Stores Feature Flag
