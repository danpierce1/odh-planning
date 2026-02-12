# Feature Specification: UI for Orchestration of Evaluations through Evaluation Stack

**Feature Branch**: `005-ui-orchestration-evaluations`
**Created**: 2026-02-12
**Status**: Draft
**Strategic Input**: [RHAISTRAT-1134](https://issues.redhat.com/browse/RHAISTRAT-1134)
**Release Target**: 3.2 (Design), 3.4+ (Implementation)

## Overview

The Evaluation Stack Control Plane (RHAISTRAT-26) provides backend routing across evaluation frameworks, benchmarks, and profiling tools. This UI democratizes access to these capabilities, enabling any RHOAI user to assess AI model capabilities, performance, and scalability without requiring AI expert-level knowledge.

**Market Differentiation**: First-of-its-kind combining multiple evaluation frameworks with BYO capability, cross-framework assessments, and integrated benchmarking/profiling tools.

**Target Users**: All RHOAI personas (Data Scientists, AI Engineers, ML Ops Engineers, Platform Engineers), regardless of AI expertise level.

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **ai-trusty** | Evaluation Stack Control Plane API contracts (RHAISTRAT-26) | Integration | BLOCKING: UI depends on backend evaluation orchestration API |
| **ai-trusty** | Available evaluation frameworks list (LM Eval Harness, RAGAS, GuideLLM, Garak, etc.) | Integration | Required for framework selection UI |
| **ai-trusty** | Evaluation task metadata schema (name, description, input/output schema, public leaderboard ranges) | Integration | Required for task detail display |
| **model-serving-metrics** | Model inference API for evaluation execution | Integration | Evaluations need to invoke models for scoring |
| **model-registry-catalog** | Model catalog API for model selection | Integration | Users select models to evaluate from catalog |
| **gen-ai** | Namespace/project context integration | Integration | Evaluations scoped to user's current project |
| **platform** | RBAC model for evaluation resources | Design | Permission requirements for who can run evaluations |
| **quality-e2e-testing** | E2E test coverage for evaluation workflows | Testing | QE involvement for test scenarios |
| **UX Team** | UX prototypes review and refinement | Design | UXD prototypes by [~rh-ee-dmcgrath] provided in Jira |

## Epics & User Stories

### Phase 1: Design & Foundation (Release 3.2)

#### Epic 1.1: Evaluation Selection & Discovery UI (Priority: P1, Owner: gen-ai)

Enable users to discover and select evaluation tasks through an intuitive filtering and browsing interface that presents complex evaluation capabilities in an accessible way.

**User Value**: Users can find relevant evaluation tasks for their use case without needing to understand underlying frameworks or algorithms.

**Technical Considerations**:
- Follow tabbed configuration pattern from Playground (see packages/gen-ai/frontend/src/app/Chatbot/ChatbotSettingsPanel.tsx)
- Configuration state should be primitive-based for cloning without circular references
- Cache available evaluation tasks at session level; per-request selection

**Outcomes by Persona**:

_AI Engineer_:
- Browse evaluation collections organized by industry and use-case type (e.g., "Healthcare - Clinical NLP", "Finance - Risk Assessment")
- Filter standardized evaluations by framework (LM Eval Harness, RAGAS, GuideLLM, Garak), category, goal, or name
- See clear descriptions of evaluation task purpose without technical jargon
- Access "good range" guidance derived from public leaderboards to interpret results

_Data Scientist_:
- View detailed evaluation task metadata including upstream task/dataset links
- Access advanced view showing expected input/output schema for understanding data requirements
- Understand which evaluation tasks measure model capabilities relevant to their experiments

_Platform Engineer_:
- Understand resource implications of different evaluation types before scheduling runs
- See evaluation task availability across different namespaces/projects

> **⚠️ Dependency**: AI-Trusty team must provide evaluation task metadata API including: task name, description, framework, category, industry/use-case tags, input/output schema, and public leaderboard "good range" values.

---

#### Epic 1.2: Custom Evaluation Task Creation (Priority: P2, Owner: gen-ai)

Allow advanced users to create custom evaluation tasks when standardized evaluations don't fit their specific requirements.

**User Value**: Users can evaluate models on proprietary datasets or domain-specific tasks not covered by standard benchmarks.

**Technical Considerations**:
- Support file upload for custom datasets (leverage useFileManagement.ts pattern from Playground)
- Multi-step wizard: dataset selection/upload → framework selection → parameter configuration
- Validation of dataset format against selected framework's expected schema

**Outcomes by Persona**:

_AI Engineer_:
- Upload custom evaluation datasets in supported formats [NEEDS CLARIFICATION: supported formats TBD by AI-Trusty team]
- Generate synthetic evaluation datasets using [NEEDS CLARIFICATION: dataset generation capability TBD]
- Select existing evaluation framework to apply to custom dataset
- Provide custom evaluation logic/scoring function [NEEDS CLARIFICATION: framework extensibility API TBD]

_Data Scientist_:
- Reuse internal benchmark datasets for consistent model comparison across experiments
- Define custom metrics aligned with business-specific success criteria

> **⚠️ Dependency**: AI-Trusty team must clarify custom evaluation framework extensibility: Can users provide custom scoring functions? What dataset formats are supported? Is synthetic dataset generation in scope?

---

### Phase 2: Execution & Monitoring (Release 3.4+)

#### Epic 2.1: Evaluation Job Configuration & Submission (Priority: P1, Owner: gen-ai)

Enable users to configure and submit evaluation jobs with clear parameter controls and target selection (model vs. AI application).

**User Value**: Users can execute evaluations with appropriate configuration for their scenario, whether evaluating a standalone model or a complete AI application.

**Technical Considerations**:
- Per-job configuration isolation using custom hook pattern (see useChatbotMessages.ts)
- Each evaluation run gets independent AbortController for cancellation support
- Configuration entity should be primitive-based (similar to PaneConfiguration from Playground Compare)

**Outcomes by Persona**:

_AI Engineer_:
- Select whether to evaluate a model or an AI application (GenAI app endpoint)
- Configure number of evaluation samples to run [NEEDS CLARIFICATION: default values and valid ranges TBD]
- Set maximum tokens per response to control output length
- View estimated cost and duration before submitting [NEEDS CLARIFICATION: cost estimation API availability TBD]

_ML Ops Engineer_:
- Schedule recurring evaluation runs [NEEDS CLARIFICATION: scheduled evaluation support in 3.4 or future release]
- Configure resource limits (memory, GPU) for evaluation jobs
- Set timeout values for long-running evaluations

_Data Scientist_:
- Save evaluation configurations as templates for reuse across experiments
- Compare multiple models against the same evaluation task by queuing multiple runs

> **⚠️ Dependency**: AI-Trusty team must provide job submission API contract including: target type (model/application), sample count, max tokens, resource limits, timeout configuration.

---

#### Epic 2.2: Real-Time Progress Tracking & Cancellation (Priority: P1, Owner: gen-ai)

Provide live progress updates during evaluation execution with ability to cancel long-running jobs.

**User Value**: Users maintain visibility into evaluation progress and can abort jobs that are taking longer than expected or were misconfigured.

**Technical Considerations**:
- Leverage existing SSE (Server-Sent Events) streaming infrastructure from Playground
- State machine: idle → running → complete/error with progress percentage
- Non-blocking architecture supports multiple concurrent evaluation runs
- AbortController per-job enables individual job cancellation without affecting sibling runs

**Outcomes by Persona**:

_AI Engineer_:
- View real-time progress percentage and estimated time remaining during evaluation
- See intermediate metrics as they become available (if framework supports streaming results)
- Cancel evaluation runs that are taking longer than expected

_ML Ops Engineer_:
- Monitor resource utilization during evaluation execution [NEEDS CLARIFICATION: resource monitoring API availability TBD]
- Receive notifications when evaluation completes or fails [NEEDS CLARIFICATION: notification mechanism TBD]
- Track evaluation queue status when multiple jobs are running

> **⚠️ Dependency**: AI-Trusty team must clarify: Does evaluation API support Server-Sent Events for real-time progress? What progress granularity is available (percentage, current sample number, etc.)?

---

### Phase 3: Results & Analysis (Release 3.4+)

#### Epic 3.1: Evaluation Results Summary & Detail View (Priority: P1, Owner: gen-ai)

Present evaluation results in a clear, actionable format with summary metrics and drill-down capabilities.

**User Value**: Users can quickly assess model performance and drill into specific failure cases or edge scenarios.

**Technical Considerations**:
- Follow inline metrics display pattern from Playground (see RuntimeMetrics entity)
- Results entity structure similar to Message entity: includes metadata, metrics, configuration snapshot
- Support sorting and filtering of result sets
- Persistent storage of results for historical comparison [NEEDS CLARIFICATION: backend persistence model TBD]

**Outcomes by Persona**:

_AI Engineer_:
- View clean summary of evaluation results with primary metrics prominently displayed
- See comparison against "good range" from public leaderboards to interpret performance
- Access links to upstream evaluation task/dataset documentation for context
- Drill into individual test cases to see input, model output, and scoring rationale

_Data Scientist_:
- Export results in multiple formats (JSON, CSV, Markdown) for analysis and reporting
- Compare results across multiple evaluation runs side-by-side
- Identify failure patterns or edge cases where model underperforms

_ML Ops Engineer_:
- View execution metadata (latency, resource usage, cost) alongside accuracy metrics
- Track evaluation performance trends over time as models are retrained
- Generate evaluation reports for stakeholder review

> **⚠️ Dependency**: AI-Trusty team must provide results schema including: overall score, per-sample scores, input/output pairs, execution metadata (latency, tokens, cost).

---

#### Epic 3.2: Evaluation History & Comparison (Priority: P2, Owner: gen-ai)

Enable users to view past evaluation runs and compare results across models, configurations, or time periods.

**User Value**: Users can track model performance improvements and make data-driven decisions about which model or configuration performs best for their use case.

**Technical Considerations**:
- Session-level state management similar to ComparisonSession pattern
- Evaluation runs stored in Map<string, EvaluationRunState> for efficient lookup
- Support filtering by status (pending/running/completed/failed)
- Sorting by creation date, completion date, or metric values

**Outcomes by Persona**:

_AI Engineer_:
- View history of all evaluation runs for the current project/namespace
- Filter evaluation history by model, framework, or date range
- Compare metrics across multiple evaluation runs in table or chart format

_ML Ops Engineer_:
- Track evaluation success/failure rates to identify infrastructure issues
- Monitor cost trends for evaluation workloads over time
- Identify evaluations that consistently timeout or fail for investigation

_Data Scientist_:
- Compare current model performance against baseline or previous versions
- Identify which configuration changes improved or degraded performance
- Export comparison data for inclusion in research reports

---

### Phase 4: Advanced Capabilities (Future Releases - P3)

#### Epic 4.1: Evaluation Collections & Templates (Priority: P3, Owner: gen-ai)

Provide curated evaluation collections for common scenarios and support user-created templates.

**User Value**: Users can quickly run comprehensive evaluation suites without manually selecting individual tasks.

**Outcomes by Persona**:

_AI Engineer_:
- Run pre-defined evaluation collections for specific industries (healthcare, finance, legal)
- Run collections focused on specific capabilities (agentic AI, RAG quality, safety/hallucination detection)
- Create custom evaluation collections combining multiple tasks for their workflow

_Platform Engineer_:
- Define organization-wide evaluation standards as shareable templates
- Enforce minimum evaluation requirements for production deployments

> **⚠️ Dependency**: AI-Trusty team must provide evaluation collection metadata and support for batch job submission.

---

#### Epic 4.2: Benchmarking & Profiling Integration (Priority: P3, Owner: gen-ai)

Integrate benchmarking (performance testing) and profiling tools alongside evaluation frameworks.

**User Value**: Users can assess not just model accuracy but also performance characteristics (latency, throughput, resource efficiency) in a unified interface.

**Outcomes by Persona**:

_ML Ops Engineer_:
- Run performance benchmarks to measure model throughput and latency under load
- Profile resource utilization (GPU memory, CPU, network) during inference
- Compare performance characteristics across model serving configurations

_Platform Engineer_:
- Establish performance baselines for capacity planning
- Identify resource bottlenecks before production deployment

> **⚠️ Dependency**: AI-Trusty team must clarify integration points for GuideLLM (benchmarking) and other profiling tools in the Evaluation Stack.

---

### Edge Cases

- **Evaluation timeout**: What happens if an evaluation runs longer than configured timeout? Should partial results be available?
- **Model unavailability**: How does the UI handle evaluation of a model that becomes unavailable mid-run (e.g., serving endpoint down)?
- **Dataset size limits**: What are maximum sample counts for evaluations? How are large datasets handled?
- **Concurrent evaluation limits**: How many evaluations can run simultaneously per user/namespace?
- **Result storage limits**: How long are evaluation results retained? Is there a quota on storage?
- **Framework version changes**: How are evaluations handled when underlying framework versions change (reproducibility concerns)?
- **Application endpoint authentication**: For AI application evaluations, how are credentials/API keys provided securely?
- **Custom dataset validation**: What happens if user uploads invalid dataset format? How is feedback provided?
- **Incomplete leaderboard data**: Not all evaluation tasks have public leaderboard references - how is this indicated in UI?
- **Multi-model evaluations**: Can users evaluate multiple models against same task in batch?

### Potential Spikes

| Area | Uncertainty | Spike Goal | Recommended Timebox |
|------|-------------|------------|---------------------|
| **Evaluation Stack API Integration** | Evaluation Stack Control Plane (RHAISTRAT-26) is under development; API contracts not finalized | Verify API surface area, streaming support, job lifecycle management, results schema | 2-3 days |
| **Real-time Progress Streaming** | Unknown if evaluation frameworks support progress reporting or if all results arrive only at completion | Investigate LM Eval Harness, RAGAS, GuideLLM, Garak progress APIs; prototype SSE integration if supported | 2 days |
| **Custom Evaluation Extensibility** | Unclear if Evaluation Stack supports custom scoring functions or dataset formats beyond standard tasks | Test custom dataset upload, framework extensibility API, validation requirements | 2 days |
| **Results Persistence Model** | Unknown if results stored in backend DB, object storage, or client-side only | Review backend storage architecture, retention policies, query capabilities | 1 day |
| **Model vs. Application Evaluation** | Uncertain how AI application endpoints are registered/discovered for evaluation targeting | Investigate application registry integration, endpoint authentication, parameter passing | 2 days |
| **Large Dataset Handling** | Performance implications of evaluating with 10k+ samples; UI responsiveness during large result rendering | Test evaluation with large sample counts, identify pagination/virtualization needs | 1-2 days |
| **Cross-Framework Metric Normalization** | Different frameworks produce metrics in different formats/scales; comparability concerns | Investigate if Evaluation Stack normalizes metrics or if UI must handle conversion | 1 day |

## Performance & Scaling

### System Impact Considerations

**Client-Side (Browser)**:
- Evaluation result datasets may be large (10k+ samples with input/output pairs); UI must handle rendering large tables efficiently
- Use virtualization (PatternFly Table with dynamic loading) for displaying large result sets
- Memory footprint increases with concurrent evaluation tracking; limit active job monitoring to [NEEDS CLARIFICATION: max concurrent jobs TBD]

**API Throughput**:
- Evaluation submission may trigger concurrent model inference requests (one per sample)
- Backend must throttle evaluation execution to prevent overwhelming model serving infrastructure
- Progress polling should use efficient mechanisms (SSE preferred over REST polling) to reduce API load

**Resource Utilization**:
- Evaluation jobs consume model serving capacity; users should see warnings if evaluations will impact other workloads
- Large evaluations may require dedicated compute resources; resource quotas must be enforced per-namespace

**Latency Considerations**:
- Initial evaluation task metadata load should complete within [NEEDS CLARIFICATION: target load time TBD]
- Progress updates should arrive with minimal lag (target: <2s delay from backend event to UI update)
- Results summary should be immediately available on completion; detailed drill-down can lazy-load

**Degradation Behavior**:
- If evaluation job fails mid-execution, partial results should be preserved if available
- UI should remain responsive if one evaluation job stalls; other concurrent jobs unaffected
- Network interruptions during streaming progress should allow resume or graceful failure reporting

## Success Criteria

### Measurable Outcomes

- **SC-001**: Users can discover and select relevant evaluation tasks efficiently without requiring expert knowledge of underlying frameworks
- **SC-002**: Evaluation job submission completes with validation feedback provided immediately (no silent failures)
- **SC-003**: Progress tracking updates appear in real-time with perceptible responsiveness (no stale status indicators)
- **SC-004**: Evaluation results are presented clearly with actionable insights (comparison to leaderboard ranges, pass/fail indicators)
- **SC-005**: Users can execute evaluations on both models and AI applications without changing workflows
- **SC-006**: [NEEDS CLARIFICATION: target completion rate for evaluation jobs TBD by PM]
- **SC-007**: [NEEDS CLARIFICATION: target user adoption metrics TBD by PM]
- **SC-008**: Reduced friction in model quality assessment workflow compared to manual evaluation processes
- **SC-009**: Evaluation history enables data-driven model selection decisions (evidenced by user feedback or usage analytics)

### Qualitative Outcomes

- Regular AI developers (not just AI experts) successfully run evaluations without documentation or support
- Users trust evaluation results sufficiently to inform production deployment decisions
- Cross-framework evaluations provide meaningful comparison insights (not apples-to-oranges confusion)
- Error messages and validation feedback are clear enough for users to self-correct issues

## Open Questions for PM Review

The following questions require PM clarification before implementation:

**Q: What are the supported custom dataset formats for user-uploaded evaluations?**
A: _______________

**Q: Does the Evaluation Stack support scheduled/recurring evaluation runs in 3.4, or is this a future release?**
A: _______________

**Q: What is the maximum number of concurrent evaluation jobs per user/namespace?**
A: _______________

**Q: What are the default and maximum sample counts for evaluation runs?**
A: _______________

**Q: How long are evaluation results retained in the backend? Is there a storage quota?**
A: _______________

**Q: Is cost estimation API available for evaluation runs (to show users before submission)?**
A: _______________

**Q: What notification mechanism is used for evaluation completion (in-app, email, webhook)?**
A: _______________

**Q: Is resource monitoring (GPU, CPU, memory) API available during evaluation execution?**
A: _______________

**Q: What are the target user adoption metrics or completion rate goals for evaluations?**
A: _______________

**Q: Does the Evaluation Stack normalize metrics across frameworks, or must the UI handle conversion?**
A: _______________

**Q: How are AI application endpoints registered/discovered for evaluation targeting?**
A: _______________

**Q: Can users evaluate multiple models against the same task in batch (single submission)?**
A: _______________
