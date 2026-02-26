# Feature Specification: Playground Prompts

**Feature Branch**: `009-playground-prompts`
**Created**: 2026-01-27
**Status**: Draft
**Input**: User description: "Playground-native Prompt experience for loading, editing, saving, and versioning prompts from Prompt Registry"

## Design References

- **Prototype**: https://andybraren.github.io/rhoai-integration-prototype/gen-ai-studio/playground
- **Figma**: https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247&t=aCrLHQvSQ2iGGXat-0

### User Flow Summary (from Figma, revised 2026-02-16)

The Prompt tab (in Configure side panel) provides entry paths for working with managed prompts:

**Path A: Draft → Save**
1. User drafts a prompt directly in the Instructions text field
2. When ready, user clicks "Save" button to open Save Prompt modal
3. Save Prompt modal fields:
   - **Name** (required) — defaults to timestamp (e.g., "05/23/2026 12:34")
   - **Prompt** — role dropdown ("System") + text area with prompt content
   - **Commit message** — describes changes for versioning
4. User clicks "Create" to save

**Path B: Load → Edit → Save**
1. User clicks "Load prompt" link to open Load Prompt modal
2. Load Prompt modal features:
   - Table with columns: Name, Last version, Last modified, Tags
   - **"Show sample prompts" checkbox** — toggles visibility of example prompts
   - **"Sample" badge** — identifies example prompts in list
   - **API filtering** — Name dropdown + search box (server-side)
   - **Pagination** — "1 - 20 of 523" with navigation
   - **Stretch**: API-based column sorting
3. User selects a prompt to see **preview sidebar**:
   - Prompt name with expand icon
   - **Version dropdown** — select specific version
   - Preview text area (scrollable)
   - Last modified date
   - **Commit message** — shows version's change description
   - Tags display
4. User clicks "Load in playground" to load
5. Loaded prompt displays with:
   - **Provenance indicator**: "[name]" + "Version X" chip
   - Instructions text area (read-only by default)
   - "Edit" and "Clear" buttons
6. User enters edit mode via "Edit" button or **double-click** on text area
7. Modified prompts show "Unsaved changes" indicator
8. User saves changes (new version or new prompt)

**Path C: Example Prompts Flow**
1. User checks "Show sample prompts" in Load Prompt modal
2. Example prompts appear with "Sample" badge
3. User previews and loads example prompt
4. Example prompt loads as read-only (can query model immediately)
5. User can save as new prompt to their project

## Epics *(mandatory)*

### Epic 1: Load Prompt Modal (Priority: P1, Owner: Dashboard/gen-ai)

Implement a "Load Prompt" modal accessible from the Prompt tab in the Configure panel, allowing users to browse, filter, preview, and load prompts from the Prompt Registry or Example Prompts with version selection.

**User Value**: Users can discover and load governed prompts directly within Playground, eliminating manual copy/paste, maintaining version awareness, and enabling reproducible experimentation.

**Technical Considerations**:
- Modal opens via "Load prompt" link in Prompt tab header
- Table-based prompt list with columns: Name, Last version, Last modified, Tags
- **"Show sample prompts" checkbox** to toggle example prompt visibility
- **"Sample" badge** on example prompts to distinguish from registry prompts
- **API filtering**: Name dropdown + search box for server-side filtering
- **Pagination**: "1 - 20 of 523" style with page size dropdown and navigation arrows
- **Stretch goal**: API-based sorting (column headers trigger server-side sort)
- **Preview sidebar** appears when a prompt is selected:
  - Prompt name with expand icon
  - Version dropdown (registry prompts only)
  - Preview text area (scrollable)
  - Last modified date
  - Commit message (version description)
  - Tags display
- **Kebab menu (⋮)** on each row for additional actions
- "Load in playground" button (primary), "Cancel" button

**Outcomes by Persona**:

_AI Engineer_:
- Click "Load prompt" to open prompt selection modal
- Browse prompts in paginated table (registry and example prompts)
- Toggle "Show sample prompts" checkbox to see example prompts
- Filter prompts by name via API search
- Select a prompt to view details in preview sidebar
- See version dropdown in sidebar for registry prompts
- View commit message describing version changes
- Switch between versions before loading
- Click "Load in playground" to load selected prompt/version

