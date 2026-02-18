# ODH Planning Session Configuration

This file configures Claude Code sessions for the ODH Planning repository, automating the spec-kit workflow for feature specification.

**IMPORTANT**: Always consult the Constitution at `.specify/memory/constitution.md` for authoritative workflow guidance. The constitution defines:
- Specification discovery process (Principle X)
- Epic structure and organization (Principle XII)
- Cross-team coordination requirements (Principle VIII)
- Factual accuracy requirements (Principle XI)

## Session Initialization

When a session starts, follow this initialization flow:

### Step 0: Load Repository Context

**Always perform these steps at session start:**

1. **Clone/verify the repository** (if not already present):
   ```bash
   gh repo clone danpierce1/odh-planning
   cd odh-planning
   ```

2. **Fetch latest state**:
   ```bash
   git fetch origin --prune
   ```

3. **Load repository context**:
   - Read `.specify/memory/constitution.md` (primary authority)
   - List existing specs: `ls specs/`
   - Check all open PRs for branch numbering:
     ```bash
     gh pr list --state open --json number,title,headRefName
     ```

4. **Determine next available branch number**:
   Since new specs branch from `main`, other features may be in-flight as open PRs but not yet merged. To avoid duplicate branch numbers:
   
   ```bash
   # Get highest number from:
   # 1. Local branches
   git branch -a | grep -oE '[0-9]{3}-' | sort -rn | head -1
   
   # 2. Open PRs (critical - these aren't in main yet!)
   gh pr list --state open --json headRefName --jq '.[].headRefName' | grep -oE '^[0-9]{3}' | sort -rn | head -1
   
   # 3. Existing spec directories
   ls specs/ | grep -oE '^[0-9]{3}' | sort -rn | head -1
   ```
   
   **Use the highest number found across ALL sources + 1 for new branches.**
   
   Example: If `main` has specs up to 004, but PRs exist for 005-feature-a and 005-feature-b, the next branch should be 006.

### Step 1: Welcome and Input

Begin by greeting the user and asking for their input:

```
Welcome to ODH Planning!

What are you working on? Please provide one of:
- A Jira link (e.g., https://issues.redhat.com/browse/RHOAIENG-XXXX)
- A STRAT description for a new feature
- An existing spec ID (e.g., "4" or "004-playground-compare")
- A PR number (e.g., "#12" or "PR 12")
```

### Step 2: Determine Flow and Setup

Based on the user's input, determine which flow to follow:

**Detect Input Type:**

| Input Pattern | Flow | Action |
|---------------|------|--------|
| Jira URL (`issues.redhat.com`, `jira.`) | New Spec | Extract description, proceed to new spec flow |
| STRAT/feature description (text) | New Spec | Use as strategic input, proceed to new spec flow |
| Spec number (`4`, `004`, `004-feature-name`) | Existing Spec | Load existing branch and spec |
| PR reference (`#12`, `PR 12`, PR URL) | Existing Spec | Load PR's branch and spec |

---

**If New Specification (Jira link or STRAT description):**

1. Confirm we're on the latest `main` branch:
   ```bash
   git checkout main
   git pull origin main
   ```

2. **If Jira link provided**: Attempt to fetch description
   - If Jira CLI/integration available: Load the issue description
   - Otherwise: Ask user to paste the Jira description

3. **Ask for additional context**:
   ```
   Do you have any additional context to provide?
   - Figma designs or mockups (URL)
   - Technical documentation
   - Related specifications in this repo
   - API contracts or dependencies
   
   (Enter URLs/paths or type "skip" to continue):
   ```

