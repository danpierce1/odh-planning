# Story: Define the gen-ai-aa-vector-stores ConfigMap Schema

## Jira Fields

| Field          | Value                                                                 |
|----------------|-----------------------------------------------------------------------|
| Issue Type     | Story                                                                 |
| Summary        | Define the gen-ai-aa-vector-stores ConfigMap Schema                   |
| Priority       | Normal                                                                |
| Project        | RHOAIENG                                                              |
| Components     | AI Core Dashboard, Gen AI Studio                                      |
| Labels         | dashboard-crimson-scrum                                               |
| Activity Type  | New Features                                                          |
| Target Version | rhoai-3.4                                                             |
| Epic Link      | [RHOAIENG-XXXXX — set once Epic is created in Jira]                   |

---

## Jira Description

As a Platform Engineer, I want a well-defined and documented ConfigMap schema for registering external vector stores, so that I can create and manage the gen-ai-aa-vector-stores ConfigMap in a GitOps-friendly way.

The ConfigMap uses a stores.yaml key containing a YAML list of vector store definitions. Each entry specifies the provider type, connection details, collection/index name, an optional Secret reference for credentials, and optional metadata fields. Reference schema: https://gist.github.com/ederign/edf4edf4f3aff8b0092c0799eac72bb8

## Acceptance Criteria

- [ ] The ConfigMap schema is defined and documented, covering required fields (type, connection endpoint, collection/index name) and optional fields (description, owner, domain, name, Secret reference for credentials).
- [ ] The schema supports all three MVP providers: PGVector, Qdrant, and Milvus.
- [ ] A validated example ConfigMap is provided in the documentation for each supported provider.
- [ ] The schema is validated by the BFF on load — malformed entries are skipped with a logged warning.

## Notes

- ConfigMap is namespace-scoped; all users in the namespace can see any registered vector store.
- ConfigMap changes require a llamastack distribution restart to take effect — this should be noted in documentation.
- Credential Secrets are referenced by name in the ConfigMap and must exist in the same namespace.