_Data Scientist_:
- Quickly find relevant prompts using search and filters
- Understand prompt purpose from preview and tags before loading

_ML Ops Engineer_:
- Reproduce experiments with specific prompt versions
- Identify prompts by tags for team organization

**Stretch Goals**:
- API-based column sorting (click column headers to sort server-side)
- Kebab menu actions: Delete prompt, Fork/Duplicate prompt
- Tag filtering in Load modal

---

### Epic 2: Prompt State & Editing (Priority: P1, Owner: Dashboard/gen-ai)

Manage prompt state in Playground including provenance display, edit mode, unsaved change tracking, and clearing loaded prompts. Example prompts are read-only and cannot be edited.

**User Value**: Users always know the source and state of their current prompt, can safely experiment with modifications without affecting saved versions, and can clear prompts to start fresh.

**Technical Considerations**:
- Provenance indicator visible in prompt area: "[name]" label + "Version X" chip
- "Edit" button to enter edit mode for registry prompts
- **Double-click on text area** as alternative way to enter edit mode
- Example prompts are read-only (no editing capability)
- Clear visual state for "Modified" / "Unsaved changes" on editable prompts
- No autosave behavior; all saves are explicit
- Inline warning near "Load prompt" link when unsaved changes exist
- "Clear" button to unload current prompt and return to blank state

**Outcomes by Persona**:

_AI Engineer_:
- See provenance indicator showing prompt name + version chip
- Click "Edit" button or double-click text area to enter edit mode
- Edit loaded registry prompts freely within Playground
- See clear "Unsaved changes" indicator when modifications exist
- See warning message near "Load prompt" when attempting to load different prompt with unsaved changes
- Click "Clear" to unload current prompt and start fresh
- Understand that example prompts are read-only templates
- Understand that changes are not automatically saved

_Data Scientist_:
- Understand governance status at a glance
- Experiment with registry prompt variations without affecting saved versions
- Use example prompts as-is for quick experimentation
- Clearly distinguish between original and modified prompt state
- Clear prompt to start over with a blank slate

**Editability by Prompt Source**:
| Prompt Source | Editable | Notes |
|---------------|----------|-------|
| Registry prompt | ✅ Yes | Can edit, save as new version or fork |
| Example prompt | ❌ No | Read-only; can only save as new prompt |
| Local/new | ✅ Yes | Editable until saved |

**Visual States**:
| State | Indicator |
|-------|-----------|
| Clean (unchanged) | No indicator |
| Modified (unsaved) | "Unsaved changes" badge/label |
| Loading different prompt | Inline warning: "Loading prompt will overwrite current prompt" |
| Example prompt loaded | "Read-only" indicator |

**Provenance Indicators**:
| Source | Display |
|--------|---------|
| Registry prompt | "[name]" label + "Version X" chip |
| Example prompt | "[name]" label + "Sample" badge |
| Local/unsaved | No indicator (blank state) |

**Actions**:
| Action | Availability | Behavior |
|--------|--------------|----------|
| Edit | Registry prompts only | Enters edit mode (also via double-click) |
| Clear | Any loaded prompt | Unloads prompt, returns to blank state |

---

### Epic 3: Save Prompt (Priority: P1, Owner: Dashboard/gen-ai)

Enable users to save prompts from Playground via the Save Prompt modal, supporting save as new version (for registry prompts), save as new prompt (fork), and save new prompt (from scratch).

**User Value**: Users can persist their prompt work from Playground to the Prompt Registry, creating new versions or new prompts as appropriate, with commit messages for change tracking.

**Technical Considerations**:
- Save options depend on prompt provenance (registry, example, local/new)
- Registry prompts can save as new version or fork as new prompt
- Example prompts can only be saved as new prompt (no versioning)
- New/unsaved prompts save as new prompt

