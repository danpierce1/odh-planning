# Story: Display External Vector Stores in the AAE Vector Stores Tab

## Jira Fields

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                              |
| Summary        | Display External Vector Stores in the AAE Vector Stores Tab                        |
| Priority       | Normal                                                                             |
| Project        | RHOAIENG                                                                           |
| Components     | AI Core Dashboard, Gen AI Studio                                                   |
| Labels         | dashboard-crimson-scrum                                                            |
| Activity Type  | New Features                                                                       |
| Target Version | rhoai-3.4                                                                          |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                                |

---

## Jira Description

As an AI Engineer, I want to see all registered external vector stores listed in the AAE page under a Vector Stores tab, so that I can review their details and optionally decide to include in a new Playground.

When the feature flag is enabled (Epic 1), a Vector Stores tab is added to the AAE page. It lists all external vector stores from the gen-ai-aa-vector-stores ConfigMap with the following columns: Vector Store (name), Provider, Embedding Model, Domain, and Status.

Reference Figma: https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=1419-6335&t=u9oLDgOyt8i0NEwO-0

## Acceptance Criteria

- [ ] A Vector Stores tab is added to the AAE page, visible only when the feature flag is enabled.
- [ ] The tab lists all external vector stores from the ConfigMap with columns: Vector Store (name), Provider, Embedding Model, Domain, Status.
- [ ] An Add to Playground action is available from this tab (behaviour implemented in Story 2).

## Notes

- This epic is pending PM/Design review — implementation details may change.
- Depends on Epic 1 (feature flag) and Epic 2 (BFF ConfigMap loading) being in place.
