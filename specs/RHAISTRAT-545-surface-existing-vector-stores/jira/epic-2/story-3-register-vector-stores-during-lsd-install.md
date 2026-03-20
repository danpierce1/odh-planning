# Story: Register External Vector Stores During BFF LSD Install

## Jira Fields

| Field          | Value                                                                                |
|----------------|--------------------------------------------------------------------------------------|
| Issue Type     | Story                                                                                |
| Summary        | Register External Vector Stores During BFF LSD Install                               |
| Priority       | Normal                                                                               |
| Project        | RHOAIENG                                                                             |
| Components     | AI Core Dashboard, Gen AI Studio                                                     |
| Labels         | dashboard-crimson-scrum                                                              |
| Activity Type  | New Features                                                                         |
| Target Version | rhoai-3.4                                                                            |
| Epic Link      | RHOAIENG-51472                                                                       |
| Jira Issue     | https://issues.redhat.com/browse/RHOAIENG-51773                                      |

---

## Jira Description

As an AI Engineer, I want to be able to select vector stores to be made available in a playground, this will be handled by a LlamaStackDistributionInstallRequest request to the BFF, with clear errors surfaced if any stores are misconfigured. The UI (handled in separate epics) will validate to ensure the vector store ids passed with this request will each have an associated embedding model which is either "available" or "registered", and if available but not registered, our bff code will also need to auto register the embedding model(s).

This story updates the BFF Install endpoint LlamaStackDistributionInstallHandler to allow for LlamaStackDistributionInstallRequest to include vector stores, which if supplied will allow the install logic to configure the vector stores as part of the playground install (and if needed, to register any associated embedding models which are not already registered). If the vector stores are included in request, the Install endpoint logic can retrieve the gen-ai-aa-vector-stores ConfigMap, and use it to include the supplied vector store entries from the ConfigMap into the llamastack distribution configuration (any issue with the config should fail the install).

Implementation approach should be informed by the spike findings (Story 2, RHOAIENG-51769).

## Acceptance Criteria

- [ ] During LSD install, the BFF checks if the install request contains vector stores (LlamaStackDistributionInstallRequest), and if present reads the gen-ai-aa-vector-stores ConfigMap and includes valid matching vector stores in the generated llamastack configuration.
- [ ] allow any valid fields in the configmap to be mapped into the provided/registered entries in the llamastack config (docs include reference to some fields at pgvector, qdrant, milvus).
- [ ] LlamaStackDistributionInstallRequest to be updated to allow for vector stores to be passed
- [ ] If the vector stores are not supplied, the LSD install will not look for configmap, and proceeds as normal with no vector stores — no error is raised.
- [ ] If a vector store has a Secret reference, the credentials are resolved from the namespace-scoped Secret before use, if valid they are included in LSD install via env vars, these would look something like: token: ${env.MILVUS_TOKEN}, api_key: ${env.QDRANT_API_KEY:=}, password: ${env.PGVECTOR_PASSWORD}
- [ ] as identified in our spike (see https://github.com/jharan1/odh-spikes/tree/main/spikes/RHOAIENG-51769-external-vector-stores), we will carry out validation on data supplied in the configmap based on what fields are expected. We can return errors for some invalid data scenarios, basic credentials validation, embedding model unavailable, but for some errors we may be reliant on surfacing a more generic error that llamastack install did not succeed, and perhaps giving a sample of the error log if available back to the user (the spike found that llamastack typically doesn't give structured errors, and errors logged vary across providers used). Give clear, actionable errors where possible.

## Notes

- Depends on Story 1 (RHOAIENG-51474, ConfigMap schema) and Story 2 (RHOAIENG-51769, spike findings) — although use discretion w.r.t timing of work (may be able to do in parallel).
- Spike findings (Story 2, RHOAIENG-51769) should help inform how we handle various issues.