**Save Prompt Modal** (revised design):
- Title: "Save prompt"
- Subtitle: "Create a new managed chat prompt in your project."
- **Name** field (required) — defaults to timestamp (e.g., "05/23/2026 12:34")
- **Prompt** section:
  - Role dropdown (e.g., "System") — selects prompt role type
  - Text area with prompt content (auto-populated from playground)
- **Commit message** field — "Describe your changes" for versioning context
- Buttons: "Create" (primary), "Cancel"

**Outcomes by Persona**:

_AI Engineer_:
- Save modified registry prompt as new version with commit message
- Fork any prompt as a new prompt with new name
- Save new prompt created from scratch
- Add meaningful commit messages for version history

_ML Ops Engineer_:
- Maintain version history with descriptive commit messages
- Track changes across prompt iterations
- Create team-specific forks of shared prompts

**Save Options by Provenance**:
| Prompt Source | Save as New Version | Save as New Prompt |
|---------------|---------------------|-------------------|
| Registry prompt | ✅ Yes | ✅ Yes (fork) |
| Example prompt | ❌ No | ✅ Yes |
| Local/new | ❌ No | ✅ Yes |

**Save Modal Behavior**:
- Name field defaults to timestamp for new prompts
- Prompt content auto-populates from current playground state
- Commit message field for describing changes (supports versioning workflow)

**Stretch Goal**: Save example/sample prompts as new editable custom prompts

> **Dependency**: Razzmatazz team (RHAISTRAT-150) must provide save/version APIs.

---

### Epic 4: Revert Unsaved Changes (Priority: P1, Owner: Dashboard/gen-ai)

Allow users to revert their edits back to the last loaded version, discarding unsaved changes without affecting registry content.

**User Value**: Users can safely undo experimentation and return to the original prompt state without reloading from the registry.

**Technical Considerations**:
- Revert only available when unsaved changes exist
- Reverts to last loaded version (not necessarily latest registry version)
- No confirmation dialog required (operation is reversible by re-editing)

**Outcomes by Persona**:

_AI Engineer_:
- Click "Revert" to discard unsaved changes
- Return to exact state of last loaded prompt
- Revert is available only when unsaved changes exist

_Data Scientist_:
- Safely abandon failed prompt experiments
- Quickly reset to known-good starting point

---

### Edge Cases

- What happens when Prompt Registry API is unavailable? Show error state in modal; allow continued editing of current prompt; disable save to registry.
- What happens if user tries to save a prompt with a name that already exists? Show error from registry API; prompt user to choose different name.
- What happens if the prompt version was deleted while user was editing? Save as new version fails; offer to save as new prompt instead.
- What happens when loading a legacy "text" prompt? Text prompts are automatically treated as chat-system prompts and loaded into the system prompt area.
- What happens when loading an MLFlow prompt with user/assistant roles? User and assistant rows are displayed as read-only; only the system prompt is editable.
- What happens if registry save fails mid-operation? Show error message; preserve local state; allow retry.
- What happens with very long prompts? Prompt area should scroll; consider character limit warnings if registry has limits.
- What happens when user clicks "Clear"? Prompt is unloaded, provenance indicator removed, prompt area returns to blank state.

## Potential Spikes

| Area | Uncertainty | Spike Goal | Recommended Timebox |
|------|-------------|------------|---------------------|
| Prompt Registry API Contract | APIs being built in parallel (RHAISTRAT-150) | Validate API contract for load/save/version operations | 2 days |
| MLFlow Prompt Handling | Prompts with user/assistant roles need special handling | Validate read-only row behavior and system prompt isolation | 1 day |

## Performance & Scaling

| Concern | Impact | Consideration |
|---------|--------|---------------|
| **Registry API latency** | Modal load time depends on registry response | Pagination implemented ("1-20 of 523"); API filtering reduces payload |
| **Large prompt catalogs** | Users may have hundreds of prompts | Server-side pagination and filtering; API sort as stretch goal |
| **Version history size** | Prompts with many versions may have long dropdown | Version dropdown in sidebar; consider limiting displayed versions |
| **Prompt size** | Very large prompts may impact editor performance | Preview area is scrollable; monitor editor performance |

## System Constraints

