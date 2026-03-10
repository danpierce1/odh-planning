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

Platform engineers are responsible for creating the gen-ai-aa-vector-stores ConfigMap. This epic updates the Install phase of LSD in the gen-ai BFF such that if vector stores are supplied in the LlamaStackDistributionInstallRequest (LlamaStackDistributionInstallHandler endpoint), then the bff will read the vector store details from the gen-ai-aa-vector-stores ConfigMap, and configure the vector stores into the LSD being installed (see InstallLlamaStackDistribution and generateLlamaStackConfig functions).

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

- [ ] When the user clicks Add to Playground and proceeds with selection of vector stores in the Vector stores Configuration modal in UI, the UI makes a request including the vector stores via LlamaStackDistributionInstallRequest request, the BFF must then check for vector stores in the LlamaStackDistributionInstallRequest request and for each vector store, it must validate vector store details, check if the associated embedding model is registered in llamastack config (if not it must auto register the embedding model(s) in the llamastack config), and register the vector store(s) in the llamastack config.
- [ ] allow any valid fields in the configmap to be mapped into the provided/registered entries in the llamastack config (docs include reference to some fields at pgvector, qdrant, milvus)
- [ ] carry out validation and surface errors accordingly (as informed by spike carried out in https://github.com/jharan1/odh-spikes/tree/main/spikes/RHOAIENG-51769-external-vector-stores)
- [ ] ConfigMap schema supports all three MVP providers: PGVector, Qdrant, Milvus.
- [ ] Credential references via Kubernetes Secrets are supported.

## Stories

- Story 1 (RHOAIENG-51474): Define the gen-ai-aa-vector-stores ConfigMap Schema
- Story 2 (RHOAIENG-51769): Spike — Investigate Llamastack Responses for Invalid Vector Store Configurations
- Story 3 (RHOAIENG-51773): Register External Vector Stores During BFF LSD Install
