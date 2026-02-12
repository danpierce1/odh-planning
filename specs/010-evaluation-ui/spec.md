# Feature Specification: UI for Orchestration of Evaluations through Evaluation Stack

**Feature Branch**: `010-evaluation-ui`
**Created**: 2026-02-12
**Status**: Draft
**Strategic Input**: [RHAISTRAT-1134](https://issues.redhat.com/browse/RHAISTRAT-1134)
**Target Release**: 3.4 (Tech Preview)
**Note**: This feature is expected to span various releases. The intent during the 3.2 release is to complete the design.

---

## Overview

The Evaluation Stack control plane (RHAISTRAT-26/RHAISTRAT-979) provides the backend for routing evaluations across evaluation frameworks, benchmarks, and profiling tools. This feature provides the **user interface** layer that enables general AI developers and users to benefit from evaluation capabilities without requiring AI expert-level knowledge.

### Business Alignment

1. **Market First**: Combining multiple evaluation frameworks, allowing BYO evaluation frameworks, and combining benchmarking and profiling tools—all while enabling assessment of AI model capabilities, performance, and scalability.
2. **Accessibility**: By exposing "evaluation collections" in a UI, regular users can run cross-framework assessments of industry- and use-case-specific advanced capabilities without needing to understand the algorithms or specific evaluation tasks behind the scenes.

### Key User Questions Addressed

- "How do I know if a model is good for my industry and my use case?"
- "How do I know which model will perform better for my use case?"
- "How do I know if the model is good for agentic AI?"

---

## Target Personas

| Persona | Role in Feature |
|---------|-----------------|
| **AI Engineer** | Primary user - selects evaluations, configures parameters, analyzes results to determine model suitability for applications |
| **Data Scientist** | Runs evaluations during model development to compare model capabilities |
| **ML Ops Engineer** | Uses evaluation results to inform deployment decisions and monitor model performance |

---

## Epics & User Stories

### Phase 1: Core Evaluation Experience (Design Sprint)

> **Note**: Per RHAISTRAT-1134, Phase 1 focuses on completing the design. Implementation timing TBD.

#### Epic 1: Evaluation Selection Interface (Priority: P1, Owner: gen-ai)

Users need to discover and select appropriate evaluations from the available options registered in the Evaluation Stack.

**User Value**: Users can browse, filter, and select from evaluation collections, standardized evaluations/benchmarks, and custom evaluation tasks without needing to understand the underlying frameworks.

**Technical Considerations**:
- Must integrate with Evaluation Stack control plane APIs (RHAISTRAT-979)
- Selection state must persist through the configuration flow
- Filter state should be URL-addressable for shareability

**Outcomes by Persona**:

_AI Engineer_:
- Browse all available evaluation tasks with filtering by: framework, category, goal, industry, use-case type, and name
- Select an "Evaluation Collection" (curated set of evaluations for specific scenarios)
- Select individual standardized evaluations or benchmarks
- Access custom evaluation task creation flow

_Data Scientist_:
- View evaluation task details including upstream task/dataset links
- Understand the goal and expected usage of each evaluation task
- Access advanced view showing expected input/output schema

> **Dependency**: Evaluation Stack (RHAISTRAT-979) must expose APIs for listing available evaluations, frameworks, and evaluation collections.

---

#### Epic 2: Evaluation Configuration (Priority: P1, Owner: gen-ai)

Once evaluations are selected, users need to configure the evaluation run with appropriate parameters.

**User Value**: Users can configure evaluation runs for either models or AI applications with clear parameter guidance.

**Technical Considerations**:
- Configuration must validate against evaluation task requirements
- Parameter defaults should be provided where sensible
- Configuration should be saveable/reusable [NEEDS CLARIFICATION: Is configuration persistence in scope?]

**Outcomes by Persona**:

_AI Engineer_:
- Select evaluation target: model or AI application
- Configure evaluation parameters including number of samples and maximum tokens per response
- View "good range" guidance for evaluation metrics derived from public leaderboards
- Access links to upstream evaluation task documentation

_Data Scientist_:
- Configure advanced evaluation parameters based on task schema
- Understand parameter impact on evaluation results

> **Dependency**: Evaluation Stack must provide parameter schemas and validation rules for each evaluation type.

---

#### Epic 3: Evaluation Execution & Progress (Priority: P1, Owner: gen-ai)

Users need visibility into evaluation progress and the ability to manage running evaluations.

**User Value**: Users can monitor evaluation progress in real-time and control evaluation execution.

**Technical Considerations**:
- Progress updates should be real-time or near-real-time
- Cancellation must cleanly terminate backend resources
- Multiple concurrent evaluations may be running [NEEDS CLARIFICATION: Concurrency limits?]

**Outcomes by Persona**:

_AI Engineer_:
- View clear progress indication for running evaluations
- Cancel an in-progress evaluation run
- Receive notification when evaluation completes

_ML Ops Engineer_:
- Monitor evaluation resource consumption [NEEDS CLARIFICATION: What metrics are available from Evaluation Stack?]

> **Dependency**: Evaluation Stack must provide progress streaming/polling APIs and cancellation endpoints.

---

#### Epic 4: Results Display & Analysis (Priority: P1, Owner: gen-ai)

Evaluation results must be presented in a clear, actionable format.

**User Value**: Users can understand evaluation outcomes and make informed decisions about model suitability.

**Technical Considerations**:
- Results visualization must accommodate different evaluation metric types
- Comparison views may be needed for multi-model evaluations [NEEDS CLARIFICATION: Is comparison in scope for MVP?]
- Results should be exportable [NEEDS CLARIFICATION: Export formats?]

**Outcomes by Persona**:

_AI Engineer_:
- View clean summary of evaluation results
- Understand whether results fall within "good range" for each metric
- Drill into detailed results per evaluation task

_Data Scientist_:
- Access raw evaluation data for further analysis
- Compare results across evaluation runs [NEEDS CLARIFICATION: Scope TBD]

_ML Ops Engineer_:
- View evaluation history for a model/application
- Export results for reporting [NEEDS CLARIFICATION: Export requirements]

---

### Phase 2: Custom Evaluations (Priority: P2, Owner: gen-ai)

> **Note**: Phase 2 extends the core experience with custom evaluation capabilities.

#### Epic 5: Custom Evaluation Task Creation (Priority: P2, Owner: gen-ai)

Advanced users need to create custom evaluation tasks beyond the standardized offerings.

**User Value**: Users can evaluate models using custom datasets and evaluation approaches specific to their use case.

**Technical Considerations**:
- Custom datasets require upload/storage mechanism
- Custom evaluation frameworks require registration with Evaluation Stack
- Validation of custom configurations is critical

**Outcomes by Persona**:

_AI Engineer_:
- Create custom evaluation task by providing a custom evaluation dataset
- Generate evaluation datasets [NEEDS CLARIFICATION: What generation capabilities exist?]
- Select an existing evaluation task from a framework to use with custom dataset
- Register/provide a custom evaluation framework [NEEDS CLARIFICATION: How does BYO framework work?]

_Data Scientist_:
- Define custom evaluation metrics and scoring
- Test custom evaluation tasks before production use

> **Dependency**: Evaluation Stack must support custom dataset registration and custom framework integration.

---

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **Evaluation Stack Backend** (RHAISTRAT-979) | APIs for evaluation listing, execution, progress, results, cancellation | Integration | Blocking - no UI possible without backend APIs |
| **Evaluation Collections** (RHAISTRAT-1147/RHAIRFE-238) | Industry/use-case evaluation collection definitions | Integration | MVP requires at least some collections available |
| **UX** (RHOAIUX-1257, RHOAIUX-1575, RHOAIUX-1576) | Design specifications and prototypes | Design | Designs provided by D. McGrath per Jira |
| **model-serving** | Model inference endpoint integration for evaluation targets | Integration | Need to evaluate deployed models |
| **platform** | Authentication/authorization for evaluation operations | Integration | RBAC for evaluation execution |
| **QE** | Test coverage for evaluation workflows | Testing | E2E tests for critical paths |

---

## Edge Cases

- **Evaluation Stack unavailable**: Display appropriate error state; disable evaluation features
- **Evaluation task fails mid-execution**: Show partial results if available; clear error messaging
- **Large result sets**: Pagination/virtualization for evaluation results with many data points
- **Concurrent evaluation limits reached**: Queue or reject with clear messaging [NEEDS CLARIFICATION: Limits TBD]
- **Invalid evaluation configuration**: Frontend validation before submission; clear error messages
- **Network interruption during progress polling**: Graceful reconnection; last known state display
- **Evaluation takes longer than expected**: Progress indication should not suggest completion; timeout handling [NEEDS CLARIFICATION: Timeout policy?]

---

## Potential Spikes

| Area | Uncertainty | Spike Goal | Recommended Timebox |
|------|-------------|------------|---------------------|
| Evaluation Stack API Contract | APIs may not yet be finalized | Verify available endpoints, request/response formats, and streaming capabilities | 2 days |
| Real-time Progress Updates | Unknown if Evaluation Stack supports streaming or requires polling | Determine optimal progress update mechanism (SSE, WebSocket, polling) | 1 day |
| Result Visualization | Evaluation metrics vary widely in format and meaning | Prototype visualization components for different metric types | 2 days |
| Custom Dataset Upload | File upload/storage patterns in odh-dashboard not established for this use case | Research existing patterns and propose approach | 1 day |

---

## Performance & Scaling

- **API Call Volume**: Each evaluation run may generate multiple progress polling requests; consider appropriate polling intervals
- **Result Data Size**: Evaluation results may be large; lazy loading and pagination recommended
- **Concurrent Users**: Multiple users may run evaluations simultaneously; ensure UI remains responsive regardless of backend load
- **Caching**: Evaluation task metadata (frameworks, collections, schemas) should be cacheable; results are user-specific

---

## Success Criteria

### Measurable Outcomes

- **SC-001**: Users can discover and select evaluations without prior knowledge of evaluation frameworks
- **SC-002**: Users can execute evaluations against models or AI applications registered in the platform
- **SC-003**: Users receive actionable results that inform model selection decisions
- **SC-004**: Evaluation execution completes reliably with clear progress indication
- **SC-005**: Users can cancel running evaluations cleanly

---

## Open Questions for PM Review

The following questions were deferred during specification creation and require PM clarification:

```
## Open Questions for PM Review

Q: Is evaluation configuration persistence (save/reuse configurations) in scope for MVP?
A: _______________

Q: What are the concurrency limits for evaluation runs per user/namespace?
A: _______________

Q: What resource consumption metrics are available from Evaluation Stack for display?
A: _______________

Q: Is model/result comparison view in scope for MVP or Phase 2?
A: _______________

Q: What export formats are required for evaluation results (CSV, JSON, PDF)?
A: _______________

Q: What dataset generation capabilities exist in the Evaluation Stack?
A: _______________

Q: How does "Bring Your Own Framework" registration work technically?
A: _______________

Q: What is the timeout policy for long-running evaluations?
A: _______________

Q: Should the UI be built within the existing gen-ai package or as a new module?
A: _______________

Q: Are there specific PatternFly patterns required per UX designs?
A: _______________
```

---

## Appendix: UX Reference

UX prototypes have been provided by D. McGrath and are attached to RHAISTRAT-1134. Key screens include:

1. **Evaluation Selection**: Main entry point showing Evaluation Collection, Standardized Evaluations, Custom Evaluation options
2. **Evaluation Task Browser**: Filterable list of available evaluation tasks with framework/category/goal filters
3. **Task Detail View**: Shows task name, description, goal, input/output schema link, "good range" guidance
4. **Configuration Screen**: Model vs. Application selection, parameter inputs (samples, max tokens)
5. **Progress View**: Running evaluation with progress indicator and cancel option
6. **Results Summary**: Clean visualization of evaluation outcomes

---

## Related Issues

- **Depends on**: RHAISTRAT-979 (Core Evaluation Stack), RHAISTRAT-1147 (Evaluation Collections), RHAIRFE-239 (Framework Integration - Closed)
- **Related UX**: RHOAIUX-1575 (Industry Evaluation Suites), RHOAIUX-1576 (Custom Evaluation Task), RHOAIUX-1257 (Benchmark aaS)
- **Clones**: RHAIRFE-926