4. Proceed to [Step 3: Start Specification](#step-3-start-specification-new-specs)

---

**If Existing Specification (Spec ID or PR):**

1. **Resolve the branch name**:
   - If spec number (e.g., `4`): Find matching branch `004-*`
   - If PR reference: Get branch from PR
     ```bash
     gh pr view <pr-number> --json headRefName,number,title,url,state,reviewDecision
     ```

2. List available spec branches if ambiguous:
   ```bash
   git branch -a | grep -E '[0-9]{3}-' | grep -i "<number>"
   ```

3. Checkout the branch:
   ```bash
   git checkout <branch-name>
   git pull origin <branch-name>
   ```

4. Check for existing PR on this branch:
   ```bash
   gh pr list --head <branch-name> --json number,title,url,state,reviewDecision
   ```

5. Load and summarize the existing spec files:
   - Read `specs/<branch-name>/spec.md`
   - Read any files in `specs/<branch-name>/checklists/`
   - Read any files in `specs/<branch-name>/contracts/`

6. Present a summary:
   ```
   Loaded specification: [Feature Name]
   Branch: [branch-name]
   
   PR Status: [PR #number - title] (state: open/merged/closed)
   URL: [pr-url]
   Review Decision: [approved/changes_requested/pending]
   
   -- or if no PR exists --
   PR Status: No PR created yet
   
   Epics: [count]
   
   What would you like to do?
   - Update the specification
   - Run /speckit.clarify to resolve open questions
   - Run /speckit.checklist to validate requirements
   - Create/view PR
   - Something else?
   ```

7. Await user command and proceed accordingly.

### Step 3: Start Specification (New Specs)

Once context is gathered, initiate the spec-kit workflow:

1. **Load the Constitution**: Read `.specify/memory/constitution.md` to ensure adherence to all principles

2. Summarize the collected input:
   ```
   Ready to create specification with:
   - Strategic Input: [link/description summary]
   - Additional Context: [list any provided]
   - Next branch number: [NNN] (based on Step 0 analysis)
   
   Starting specification discovery...
   ```

3. Begin the `/speckit.specify` flow (per Constitution Principle X), which will:
   - Conduct discovery through sequential clarifying questions (ONE AT A TIME)
   - Create the feature branch **off main** using `.specify/scripts/bash/create-new-feature.sh`
     - Pass `--number N` where N is the next available number determined in Step 0
     - This ensures no conflicts with branches in open PRs
   - Generate the spec.md following the template and constitution guidelines
   - Validate against the spec quality checklist

4. After specification is complete, **automatically proceed to clarify**:
   - Run `/speckit.clarify` to resolve any [NEEDS CLARIFICATION] markers
   - Continue clarification until all markers are resolved
   - Re-validate the spec after clarifications are incorporated

5. Once clarification is complete, proceed to [Step 4: Verification and PR](#step-4-verification-and-pr)

### Step 4: Verification and PR

After the spec is written and validated:

1. **Verify with User**:
   ```
   Specification complete! Please review the generated spec.md.
   
   Summary:
   - Branch: [branch-name]
   - Spec: specs/[branch-name]/spec.md
   - Epics: [count with priorities]
   - Cross-team dependencies: [list teams]
   - Potential spikes: [count]
   
   Does this specification look correct? (yes/no/needs changes)
   ```

2. **If changes needed**: Make requested updates and re-verify

3. **If verified**: Prompt for PR creation:
   ```
   Ready to open a Pull Request for review?
   
   This will:
   - Push the branch to origin
   - Create a PR targeting main
   - Add spec summary to PR description
   
   Create PR? (yes/no)
   ```

4. **Create PR** (if user confirms):
   
   **Confirm before pushing:**
   ```
   About to push branch '<branch-name>' to origin and create a PR.
   
   Changes to be pushed:
   - [list of commits/files]
   
   Proceed? (yes/no)
   ```
   
   **If confirmed:**
   ```bash
   # Push branch to remote
   git push -u origin <branch-name>
   
   # Create PR with spec summary
   gh pr create \
     --base main \
     --head <branch-name> \
     --title "Spec: [Feature Name]" \
     --body "$(cat <<'EOF'
   ## Specification: [Feature Name]
   
   **Branch**: `[branch-name]`
   **Strategic Input**: [link if provided]
   
   ### Summary
   [Brief description from spec]
   
   ### Epics
   - [ ] Epic 1: [name] (P1)
   - [ ] Epic 2: [name] (P1)
   - [ ] Epic 3: [name] (P2)
   
   ### Cross-Team Dependencies
   | Team | Requirement | Type |
   |------|-------------|------|
   | [team] | [requirement] | [type] |
   
   ### Review Checklist
   - [ ] Spec follows constitution guidelines
   - [ ] All [NEEDS CLARIFICATION] markers resolved
   - [ ] Cross-team dependencies identified
   - [ ] Potential spikes documented
   - [ ] Personas and user value clearly defined
   
   EOF
   )"
   ```

5. **Report PR creation**:
   ```
   PR created successfully!
   
   PR #[number]: [title]
   URL: [pr-url]
   
   Next steps:
   - Share PR with stakeholders for review
   - Address any review feedback
   - Once approved, spec is ready for refinement
   ```

## Key References

### Constitution (Primary Authority)
**Always consult**: `.specify/memory/constitution.md` - Crimson Dashboard Constitution (v3.0.0)

The constitution is the authoritative source for:
- Workflow principles and processes
- Specification discovery requirements (Principle X)
- Epic structure and user story format (Principle XII)
- Cross-team dependency identification (Principle VIII)
- Factual accuracy requirements - no fabricated metrics (Principle XI)
- Codebase-informed architecture guidance (Principle XIII)

### Spec-Kit Commands (defined in `.cursor/commands/`)
- `/speckit.specify` - Create new specification from description
- `/speckit.clarify` - Resolve [NEEDS CLARIFICATION] markers (run after specify)
- `/speckit.checklist` - Validate specification completeness
- `/speckit.plan` - (Not used in current workflow - spec.md is terminal artifact)

### Templates
- `.specify/templates/spec-template.md` - Specification structure
- `.specify/templates/checklist-template.md` - Requirements checklist format

### Team Ownership Reference
Consult Constitution Principle VIII for team-to-code-area mappings. Primary teams:
- `gen-ai` - GenAI features
- `model-serving-metrics` - Model serving
- `model-registry-catalog` - Model registry
- `pipelines` - ML pipelines
- `workbenches` - Notebooks/workbenches
- `platform` - Core platform

### Target Personas
All specifications must reference these personas (Constitution Section):
- **Data Scientist** - The Innovator (notebooks, experimentation)
- **AI Engineer** - The Builder (GenAI apps, prompt engineering)
- **ML Ops Engineer** - The Automator (deployment, monitoring)
- **Platform Engineer** - The Enabler (infrastructure, security)

## Workflow Principles

**Always defer to the Constitution** - These principles are summaries; the constitution is authoritative.

1. **Spec.md is the terminal artifact** - No separate planning or task generation phase (Principle IX)
2. **Sequential discovery** - Ask clarifying questions ONE AT A TIME (Principle X)
3. **Specify → Clarify → Verify → PR** - Complete each phase before proceeding
4. **Cross-team dependencies** - Identify early using OWNERS reference (Principle VIII)
5. **Spike identification** - Flag areas of uncertainty for time-boxed research (Principle X)
6. **No fabricated metrics** - Only use numbers explicitly provided by user (Principle XI)
7. **Immutable specs** - Once created, tracking happens in Jira/external systems
8. **Technology-agnostic** - Specify WHAT and WHY, never HOW to implement (Principle IX)
9. **Confirm before pushing** - Always confirm with user before any `git push` or PR creation

## Error Handling

- If branch doesn't exist: Offer to create it or list available branches
- If spec file is missing: Offer to initialize from template
- If git operations fail: Check remote connectivity, suggest manual steps
- If strategic link is inaccessible: Fall back to manual description input
- If PR creation fails: Check gh authentication, verify branch is pushed, suggest manual PR creation
- If PR already exists: Display existing PR info instead of creating duplicate
- If branch number conflict detected: Re-scan open PRs and use next available number
- If gh CLI not authenticated: Run `gh auth login` and retry

## Session Commands

The user may use these commands at any time:
- `status` - Show current branch, spec summary, and PR status
- `pr` - View existing PR or create new one for current branch
- `constitution` - Display relevant constitution principles
- `teams` - Show team ownership reference
- `personas` - Display target persona summaries

## JIRA Ticket Creation Guidelines (Optional)

When creating JIRA tickets for this project, follow these standards based on the RHOAIENG project patterns (e.g., RHOAIENG-47881).

### Labels

All tickets MUST include the appropriate scrum team label:
- `dashboard-crimson-scrum` - For Crimson scrum team work

Additional labels to apply as relevant:
- `must-do` - For critical/required work
- `needs-documentation` - When documentation is required
- `tech-reviewed` - After technical review is complete
- `<version>-candidate` (e.g., `3.4-candidate`) - Target release version

### Components

Set the appropriate component(s) for the area of work. Components help with filtering and assignment routing.

### Issue Hierarchy and Linking

Follow this hierarchy for organizing work:

```
STRAT (Feature) - RHAISTRAT project
    └── Epic - RHOAIENG project (linked via Parent Link)
            └── Story/Task - RHOAIENG project (linked via Epic Link)
```

#### Creating a STRAT-linked Epic

1. **Issue Type**: Epic
2. **Parent Link**: Link to the parent RHAISTRAT feature ticket
3. **Epic Name**: Short identifier for the epic (e.g., "mlflow-go sdk")
4. **Priority**: Set appropriately (Critical, Major, Minor, etc.)
5. **Labels**: Include scrum team label at minimum

#### Creating Stories/Tasks under an Epic

1. **Issue Type**: Story or Task
2. **Epic Link**: Link to the parent Epic
3. **Labels**: Inherit scrum team label from parent Epic

#### Issue Links

Use issue links to show dependencies between tickets:
- **Blocks/Is blocked by**: For dependency relationships between epics or tasks
- **Parent Link**: To connect Epics to STRAT features

### Description Standards

#### For Epics

Include the following in Epic descriptions:

1. **Goal/Objective**: Clear statement of what will be delivered
2. **Scope**: What is included (and optionally what is excluded)
3. **Technical Details**: Key technical information such as:
   - Target technologies/frameworks
   - API compatibility requirements
   - Testing requirements (unit, integration)
4. **Repository Link**: Link to the relevant repository if applicable

Example structure:
```
[Goal statement - what will be delivered]

[Technical scope and requirements]

Repository: [link to repo]
```

#### For STRAT Features

STRAT tickets should include comprehensive details:

1. **Problem Statement**: Why this work is needed
2. **Goal**: What success looks like
3. **Scope (MVP)**: What's in and out of scope
4. **User Experience Overview**: How users will interact
5. **Use Cases**: Numbered list of user scenarios
6. **Functional Requirements**: Grouped by category
7. **Non-Functional Requirements**: Performance, compatibility, etc.
8. **Acceptance Criteria**: Measurable success conditions
9. **Dependencies**: Links to dependent work
10. **Notes/Future Considerations**: Out-of-scope items for future

### Workflow States

1. **New**: Initial state when created
2. **In Progress**: Work has started
3. **Complete**: Work is done

### Activity Type

Set the Activity Type field appropriately:
- `New Features` - For new functionality
- (Other values as appropriate for bug fixes, improvements, etc.)

### Example: Creating an Epic for a STRAT

When asked to create an Epic for a STRAT feature:

1. Create the Epic in RHOAIENG project
2. Set Parent Link to the RHAISTRAT ticket
3. Add scrum team label (e.g., `dashboard-crimson-scrum`)
4. Set Epic Name
5. Write description with goal, scope, and technical details
6. Create child Stories/Tasks as needed with Epic Link
7. Add blocking relationships to related Epics if applicable
