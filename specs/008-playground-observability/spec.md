# Feature Specification: Playground Observability

**Feature Branch**: `008-playground-observability`
**Created**: 2026-01-27
**Status**: Draft
**Input**: User description: "Metrics and observability dashboard providing performance and cost insights for each chat instance, plus deep tracing of interactions"

## Epics *(mandatory)*

### Epic 1: Metrics Collection Backend (Priority: P1, Owner: Platform/Backend)

Implement backend infrastructure to capture and return performance metrics with inference responses, enabling the Dashboard to display timing and token usage data.

**User Value**: Users receive accurate, backend-measured metrics rather than estimates, providing reliable data for performance evaluation and optimization.

**Technical Considerations**:
- Metrics returned as part of inference response headers or body
- Must support streaming responses (metrics finalized on completion)
- Core metrics: total tokens (input/output), basic timing

**Outcomes by Persona**:

_AI Engineer_:
- Receive accurate token counts with each inference response
- Access timing data measured at the backend (not client-side estimates)

_ML Ops Engineer_:
- Metrics data available for aggregation and analysis
- Consistent metric format across different model backends

**Core Metrics (v1 Required)**:
| Metric | Description | Source |
|--------|-------------|--------|
| Total Input Tokens | Token count for prompt/context | Backend response |
| Total Output Tokens | Token count for generated response | Backend response |
| Total Latency | End-to-end request duration | Backend response |

**Stretch Metrics (if time permits)**:
| Metric | Description | Source |
|--------|-------------|--------|
| Time-to-First-Token | Latency until first token streams | Backend response |
| Tokens/sec | Output token generation rate | Calculated |
| Request/Response Size | Payload sizes in bytes | Backend response |
| Estimated Cost | Based on model pricing metadata | Calculated from registry |

> **Dependency**: Model Serving team must include metrics in inference response.

---

### Epic 2: Per-Message Metrics Display (Priority: P1, Owner: Dashboard/gen-ai)

Display performance metrics inline beneath each chat message in the Playground, providing immediate visibility into model performance.

**User Value**: Users can evaluate model performance for each interaction without leaving the chat interface or accessing external tools.

**Technical Considerations**:
- Metrics displayed after response completes (not during streaming)
- Per-pane inline display for Playground Compare scenarios
- Compact, non-intrusive presentation that doesn't disrupt chat flow

**Outcomes by Persona**:

_AI Engineer_:
- See token counts displayed beneath each assistant response
- View latency for each message
- Compare metrics across messages within a session

_Data Scientist_:
- Quickly assess which prompts result in longer/shorter responses
- Identify patterns in token usage across conversation

---

### Epic 3: Session Aggregate Metrics (Priority: P1, Owner: Dashboard/gen-ai)

Provide session-level aggregate metrics showing running totals for the entire chat session, enabling users to track cumulative usage.

**User Value**: Users understand total resource consumption for their experimentation session, informing cost awareness and usage patterns.

**Technical Considerations**:
- Running totals update after each message completes
- Aggregates persist for session duration (lost on refresh)
- Display alongside or above per-message metrics

**Outcomes by Persona**:

_AI Engineer_:
- See cumulative token count for entire session
- Track total session duration and message count
- Understand aggregate cost implications (when cost data available)

_ML Ops Engineer_:
- Evaluate total resource consumption for experimentation workflows

**Session Aggregates**:
- Total messages in session
- Cumulative input tokens
- Cumulative output tokens
- Total session latency (sum of all request durations)

---

### Epic 4: Multi-Pane Metrics Comparison (Priority: P1, Owner: Dashboard/gen-ai)

Support side-by-side metric comparison when multiple chat panes are active in Playground Compare, enabling direct performance comparison across models or configurations.

**User Value**: Users can directly compare model performance, token efficiency, and latency across different configurations in a single view.

**Technical Considerations**:
- Each pane displays its own metrics inline (per Epic 2)
- Metrics layout consistent across panes for easy visual comparison
- Session aggregates per-pane (not combined across panes)

**Outcomes by Persona**:

_AI Engineer_:
- Compare token usage across models side-by-side
- Identify which model/configuration responds faster
- Evaluate cost efficiency across different configurations

