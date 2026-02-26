# ODH Planning

Planning and specification repository for OpenShift Data Hub (ODH) features, with a focus on the Crimson Dashboard.

## Purpose

This repository provides a structured approach to feature specification using the `.specify` framework. Specifications feed directly into team refinement sessions for manual breakdown and sprint planning.

## Workflow

```
Strategic Input → /speckit.specify → spec.md → Refinement Session → Sprint
```

- **`spec.md`** is the terminal artifact — no separate planning or task generation phase
- Specifications are immutable after creation; work tracking happens in external systems (Jira, etc.)
- One spec per strategic input; may contain multiple epics across sprints/teams

## Repository Structure

```
specs/
├── 001-playground-compare/      # Feature: Model comparison in playground
├── 002-playground-prompt-ui/    # Feature: Prompt-centric playground UI
└── 003-playground-prompt-lab/   # Feature: Prompt lab functionality
```

Each feature folder contains:
- **spec.md** - Feature specification with epics, dependencies, edge cases, and potential spikes (required)
- **checklists/** - Requirements and acceptance criteria (optional)
- **contracts/** - API contracts and dependencies (optional)
- **coordination/** - Executive summaries and team briefs (optional)

## Constitution

The planning framework follows the [Crimson Dashboard Constitution](.specify/memory/constitution.md), which defines:
- Target personas (Data Scientist, AI Engineer, ML Ops Engineer, Platform Engineer)
- Core principles for specification and development
- Epic structure and cross-team coordination requirements
- Team ownership reference (from [odh-dashboard OWNERS](https://github.com/opendatahub-io/odh-dashboard))

## Contributing

See [CODEOWNERS](.github/CODEOWNERS) for team ownership and review requirements.

## License

Apache License 2.0 - See [LICENSE](LICENSE) for details.