- Prompts are model-agnostic; model configuration is set separately in playground (not stored with prompt)
- No autosave; all saves are explicit user actions — all edits create new versions
- Example prompts are read-only and cannot be edited (can only be saved as new prompt)
- Example prompts cannot be versioned (only forked as new prompt)
- Prompt Registry APIs must be available for full functionality (partial degradation if unavailable)
- All prompts are treated as "chat" type with system prompt component
- Legacy "text" prompts are not supported in 3.4; they will be migrated to chat-system prompts in future releases
- Users can swap between prompt types (text ↔ chat) — existing text prompts load and save as chat-system prompts
- MLFlow-originated prompts with "user" or "assistant" roles have those rows as read-only; only system prompt is editable
- Create prompt modal uses its own UI (not duplicating MLFlow UI) for flexibility

## Key Entities

- **Prompt**: A reusable instruction template; all prompts are "chat" type with system prompt component in 3.4
- **Prompt Version**: A specific revision of a registry prompt, with version number, timestamp, and commit message
- **Commit Message**: A description of changes for a prompt version (git-like versioning semantics)
- **Prompt Registry**: The governed store for versioned prompts (RHAISTRAT-150)
- **Example Prompt**: A non-versioned example prompt for learning/templates; read-only; identified by "Sample" badge
- **Prompt Provenance**: The source and governance status of a prompt (Registry, Example, Local)
- **Tags**: Labels attached to prompts for organization and filtering (e.g., "Coding", "HR", "POC")
- **MLFlow Prompt**: A prompt originating from MLFlow UI that may contain user/assistant roles (read-only rows)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can load prompts from Prompt Registry into Playground via modal
- **SC-002**: Users can load example prompts into Playground (via "Show sample prompts" checkbox)
- **SC-003**: Users can select specific versions when loading registry prompts
- **SC-004**: Users can edit registry prompts without immediately affecting saved versions (example prompts are read-only)
- **SC-005**: Users can save edits as new version (registry prompts) or new prompt with commit message
- **SC-006**: Example prompts cannot be versioned (only saved as new prompt)
- **SC-007**: Users can revert unsaved edits back to loaded version
- **SC-008**: Prompt provenance is clearly visible (name + version chip)
- **SC-009**: Unsaved changes are clearly indicated with "Modified" state
- **SC-010**: Users can clear loaded prompt to return to blank state
- **SC-011**: Users can filter prompts via API search in Load modal
- **SC-012**: Users can view commit message and tags in prompt preview sidebar

## Cross-Team Dependencies

| Team | Requirement | Type | Notes |
|------|-------------|------|-------|
| **Razzmatazz** | Prompt Registry APIs (RHAISTRAT-150) | Integration | Load, save, version, list prompts; blocking dependency |
| **UX Team** | Load prompt modal, version selector, state indicators | Design | Modal layout, sidebar, provenance display |
| **Platform/Backend** | API integration | Integration | Proxy or direct calls to Prompt Registry |
| **QE Team** | E2E coverage for prompt workflows | Testing | Load, edit, save, revert flows |
| **Playground Team (gen-ai)** | UI implementation | Integration | Main implementation team |

### Blocking Dependencies

- **Razzmatazz (RHAISTRAT-150)**: Prompt Registry APIs must be available for registry prompt functionality
- **UX Team**: Designs for load modal, version selection, and state indicators

### Informational (Capacity/Awareness)

- **Docs Team**: New prompt workflow needs user documentation

## Needs Clarification

The following questions require PM input before finalizing the specification:

### Q1: Example Prompts Source
Where do example prompts come from?
1. Bundled with product - Example prompts shipped as static content with Dashboard
2. ConfigMap/admin-managed - Platform admin configures example prompts
3. External repository - Fetched from a Red Hat-managed samples repository
4. Other

### Q2: Save Destination Model
When a user saves a new prompt or new version from Playground, where does it go?
1. Always to Prompt Registry (governed)
2. User chooses destination (registry, personal workspace, etc.)
3. Local only option (browser/session) without registry
4. Registry within user's namespace
5. Other

## Clarifications

