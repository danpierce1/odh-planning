# Tasks: Playground Prompts

**Feature Branch**: `009-playground-prompts`
**Input**: Design documents from `/specs/009-playground-prompts/`
**Figma**: [3.4-Playground User Flow](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

---

## Phase 1: Foundation

**Purpose**: API integration and shared infrastructure required before any epic can be implemented.

**Blocking Dependency**: Razzmatazz team (RHAISTRAT-150) must provide Prompt Registry APIs.

- [ ] **T001** — Validate Prompt Registry API contract with Razzmatazz team
  > Coordinate with Razzmatazz team to confirm API endpoints for list, load, save, and version operations. Validate request/response shapes, pagination approach, and error codes match expected contract.

- [ ] **T002** — Implement API client for Prompt Registry
  > Create API client that handles authentication, error responses, and pagination for all Prompt Registry operations. This client will be used by all epics.

- [ ] **T003** — Create shared prompt state management
  > Implement state management that tracks: loaded prompt metadata (id, name, version), provenance (registry/sample/local), edit mode, original content (for change detection), and unsaved changes flag.

**Checkpoint**: API integration ready — epics can proceed

---

## Phase 2: Epic 1 — Load Prompt Modal (P1)

**Owner**: Dashboard/gen-ai
**Figma**: [Load Prompt Modal](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

**User Value**: Users can discover and load governed prompts directly within Playground, eliminating manual copy/paste, maintaining version awareness, and enabling reproducible experimentation.

### Acceptance Criteria

- [ ] SC-001: Users can load prompts from Prompt Registry into Playground via modal
- [ ] SC-003: Users can select specific versions when loading registry prompts
- [ ] SC-011: Users can filter prompts via API search in Load modal
- [ ] SC-012: Users can view commit message and tags in prompt preview sidebar

### Tasks

- [ ] **T004** [Epic1] — Add "Load prompt" link to Prompt tab header
  > Add a clickable "Load prompt" link in the Prompt tab header (within Configure side panel). Clicking this link opens the Load Prompt modal. Link should be styled consistently with existing Playground patterns.
  > **Figma**: [Prompt Tab Header](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T005** [Epic1] — Implement Load Prompt modal shell
  > Create the modal container with title "Load prompt", subtitle "Select a saved prompt from the current project or a global prompt template", and two-panel layout (table on left, preview sidebar on right). Include "Load in playground" (primary) and "Cancel" buttons.
  > **Figma**: [Load Prompt Modal](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T006** [Epic1] — Display paginated prompt table
  > Show table with columns: Name, Last version, Last modified, Tags. Each row represents a prompt from the registry. Table should handle empty state when no prompts exist.
  > **Figma**: [Prompt Table](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T007** [Epic1] — Implement pagination controls
  > Add pagination showing "1 - 20 of [total]" with page size dropdown and navigation arrows (previous/next). Pagination triggers API calls to fetch requested page. Reset to page 1 when filters change.
  > **Figma**: [Pagination](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T008** [Epic1] — Add name filter with API search
  > Add Name dropdown + search box above the table. User input triggers debounced (300ms) API call for server-side filtering. Show loading state during fetch. Clear filter button to reset.
  > **Figma**: [Name Filter](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T009** [Epic1] — Implement preview sidebar on row selection
  > When user clicks a table row, highlight the row and show preview sidebar on right side. Sidebar displays: prompt name with expand icon, version dropdown, preview text area (scrollable, read-only), last modified date, commit message, and tags.
  > **Figma**: [Preview Sidebar](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T010** [Epic1] — Add version dropdown to preview sidebar
  > Version dropdown defaults to latest version. Dropdown lists all available versions with version number. Selecting a different version fetches that version's content and updates the preview text, last modified date, and commit message.
  > **Figma**: [Version Dropdown](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T011** [Epic1] — Implement "Load in playground" action
  > Clicking "Load in playground" button loads the selected prompt and version into the Prompt tab's Instructions text area. Close the modal. Update prompt state with loaded prompt metadata (id, name, version, source). Button is disabled until a prompt is selected.

- [ ] **T012** [Epic1] — Handle cancel and close
  > Cancel button and modal X close the modal without any changes. Any selection state in the modal is discarded. If user had unsaved changes warning (from Epic 2), they remain after cancel.

**Checkpoint**: User can browse, filter, preview, select version, and load prompts from registry

---

## Phase 3: Epic 2 — Prompt State & Editing (P1)

**Owner**: Dashboard/gen-ai
**Figma**: [Loaded Prompt State](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

**User Value**: Users always know the source and state of their current prompt, can safely experiment with modifications without affecting saved versions, and can clear prompts to start fresh.

### Acceptance Criteria

- [ ] SC-004: Users can edit registry prompts without immediately affecting saved versions
- [ ] SC-008: Prompt provenance is clearly visible (name + version chip)
- [ ] SC-009: Unsaved changes are clearly indicated with "Modified" state
- [ ] SC-010: Users can clear loaded prompt to return to blank state

### Tasks

- [ ] **T013** [Epic2] — Display provenance indicator for loaded prompts
  > When a prompt is loaded, show provenance indicator above the Instructions text area. For registry prompts: "[name]" label + "Version X" chip. For example prompts: "[name]" label + "Sample" badge. For local/new prompts: no indicator (blank state).
  > **Figma**: [Provenance Indicator](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T014** [Epic2] — Set loaded prompt to read-only by default
  > When a prompt is loaded (registry or example), display the content in read-only mode. Text area should appear disabled or have visual indication of read-only state (e.g., gray background). User cannot type until edit mode is entered.

- [ ] **T015** [Epic2] — Add "Edit" and "Clear" buttons
  > Show "Edit" and "Clear" buttons below the Instructions text area when a prompt is loaded. "Edit" button only appears for registry prompts (not example prompts). "Clear" button appears for any loaded prompt.
  > **Figma**: [Edit/Clear Buttons](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T016** [Epic2] — Implement "Edit" button to enter edit mode
  > Clicking "Edit" button enables the text area for editing. User can now modify the prompt content. Button may change to "Editing" state or disappear while in edit mode. Only available for registry prompts.

- [ ] **T017** [Epic2] — Enable double-click to enter edit mode
  > Double-clicking on the Instructions text area enters edit mode (same as clicking Edit button). Cursor should be positioned where user double-clicked. Only works for registry prompts.

- [ ] **T018** [Epic2] — Show "Unsaved changes" indicator when content modified
  > When user modifies prompt content (current content differs from originally loaded content), display "Unsaved changes" badge/label near the provenance indicator. Indicator disappears when content matches original or after save/revert.
  > **Figma**: [Unsaved Changes Badge](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T019** [Epic2] — Show warning when loading prompt with unsaved changes
  > If user clicks "Load prompt" while having unsaved changes, display inline warning near the "Load prompt" link: "Loading prompt will overwrite current prompt". User can proceed (changes lost) or cancel. No separate confirmation dialog.

- [ ] **T020** [Epic2] — Implement "Clear" action
  > Clicking "Clear" button unloads the current prompt: remove content from text area, remove provenance indicator, return to blank/default state. If there were unsaved changes, they are discarded (no confirmation needed).

**Checkpoint**: User can see provenance, edit prompts, track changes, and clear to start fresh

---

## Phase 4: Epic 3 — Save Prompt (P1)

**Owner**: Dashboard/gen-ai
**Figma**: [Save Prompt Modal](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

**User Value**: Users can persist their prompt work from Playground to the Prompt Registry, creating new versions or new prompts as appropriate, with commit messages for change tracking.

### Acceptance Criteria

- [ ] SC-005: Users can save edits as new version (registry prompts) or new prompt with commit message

### Tasks

- [ ] **T021** [Epic3] — Add "Save" button to prompt area
  > Add "Save" button that opens the Save Prompt modal. Button should be visible when user has content to save (either new prompt or loaded prompt with/without changes). Consider disabling when no content exists.

- [ ] **T022** [Epic3] — Implement Save Prompt modal shell
  > Create modal with title "Save prompt", subtitle "Create a new managed chat prompt in your project." Include form fields and "Create" (primary) + "Cancel" buttons.
  > **Figma**: [Save Prompt Modal](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T023** [Epic3] — Add Name field with timestamp default
  > Name field (required) defaults to current timestamp (e.g., "05/23/2026 12:34") for new prompts. Field is editable. Show validation error if empty on submit.
  > **Figma**: [Name Field](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T024** [Epic3] — Add Prompt section with role dropdown
  > Prompt section includes: role dropdown (defaulting to "System") and text area auto-populated with current playground prompt content. User can review/edit content before saving.
  > **Figma**: [Prompt Section](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T025** [Epic3] — Add Commit message field
  > Add commit message text field with placeholder "Describe your changes". Field is optional but encouraged. Commit message is stored with the prompt version for change tracking.
  > **Figma**: [Commit Message Field](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247)

- [ ] **T026** [Epic3] — Handle save for new/local prompts
  > When saving a new prompt (not loaded from registry): create new prompt in registry with provided name, content, and commit message. On success, close modal and update prompt state to reflect saved prompt (now a registry prompt with version 1).

- [ ] **T027** [Epic3] — Handle save for existing registry prompts
  > When saving an edited registry prompt: provide option to "Save as new version" (increments version on same prompt) or "Save as new prompt" (fork with new name). UI could be radio buttons, tabs, or separate buttons.

- [ ] **T028** [Epic3] — Validate and handle name conflicts
  > On submit, validate Name is required. If API returns error that name already exists (409 Conflict), display error under Name field prompting user to choose different name. Allow retry without closing modal.

- [ ] **T029** [Epic3] — Handle API errors during save
  > If save API call fails (network error, server error), display error message in modal. Preserve form state so user can retry. Show appropriate error messages based on error type.

**Checkpoint**: User can save new prompts and new versions with commit messages

---

## Phase 5: Epic 4 — Revert Unsaved Changes (P1)

**Owner**: Dashboard/gen-ai

**User Value**: Users can safely undo experimentation and return to the original prompt state without reloading from the registry.

### Acceptance Criteria

- [ ] SC-007: Users can revert unsaved edits back to loaded version

### Tasks

- [ ] **T030** [Epic4] — Show "Revert" button when unsaved changes exist
  > Display "Revert" button/link only when the user has unsaved changes (current content differs from originally loaded content). Button appears near the Save button or in the prompt action area.

- [ ] **T031** [Epic4] — Implement revert action
  > Clicking "Revert" restores the prompt content to the originally loaded version. This is a client-side operation using cached original content — no API call needed. Operation is instant.

- [ ] **T032** [Epic4] — Update state after revert
  > After revert: "Unsaved changes" indicator disappears, "Revert" button disappears (no longer needed), user remains in edit mode (can continue editing), content shows original loaded version.

**Checkpoint**: User can revert changes without reloading from registry

---

## Phase 6: Example Prompts Support (P1)

**Builds on**: Epic 1 + Epic 2
**Figma**: [Sample Prompts Toggle](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

**User Value**: Users can browse example prompts for learning and templates, load them for immediate use, and save them as new custom prompts.

### Acceptance Criteria

- [ ] SC-002: Users can load example prompts into Playground (via "Show sample prompts" checkbox)
- [ ] SC-006: Example prompts cannot be versioned (only saved as new prompt)

### Tasks

- [ ] **T033** [Epic1] — Add "Show sample prompts" checkbox to Load modal
  > Add checkbox above/near the prompt table labeled "Show sample prompts". When unchecked (default), only registry prompts are shown. When checked, API includes example prompts in results.
  > **Figma**: [Sample Checkbox](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T034** [Epic1] — Display "Sample" badge on example prompts
  > In the prompt table, example prompts display with a "Sample" badge next to the name to distinguish them from user's registry prompts. Badge should be visually distinct (e.g., blue or gray label).
  > **Figma**: [Sample Badge](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031)

- [ ] **T035** [Epic1] — Hide version dropdown for example prompts in preview
  > When an example prompt is selected in the Load modal, the preview sidebar should not show a version dropdown (example prompts are not versioned). Show prompt details without version selection.

- [ ] **T036** [Epic2] — Show "Sample" provenance for loaded example prompts
  > When an example prompt is loaded, display "[name]" label + "Sample" badge as provenance indicator (instead of version chip). Indicates this is a template prompt, not a user's registry prompt.

- [ ] **T037** [Epic2] — Hide "Edit" button for example prompts
  > When an example prompt is loaded, do not show the "Edit" button. Example prompts are read-only and cannot be edited. User can only use as-is or save as new prompt.

- [ ] **T038** [Epic3] — Handle save for example prompts
  > When saving an example prompt: Name field defaults to today's date/time (same as new prompts, e.g., "05/23/2026 12:34"). Only option is "Save as new prompt" — no "Save as new version" since example prompts are not versioned.

**Checkpoint**: User can browse, load, and fork example prompts; cannot edit or version them

---

## Phase 7: Edge Cases & Error Handling

**Purpose**: Handle error states, edge cases, and special scenarios

### Tasks

- [ ] **T039** — Handle Prompt Registry API unavailable
  > When Prompt Registry API is unavailable: show error state in Load modal with retry option, allow continued editing of current prompt in playground, disable save to registry with appropriate message.

- [ ] **T040** — Handle very long prompts
  > Ensure prompt areas scroll for long content. Text areas in modals (preview, save) should be scrollable. Monitor editor performance with large prompts. Consider character limit warnings if registry has limits.

- [ ] **T041** — Handle MLFlow prompts with user/assistant roles
  > When loading an MLFlow-originated prompt that has "user" or "assistant" roles: display those rows as read-only content, only the system prompt portion is editable, clearly indicate which parts are editable.

- [ ] **T042** — Handle legacy "text" prompts
  > When loading a legacy "text" type prompt: automatically treat as chat-system prompt, load content into the system prompt area. User should not notice any difference in the UI.
  > 
  > **⚠️ NEEDS CLARIFICATION**: Unclear if legacy text prompts will exist in the registry or how the API will surface them. May need spike to determine if this is a real scenario or can be deferred.

- [ ] **T043** — Handle registry save failure mid-operation
  > If save operation fails (network error, timeout): show error message with details, preserve local state so user can retry, allow user to copy content as fallback if repeated failures.

**Checkpoint**: All error states and edge cases handled gracefully

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Foundation (API integration)
    ↓ BLOCKS ALL EPICS
Phase 2: Epic 1 — Load Prompt Modal ──┐
Phase 3: Epic 2 — Prompt State & Editing ──┼── Can proceed in parallel
Phase 4: Epic 3 — Save Prompt ──┤
Phase 5: Epic 4 — Revert ───────┘
    ↓
Phase 6: Example Prompts (builds on Epic 1 + Epic 2)
    ↓
Phase 7: Edge Cases & Error Handling
```

### Epic Dependencies

| Epic | Depends On | Can Run In Parallel With |
|------|------------|--------------------------|
| Epic 1 (Load) | Foundation | Epic 2, Epic 3, Epic 4 |
| Epic 2 (State/Edit) | Foundation | Epic 1, Epic 3, Epic 4 |
| Epic 3 (Save) | Foundation | Epic 1, Epic 2, Epic 4 |
| Epic 4 (Revert) | Foundation | Epic 1, Epic 2, Epic 3 |
| Example Prompts | Epic 1, Epic 2 | — |

### Recommended MVP Path

1. **Foundation** → API integration ready
2. **Epic 1** → User can browse and load prompts
3. **Epic 2** → User can see provenance, edit, track changes
4. **Demo/Validate** → Core load/edit flow works end-to-end
5. **Epic 3** → User can persist their work
6. **Epic 4** → User can undo changes
7. **Example Prompts** → Sample prompts flow
8. **Edge Cases** → Production-ready error handling

---

## Summary

| Phase | Epic | Tasks | Description |
|-------|------|-------|-------------|
| 1 | Foundation | 3 | API integration, shared state |
| 2 | Epic 1 | 9 | Load Prompt Modal |
| 3 | Epic 2 | 8 | Prompt State & Editing |
| 4 | Epic 3 | 9 | Save Prompt |
| 5 | Epic 4 | 3 | Revert Unsaved Changes |
| 6 | Example Prompts | 6 | Sample prompts support |
| 7 | Edge Cases | 5 | Error handling |
| **Total** | | **43** | |

### Figma Reference Links

| Element | Link |
|---------|------|
| User Flow Overview | [node 440:3247](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247) |
| Load Prompt Modal | [node 951:56031](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=951-56031) |
| Save Prompt Modal | [node 440:3247](https://www.figma.com/design/0KwA2EuFmA48GAQAOyjbIb/3.4-Playground?node-id=440-3247) |

**Parallel Opportunities**: Epics 1-4 can proceed in parallel after Foundation completes.
