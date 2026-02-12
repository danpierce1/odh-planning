# Feature Specification: UI for Evaluation Orchestration

**Feature Branch**: `010-evaluation-ui`
**Created**: 2026-02-12
**Status**: Draft
**Input**: RHAISTRAT-1134 - UI for Orchestration of Evaluations through Evaluation Stack
**Strategic Link**: https://issues.redhat.com/browse/RHAISTRAT-1134

> **NOTE**: This feature is expected to span various releases. The intent during the 3.2 release is to complete the design. Implementation is targeted for subsequent releases.

## Overview

The Evaluation Stack control plane (RHAISTRAT-26) provides the backend for routing evaluations across evaluation frameworks, benchmarks, and profiling tools. This specification covers the **UI layer** that exposes these capabilities to users, enabling them to execute evaluations, benchmarking, and profiling without requiring AI expert-level knowledge.

### Business Alignment

- **Market First**: Combines multiple evaluation frameworks, allows BYO evaluation frameworks, and integrates benchmarking and profiling tools—all while enabling assessment of AI model capabilities, performance, and scalability
- **Democratized Access**: Exposes "evaluation collections" in a UI so regular users can run cross-framework assessments without understanding underlying algorithms or evaluation tasks

### Key User Questions This Feature Addresses

- "How do I know if a model is good for my industry and use case?"
- "How do I know which model will perform better for my use case?"
- "How do I know if the model is good for agentic AI?"

---

## Epics *(mandatory)*

### Phase 1: Core Evaluation UI (3.4 Release)

#### Epic 1: Evaluation Selection Interface (Priority: P1, Owner: gen-ai)

Provide users with a comprehensive interface to select evaluation types, including evaluation collections, standardized evaluations/benchmarks, and custom evaluation tasks.

**User Value**: Users can easily discover and select the appropriate evaluation approach for their models without needing to understand the underlying frameworks.

**Technical Considerations**:
- UI must integrate with Evaluation Stack control plane APIs to fetch available evaluations
- Selection state must handle multiple evaluation types (collections, standardized, custom) with different configuration requirements
- Lazy loading for large evaluation catalogs to maintain responsiveness

**Outcomes by Persona**:

_AI Engineer_:
- Select from three evaluation types: Evaluation Collection, Standardized Evaluation/Benchmark, or Custom Evaluation Task
- View evaluation tasks available from registered frameworks (LM Evaluation Harness, RAGAS, GuideLLM, Garak, etc.)
- Filter and search evaluation tasks by: framework, category, goal, industry, use-case type, or name
- See clear descriptions of each evaluation task's purpose and expected outcomes

_Data Scientist_:
- Browse evaluation collections curated for specific industries or use cases
- Understand what each evaluation measures without deep framework expertise

> **Dependency**: Evaluation Stack backend (RHAISTRAT-26) must provide API for querying available evaluations, collections, and metadata.

---

#### Epic 2: Evaluation Task Details View (Priority: P1, Owner: gen-ai)

Display comprehensive information about selected evaluation tasks, including metadata, expected behavior, and performance guidance.

**User Value**: Users can make informed decisions about which evaluations to run by understanding what each task measures and what constitutes a "good" result.

**Technical Considerations**:
- Schema viewer component for input/output formats (advanced view)
- Performance range data may be pulled from public leaderboards or provided by the backend
- Links to upstream evaluation task documentation must be externally resolvable

**Outcomes by Persona**:

_AI Engineer_:
- See evaluation task name with link to upstream evaluation task or dataset documentation
- View description of the evaluation type and its intended usage/goal
- Access advanced view showing expected input and output schema
- See guidance on "good range" for evaluation results (derived from public leaderboards when available)

_Data Scientist_:
- Understand what success looks like for a given evaluation before running it
- Compare expected performance ranges across different evaluation tasks

---

#### Epic 3: Evaluation Target Configuration (Priority: P1, Owner: gen-ai)

Enable users to configure evaluation targets (model or AI application) and set evaluation parameters before execution.

**User Value**: Users can precisely configure evaluations to match their testing requirements, including the specific model/application and execution parameters.

**Technical Considerations**:
- Model/application selector must integrate with existing model registry and serving infrastructure
- Parameter validation should occur client-side before submission
- Configuration must be serializable for potential persistence or sharing

**Outcomes by Persona**:

_AI Engineer_:
- Select whether to evaluate a model or an AI application
- Choose the specific model or application endpoint to evaluate
- Configure evaluation parameters including number of samples and maximum tokens per response
- Validate configuration before starting the evaluation run

_ML Ops Engineer_:
- Configure evaluation runs for deployed models in production-like environments
- Set resource constraints appropriate for evaluation workload

> **Dependency**: Model Serving team must expose APIs for listing available models/endpoints suitable for evaluation.

---

#### Epic 4: Evaluation Execution and Progress (Priority: P1, Owner: gen-ai)

