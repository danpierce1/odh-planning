# Epic: External Vector Stores Registration During BFF LSD Install

## Jira Fields

| Field          | Value                                                                              |
|----------------|------------------------------------------------------------------------------------|
| Issue Type     | Epic                                                                               |
| Summary        | External Vector Stores Registration During BFF LSD Install                         |
| Epic Name      | External Vector Stores Registration During BFF LSD Install                         |
| Priority       | Major                                                                              |
| Project        | RHOAIENG                                                                           |
| Components     | AI Core Dashboard, Gen AI Studio                                                   |
| Labels         | dashboard-crimson-scrum                                                            |
| Activity Type  | New Features                                                                       |
| Target Version | rhoai-3.4                                                                          |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51472                                    |

---

## Jira Description

Platform engineers are responsible for creating the gen-ai-aa-vector-stores ConfigMap. This epic updates the Install phase of LSD in the gen-ai BFF to check for the presence of that ConfigMap and, if found, configure any valid external vector stores into the LSD being created (see InstallLlamaStackDistribution and generateLlamaStackConfig functions).

This is the foundational backend work that makes external vector stores available for selection in the Gen AI Playground.

**Outcomes by Persona**

Platform Engineer:
- Can register existing vector databases (PGVector, Qdrant, Milvus) by creating a ConfigMap in the namespace.
- Can provide metadata (description, owner, domain) so users understand data provenance.
- Knows that ConfigMap updates require a llamastack distribution restart to take effect.

AI Engineer:
- Can view all available vector stores in the namespace from the Knowledge Sources section.
- Can use vector stores without needing to know database credentials or connection details.

## Acceptance Criteria

- [ ] BFF reads the gen-ai-aa-vector-stores ConfigMap during LSD install and includes valid vector stores in the generated llamastack configuration.
- [ ] Vector stores with invalid or unreachable connections or unavailable embedding model cause a failure with error returned to the UI.
- [ ] ConfigMap schema supports all three MVP providers: PGVector, Qdrant, Milvus.
- [ ] Credential references via Kubernetes Secrets are supported.

## Stories

- Story 1: Define the gen-ai-aa-vector-stores ConfigMap Schema
- Story 2: Spike — Investigate Llamastack Responses for Invalid Vector Store Configurations
- Story 3: Register External Vector Stores During BFF LSD Install
