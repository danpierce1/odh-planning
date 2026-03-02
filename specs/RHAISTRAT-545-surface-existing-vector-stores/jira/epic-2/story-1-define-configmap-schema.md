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
| Epic Link      | RHOAIENG-51472                                                        |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51474                       |

---

## Jira Description

As a Platform Engineer, I want a well-defined and documented ConfigMap schema for registering external vector stores, so that I can create and manage the gen-ai-aa-vector-stores ConfigMap in a GitOps-friendly way.

The ConfigMap uses a stores.yaml key containing a YAML list of vector store definitions. The schema should mirror llamastack config fields as closely as possible, keeping Gen AI Studio-specific fields (e.g. display name, domain, owner) grouped under a custom_gen_ai block — the same pattern Nick used when defining the external models schema (gen-ai-aa-external-models ConfigMap): https://gist.github.com/NickGagan/564d1209acf0452ccd0a3a15e91bed51

Each entry specifies the provider type, connection details, collection/index name, an optional Secret reference for credentials, and optional metadata fields. Reference draft schema: https://gist.github.com/ederign/edf4edf4f3aff8b0092c0799eac72bb8

## Acceptance Criteria

- [ ] The ConfigMap schema fields mirror llamastack config fields as closely as possible, with Gen AI Studio-specific fields (e.g. display name, domain, owner) grouped under a custom_gen_ai block — consistent with the pattern used for the gen-ai-aa-external-models ConfigMap.
- [ ] The ConfigMap schema is defined and documented, covering required fields (type, connection endpoint, collection/index name) and optional fields (description, owner, domain, name, Secret reference for credentials).
- [ ] The schema supports all three MVP providers: PGVector, Qdrant, and Milvus.
- [ ] A validated example ConfigMap is provided in the documentation for each supported provider.
- [ ] The schema is validated by the BFF on load — malformed entries are skipped with a logged warning.

## Notes

- ConfigMap is namespace-scoped; all users in the namespace can see any registered vector store.
- ConfigMap changes require a llamastack distribution restart to take effect — this should be noted in documentation.
- Credential Secrets are referenced by name in the ConfigMap and must exist in the same namespace.