Provide real-time visibility into evaluation execution progress with the ability to cancel running evaluations.

**User Value**: Users can monitor evaluation progress, understand estimated completion, and stop evaluations that are taking too long or no longer needed.

**Technical Considerations**:
- WebSocket or polling mechanism for real-time progress updates from the Evaluation Stack
- Cancellation must gracefully terminate backend evaluation jobs
- Progress state must persist across page refreshes during active runs [NEEDS CLARIFICATION: confirm persistence requirement with PM]

**Outcomes by Persona**:

_AI Engineer_:
- See clear progress indication during evaluation execution
- Monitor which evaluation tasks are running, completed, or pending
- Cancel a running evaluation at any point
- Receive notification when evaluation completes or fails

_ML Ops Engineer_:
- Track resource utilization during evaluation runs
- Identify stuck or long-running evaluations for troubleshooting

---

#### Epic 5: Evaluation Results Display (Priority: P1, Owner: gen-ai)

Present evaluation results in a clean, understandable summary that enables informed decision-making.

**User Value**: Users can quickly understand evaluation outcomes and compare them to expected performance ranges or previous runs.

**Technical Considerations**:
- Results schema varies by evaluation framework; UI must handle heterogeneous result formats
- Summary view should highlight key metrics; detailed view should show full results
- Results must be exportable for reporting and comparison workflows

**Outcomes by Persona**:

_AI Engineer_:
- View clean summary of evaluation results upon completion
- See key metrics highlighted with comparison to expected ranges
- Access detailed results for deep-dive analysis
- Export results for offline review or sharing

_Data Scientist_:
- Interpret results without needing framework-specific expertise
- Compare results across different models or evaluation runs

---

### Phase 2: Custom Evaluations (Future Release)

#### Epic 6: Custom Evaluation Dataset Management (Priority: P2, Owner: gen-ai)

Allow users to create or upload custom evaluation datasets for use with existing or custom evaluation tasks.

**User Value**: Users can evaluate models against their own proprietary or domain-specific data, enabling organization-specific quality assessments.

**Outcomes by Persona**:

_AI Engineer_:
- Upload custom evaluation datasets in supported formats
- Generate evaluation datasets using assisted tooling [NEEDS CLARIFICATION: generation method TBD]
- Associate custom datasets with standard evaluation tasks

_Data Scientist_:
- Create evaluation datasets from existing experiment data
- Validate dataset format before use in evaluations

---

#### Epic 7: Custom Evaluation Framework Integration (Priority: P2, Owner: gen-ai)

Enable users to bring their own evaluation frameworks and integrate them with the evaluation orchestration system.

**User Value**: Organizations can leverage their proprietary evaluation methods while benefiting from the unified orchestration and UI experience.

**Outcomes by Persona**:

_AI Engineer_:
- Register custom evaluation frameworks with the Evaluation Stack
- Configure custom framework execution parameters
- Use custom frameworks alongside standard evaluation tasks

_ML Ops Engineer_:
- Manage custom framework deployments and dependencies
- Monitor custom framework execution alongside standard evaluations

> **Dependency**: Evaluation Stack backend must support custom framework registration and execution APIs.

---

### Edge Cases

- What happens when an evaluation task fails mid-execution? The UI should display partial results (if available) and a clear error message indicating the failure point.
- How does the system handle evaluation frameworks becoming unavailable? The UI should gracefully degrade, showing which frameworks are unavailable and allowing selection of alternatives.
- What happens when evaluation results exceed expected display limits? Results should paginate or virtualize to maintain UI responsiveness.
- How are conflicting evaluation parameters handled? Validation should occur before execution with clear error messaging.
- What happens if the user navigates away during an active evaluation? [NEEDS CLARIFICATION: Should evaluations continue in background? Should user be warned?]
- How are concurrent evaluation requests from the same user handled? [NEEDS CLARIFICATION: Allow multiple concurrent evaluations or queue them?]

### Potential Spikes

| Area | Uncertainty | Spike Goal | Recommended Timebox |
|------|-------------|------------|---------------------|
| **Evaluation Stack API Integration** | API contracts and authentication patterns not yet finalized | Validate integration patterns with Evaluation Stack control plane (RHAISTRAT-26) | 2-3 days |
| **Heterogeneous Result Rendering** | Each evaluation framework returns different result schemas | Prototype a flexible result renderer that handles multiple schemas | 2 days |
| **Real-time Progress Updates** | Optimal approach for streaming progress (WebSocket vs polling) unknown | Evaluate WebSocket vs polling trade-offs for evaluation progress | 1-2 days |
| **Public Leaderboard Integration** | Source and format of "good range" performance data unclear | Research available leaderboard APIs and data formats | 1 day |

---

## Performance & Scaling