_Data Scientist_:
- Run same prompt across models and compare metrics
- Make data-driven model selection decisions

---

### Epic 5: Metrics Export (Priority: P2, Owner: Dashboard/gen-ai)

Enable users to export session metrics in OpenTelemetry format for analysis in external observability tools.

**User Value**: Users can integrate Playground experimentation data into their existing observability pipelines and perform deeper analysis.

**Technical Considerations**:
- Export in OTLP/JSON format for compatibility
- Include per-message metrics and session aggregates
- Export triggered by user action (button click)

**Outcomes by Persona**:

_AI Engineer_:
- Export session metrics for offline analysis
- Import metrics into Jaeger, Grafana, or other OTLP-compatible tools
- Share performance data with team members

_ML Ops Engineer_:
- Aggregate exported metrics across sessions
- Build dashboards from exported data

---

### Epic 6: Tracing Integration (Priority: P2, Conditional on RHAIRFE-710, Owner: Dashboard/gen-ai)

If tracing is enabled via RHAIRFE-710, display execution traces showing the flow between LLM, MCP servers, knowledge sources, and guardrails.

**User Value**: Users can identify bottlenecks in complex workflows involving multiple components (MCP, RAG, guardrails) and understand where time is spent.

**Technical Considerations**:
- Conditional on RHAIRFE-710 delivering tracing infrastructure
- Simple ordered list visualization of spans with timing
- Trace data sourced from llama-stack telemetry (if available)
- Expandable per-message to show trace details

**Outcomes by Persona**:

_AI Engineer_:
- Expand a message to see execution trace
- Identify which component (LLM, MCP, RAG, guardrail) contributed most to latency
- Diagnose slow MCP or knowledge source retrieval

_ML Ops Engineer_:
- Analyze execution flow for optimization opportunities
- Export traces in OpenTelemetry format for external analysis

**Trace Visualization (Simple List)**:
```
▼ Message Trace (1.2s total)
  ├─ Guardrail (input)     12ms
  ├─ Knowledge Retrieval   234ms
  ├─ MCP: GitHub           156ms
  ├─ LLM Inference         780ms
  └─ Guardrail (output)    18ms
```

> **Dependency**: RHAIRFE-710 must land for tracing to be available. If not available, this epic is deferred.

---

### Epic 7: Cost Estimation (Priority: P3, Stretch, Owner: Dashboard/gen-ai)

Display estimated cost per message and session based on model pricing metadata from the Model Registry.

**User Value**: Users can estimate costs for their experimentation and make cost-aware decisions about model selection.

**Technical Considerations**:
- Pricing data from Model Registry/preset metadata (per STRAT)
- Cost displayed only when pricing data available for the model
- Show "N/A" or hide cost when pricing unavailable

**Outcomes by Persona**:

_AI Engineer_:
- See estimated cost per message (when pricing available)
- View cumulative session cost estimate
- Compare cost efficiency across models

**Cost Calculation**:
- Cost = (input_tokens × input_price_per_token) + (output_tokens × output_price_per_token)
- Displayed as currency (e.g., "$0.0023") or "Pricing unavailable"

---

### Edge Cases

- What happens if backend doesn't return metrics? Display "Metrics unavailable" rather than empty/broken UI.
- What happens during streaming before completion? Metrics area shows loading/pending state; populated on completion.
- What happens if one pane in Compare has metrics and another doesn't? Each pane independent; show what's available per pane.
- What happens if tracing is partially available? Show available spans; indicate if trace is incomplete.
- What happens with very long traces (many MCP calls)? Consider collapsing/truncating with "show all" option.
- What happens if pricing metadata is missing for a model? Hide cost or show "N/A"; don't show $0.00.
- What happens on page refresh? All metrics and aggregates lost (session-only persistence).

## Performance & Scaling

| Concern | Impact | Consideration |
|---------|--------|---------------|
| **Metrics overhead** | Backend must calculate and return metrics | Minimal overhead; token counting already needed for context management |
| **UI rendering** | Metrics displayed for every message | Lightweight display; no complex visualizations in v1 |
| **Export size** | Long sessions may have large export files | Consider chunking or size limits for very long sessions |
| **Multi-pane metrics** | 4 panes × many messages = many metrics | Render efficiently; consider virtualization for very long chats |