### Session 2026-02-16 (Design Revision)

- Q: How are example prompts accessed in Load modal? → A: "Show sample prompts" checkbox toggles visibility; example prompts marked with "Sample" badge
- Q: What filtering/sorting does Load modal support? → A: API filtering (Name dropdown + search), pagination ("1-20 of 523"); API sort is stretch goal
- Q: What information is shown in Load modal preview sidebar? → A: Prompt name, version dropdown, preview text, last modified, commit message, tags
- Q: How does Save Prompt modal work? → A: Name (defaults to timestamp), Prompt (role dropdown + text), Commit message field, "Create" button
- Q: How does user enter edit mode for loaded prompt? → A: "Edit" button or double-click on text area
- Q: What are stretch goals for 3.4? → A: Kebab menu (delete, fork/duplicate), tag filtering, save example prompts as editable custom prompts

### Session 2026-02-16

- Q: What is the canonical term for non-versioned template prompts (starter/global/example)? → A: "Example prompts"
- Q: Are example prompts in scope for 3.4? → A: Yes, in scope — browsable and loadable in load modal
- Q: Should example prompts have dedicated tab or mixed list in load modal? → A: Resolved — checkbox toggle ("Show sample prompts") with "Sample" badge
- Q: Should save modal include model configuration? → A: No — prompts are model-agnostic; model set separately in playground
- Q: When saving an example prompt, should name pre-populate? → A: No — blank name; user must provide new name
- Q: Are epics still valid after Figma review? → A: Consolidated from 6 to 4 epics; added "Clear" action to Epic 2

### Design Refinement Session 2026-02-11

- Q: Should we duplicate the MLFlow UI in the create prompt modal? → A: No, use different UI for flexibility
- Q: Can users swap prompt types? → A: Yes, can swap between text and chat types
- Q: How are edits treated? → A: All edits to prompts are treated as new versions
- Q: Do we support the old "text" prompt type? → A: Not for 3.4; text prompts will later be translated to system prompts
- Q: How are prompts treated? → A: All prompts treated as "chat" type; existing text prompts load and save as chat-system prompts
- Q: What about MLFlow prompts with user/assistant roles? → A: User/assistant rows are read-only; only system prompt is editable
- Q: What do we call sample prompts? → A: "Example prompts"; namespace/published prompts come later

### Session 2026-01-27

- Q: Is Prompt Registry available? → A: Being built in parallel (RHAISTRAT-150); some APIs exist, others need coordination
- Q: Where does "Load Prompt" appear? → A: Button in existing prompt area opens modal/drawer
- Q: How does version selection work? → A: Sidebar in modal shows prompt details with version dropdown
- Q: What happens with unsaved changes when loading new prompt? → A: Inline warning "Loading prompt will overwrite current prompt" (no separate dialog)
- Q: What prompt types exist? → A: "text" or "chat"; chat has system prompt component
- Q: Who owns Prompt Registry? → A: Razzmatazz team (RHAISTRAT-150)

## Assumptions

- Prompt Registry (RHAISTRAT-150) will provide APIs for listing, loading, saving, and versioning prompts
- The existing Playground prompt area can accommodate provenance indicators and state badges
- Example prompts exist and are accessible (source to be clarified)
- Users understand the difference between governed (registry) and example prompts
- Modal/drawer pattern is consistent with existing Playground UI patterns
- Prompt Registry handles authorization (who can save/version prompts)

## Future Considerations (Post-MVP)

**Stretch Goals for 3.4** (if capacity allows):
- API-based sorting in Load modal
- Kebab menu actions: Delete prompt, Fork/Duplicate prompt
- Tag filtering in Load modal
- Save example/sample prompts as new editable custom prompts

**Post-3.4**:
- Namespace/published prompts (beyond example prompts)
- Support for legacy "text" prompt type migration to chat-system prompts
- Visual diff between prompt versions
- Inline prompt metadata editing (tags, owner, status)
- Read-only view for certified prompts
- Prompt comparison across models (ties into eval workflows)
- Approval workflows for prompt changes
- Non-playground consumers (SDK, CI, etc.)