| Concern | Impact | Consideration |
|---------|--------|---------------|
| **Evaluation Catalog Size** | Large number of evaluation tasks across frameworks | Implement pagination, filtering, and lazy loading for evaluation lists |
| **Long-Running Evaluations** | Some evaluations may run for extended periods | Progress UI must remain responsive; consider background execution patterns |
| **Result Data Volume** | Detailed evaluation results can be large | Implement summary/detail view pattern; paginate detailed results |
| **Concurrent Evaluations** | Users may want to run multiple evaluations simultaneously | [NEEDS CLARIFICATION: Determine concurrency limits and queueing behavior] |
| **Backend Load** | Evaluation execution is resource-intensive | UI should communicate estimated wait times when backend is loaded |

---

## System Constraints

- Users can only access evaluation frameworks and tasks they have permission to use
- Evaluation execution is handled by the Evaluation Stack backend; UI orchestrates but does not execute
- UX designs must be completed and approved before UI implementation begins (3.2 release focus)
- Custom evaluation framework integration is deferred to future releases

---

## Key Entities

- **Evaluation Task**: A specific evaluation operation from a registered framework (e.g., a benchmark from LM Evaluation Harness)
- **Evaluation Collection**: A curated set of evaluation tasks grouped for specific industries, use cases, or assessment goals
- **Evaluation Run**: An instance of executing one or more evaluation tasks against a specific model or application
- **Evaluation Result**: The output data from a completed evaluation run, including metrics and detailed scores
- **Evaluation Framework**: A registered evaluation system (LM Evaluation Harness, RAGAS, GuideLLM, Garak, etc.)
- **Custom Evaluation Dataset**: User-provided data used as input for evaluation tasks

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can execute evaluations from registered frameworks (LM Evaluation Harness, RAGAS, GuideLLM, Garak) through the UI
- **SC-002**: Users can discover and select evaluation tasks using filtering by framework, category, goal, industry, or use-case
- **SC-003**: Users can view evaluation task details including description, schema, and expected performance ranges
- **SC-004**: Users can configure evaluation targets (model or application) and execution parameters before running
- **SC-005**: Users can monitor evaluation progress in real-time and cancel running evaluations
- **SC-006**: Users can view evaluation results in a clean summary format upon completion
- **SC-007**: Users without AI expert knowledge can successfully run evaluations appropriate for their use case

---

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **Evaluation Stack Backend** | API for evaluation discovery, execution, and results | Integration | Primary dependency - UI wraps backend capabilities (RHAISTRAT-26) |
| **Model Serving** | API for listing models/endpoints available for evaluation | Integration | Target selection requires model inventory |
| **UX Team** | Complete UI designs per Figma mockups | Design | Blocking for implementation; design phase in 3.2 release |
| **Model Registry** | Model metadata for display in evaluation target selection | Integration | Model names, versions, capabilities |
| **QE Team** | Test coverage for evaluation workflows | Testing | Complex multi-step flows require comprehensive E2E testing |
| **Platform/Backend** | Authentication and authorization integration | Integration | Users should only see evaluations they can access |
| **Docs Team** | User documentation for evaluation features | Informational | New user-facing capability requires documentation |

### Blocking Dependencies

- **Evaluation Stack Backend (RHAISTRAT-26)**: UI cannot function without backend APIs for evaluation orchestration
- **UX Team**: Implementation blocked until designs are complete (3.2 release deliverable)

### Informational (Capacity/Awareness)

- **Docs Team**: New feature requiring user-facing documentation
- **QE Team**: Complex workflow requiring expanded test coverage

---

## Open Questions for PM Review

Q: Should evaluation progress persist across page refreshes during active runs?
A: _______________

Q: Should users be warned or blocked when navigating away during an active evaluation?
A: _______________

Q: Should users be able to run multiple concurrent evaluations, or should they be queued?
A: _______________

Q: What is the expected method for generating custom evaluation datasets (Phase 2)?
A: _______________

Q: Are there specific frameworks beyond LM Evaluation Harness, RAGAS, GuideLLM, and Garak that must be supported at launch?
A: _______________

Q: What are the performance targets for evaluation discovery (e.g., maximum load time for evaluation catalog)?
A: _______________

---

## Assumptions

- The Evaluation Stack control plane (RHAISTRAT-26) provides all necessary backend APIs for evaluation orchestration
- UX designs referenced in the Jira issue (by Dave McGrath) will be finalized during the 3.2 release
- Users have existing access to models and AI applications through the platform's standard permission system
- The existing platform infrastructure can support the UI components required for this feature
- Evaluation result formats from different frameworks can be normalized sufficiently for a unified display experience
- Public leaderboard data for "good range" performance guidance is available for common evaluation tasks

---

## References

- **Parent Feature**: [RHAISTRAT-26 - Evaluation Stack Control Plane](https://issues.redhat.com/browse/RHAISTRAT-26)
- **UX Prototypes**: By Dave McGrath (referenced in RHAISTRAT-1134)
- **Supported Frameworks**: LM Evaluation Harness, RAGAS, GuideLLM, Garak (initial set)