## System Constraints

- Metrics shown on completion only (not during streaming)
- Metrics sourced from backend inference response (not client-side timing)
- Session aggregates persist only for session duration (lost on refresh)
- Tracing conditional on RHAIRFE-710 availability
- Cost estimation requires pricing metadata in Model Registry
- Export format is OpenTelemetry (OTLP/JSON)

## Key Entities

- **Message Metrics**: Performance data associated with a single chat message (tokens, latency)
- **Session Metrics**: Aggregate performance data for an entire chat session
- **Execution Trace**: Ordered list of spans showing component-level timing (LLM, MCP, RAG, guardrails)
- **Span**: A single unit of work within a trace (e.g., "MCP: GitHub" with duration)
- **Model Pricing**: Cost-per-token metadata from Model Registry used for cost estimation

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Token counts (input/output) appear for each message after completion
- **SC-002**: Basic timing/latency displayed per message
- **SC-003**: Session aggregates show cumulative totals (tokens, messages, duration)
- **SC-004**: Metrics display correctly in multi-pane Playground Compare scenarios
- **SC-005**: Users can export session metrics in OpenTelemetry format
- **SC-006**: (If RHAIRFE-710 lands) Users can expand messages to see execution trace
- **SC-007**: (Stretch) Estimated cost displayed when model pricing metadata available
- **SC-008**: Metrics panel does not disrupt chat UX or degrade performance

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **Model Serving** | Return metrics in inference response | Integration | Token counts, timing; blocking dependency |
| **llama-stack Team** | Tracing data (if RHAIRFE-710 lands) | Integration | Span data for execution traces; conditional |
| **Model Registry** | Pricing metadata for cost estimation | Integration | Stretch goal; not blocking |
| **UX Team** | Metrics panel and trace visualization design | Design | Layout, placement, export UX |
| **QE Team** | Validate metrics accuracy | Testing | Compare displayed vs. actual values |
| **Playground Team (gen-ai)** | UI implementation | Integration | Main implementation team |

### Blocking Dependencies

- **Model Serving**: Must return token counts and timing in inference response before metrics can display
- **UX Team**: Designs for metrics panel layout and trace visualization

### Conditional Dependencies

- **llama-stack Team / RHAIRFE-710**: Tracing epic depends on this landing; if not, defer to future release

### Informational (Capacity/Awareness)

- **Model Registry Team**: FYI on pricing metadata usage for cost estimation (stretch goal)

## Clarifications

### Session 2026-01-27

- Q: Is tracing available? → A: Conditional on RHAIRFE-710; spec both paths
- Q: Where do metrics come from? → A: Backend/inference response headers
- Q: How is cost calculated? → A: From Model Registry pricing metadata
- Q: How are metrics displayed? → A: Per-pane inline beneath each message
- Q: When are metrics updated? → A: On completion only (not during streaming)
- Q: Do metrics persist? → A: Session aggregates for session duration; per-message history retained
- Q: What export format? → A: OpenTelemetry (OTLP/JSON)
- Q: How are traces visualized? → A: Simple ordered list of spans with timing
- Q: Which metrics are required for v1? → A: Minimal MVP (total tokens, basic timing); rest are stretch

## Assumptions

- Model Serving infrastructure can return token counts and timing with inference responses
- The existing Playground UI can accommodate inline metrics display without major refactoring
- OpenTelemetry export format is appropriate for the expected user base
- RHAIRFE-710 tracing work will expose trace data in a consumable format (if it lands)
- Model Registry schema can be extended to include pricing metadata for cost estimation
- Users understand basic observability concepts (tokens, latency, traces)

## Future Considerations (Post-v1)

- GPU/memory utilization metrics
- Real-time streaming metric updates (TTFT shown immediately)
- Advanced trace visualization (waterfall/timeline)
- Cost aggregation per run or session
- Alerting on cost thresholds
- Metrics persistence beyond session (history)
- Integration with external observability platforms (Prometheus, Grafana)
