# Rule: Generating a Task List from User Requirements

## Goal

To guide an AI assistant in creating a detailed, step-by-step task list in Markdown format based on user requirements, feature requests, or existing documentation. The task list should guide a developer through implementation.

## Output

- **Format:** Markdown (`.md`)
- **Location:** `/tasks/`
- **Filename:** `tasks-[feature-name].md` (e.g., `tasks-user-profile-editing.md`)

## Process

1.  **Receive Requirements:** The user provides a feature request, task description, or points to existing documentation
2.  **Analyze Requirements:** The AI analyzes the functional requirements, user needs, and implementation scope from the provided information
3.  **Phase 1: Generate Parent Tasks:** Based on the requirements analysis, create the file and generate the main, high-level tasks required to implement the feature. **IMPORTANT: Always include task 0.0 "Create feature branch" as the first task, unless the user specifically requests not to create a branch.** Use your judgement on how many additional high-level tasks to use. It's likely to be about 5. Present these tasks to the user in the specified format (without sub-tasks yet). Inform the user: "I have generated the high-level tasks based on your requirements. Ready to generate the sub-tasks? Respond with 'Go' to proceed."
4.  **Wait for Confirmation:** Pause and wait for the user to respond with "Go".
5.  **Phase 2: Generate Sub-Tasks:** Once the user confirms, break down each parent task into smaller, actionable sub-tasks necessary to complete the parent task. Ensure sub-tasks logically follow from the parent task and cover the implementation details implied by the requirements.
6.  **Identify Relevant Files:** Based on the tasks and requirements, identify potential files that will need to be created or modified. List these under the `Relevant Files` section, including corresponding test files if applicable.
7.  **Generate Final Output:** Combine the parent tasks, sub-tasks, relevant files, and notes into the final Markdown structure.
8.  **Save Task List:** Save the generated document in the `/tasks/` directory with the filename `tasks-[feature-name].md`, where `[feature-name]` describes the main feature or task being implemented (e.g., if the request was about user profile editing, the output is `tasks-user-profile-editing.md`).

---

## Parent Task Completion Sequence

Every parent task ends with the same three mandatory steps, in this order:

```
[Implementation sub-tasks: 1.1, 1.2, 1.3, ...]
        ↓
Write task summary → tasks/task_summaries/
        ↓
Code review → tasks/code_reviews/
        ↓
Code cleanup → remove temp files, verify clean directory
        ↓
Move to next parent task (2.0)
```

The AI must not start the next parent task until the summary, code review, and cleanup are all complete.

---

## Task Summary Documents

### Purpose

After completing all implementation sub-tasks within a parent task, a **task summary document** must be written. This captures decisions, insights, and outcomes while they're fresh and creates a knowledge trail for the team.

### When to Write

- **After:** All implementation sub-tasks under a parent task are complete.
- **Before:** The code review.

### Output

- **Format:** Markdown (`.md`)
- **Location:** `/tasks/task_summaries/`
- **Filename:** `summary-[feature-name]-[track]-task-[N].md`
  - Example: `summary-mvp1-intake-system-frontend-task-1.md`
  - Example: `summary-mvp1-intake-system-adk-agents-task-3.md`

### Summary Document Structure

Every task summary must follow this structure:

```markdown
# Task Summary: [Parent Task Title]

## Task Reference

- **Feature:** [Feature name]
- **Track:** [Frontend | FastAPI Backend | ADK Agents | Integration]
- **Task:** [N.0 — Parent Task Title]
- **Date Completed:** [YYYY-MM-DD]
- **Task File:** `tasks/tasks-[feature-name]-[track].md`

## What Was Built

A brief description of what was implemented in this parent task. 2-4 sentences covering the scope of work completed.

## Key Decisions

Decisions made during implementation that future developers should know about. Include the decision, the alternatives considered, and why this approach was chosen.

- **Decision:** [What was decided]
  - **Why:** [Reasoning]
  - **Alternatives considered:** [What else was considered and why it was rejected]

## Key Insights

Things learned during implementation that weren't obvious from the requirements. Gotchas, surprises, or important context that would help someone working on related tasks.

- [Insight 1]
- [Insight 2]

## Key Outcomes

Concrete results of the work. What now exists that didn't before.

- [File created/modified and what it does]
- [Endpoint added and what it serves]
- [Component built and what it renders]

## Files Created or Modified

| File | Action | Description |
|------|--------|-------------|
| `path/to/file.ts` | Created | Brief description |
| `path/to/existing.py` | Modified | What changed and why |

## Dependencies and Impact

- Any new dependencies added (npm packages, pip packages)
- Any impact on other tracks or existing functionality
- Any follow-up work identified that isn't in the current task list

## Open Issues or TODOs

- Any unresolved questions or deferred decisions
- Any known limitations or technical debt introduced
- Any items that should be addressed in a future task
```

---

## Code Review Documents

### Purpose

After the task summary is written, the AI conducts a **code review** of everything built in the parent task. The goal is to walk through the code as if explaining it to a junior developer — verifying that the approach is clear, the code follows project best practices, and the process flow is easy to follow. This is also a quality gate to catch anything that doesn't meet the project's coding standards.

### What the Code Review Covers

The AI must walk through each file created or modified in the parent task and review:

1. **Approach & Architecture:** Is the overall approach sound? Does it follow the project's patterns (adapter pattern, YAML-driven behavior, dumb frontend, etc.)?
2. **Code Workflow:** Can a junior developer follow the code from entry point to output? Is the flow logical and easy to trace?
3. **Function Documentation:** Does every function have a docstring/JSDoc with description, inputs, and outputs per the project standards?
4. **Code Comments:** Do comments explain "why" not "what"? Are business logic decisions documented inline?
5. **Readability:** Is the code readable by a junior developer? Are variable names descriptive? Are there nested ternaries, clever one-liners, or deep nesting that should be refactored?
6. **Error Handling:** Does every API call, async operation, and adapter call have proper try/catch with meaningful error messages?
7. **Best Practices:** No `console.log` (frontend) or `print()` (backend). Type hints on all Python functions. No `any` types in TypeScript. Named exports. Constants instead of hardcoded strings.
8. **Testing:** Are there tests for the new code? Do they cover at least the happy path?

### How the Code Review Works

The code review is a **conversation between the AI and the user**. The AI presents its review of the code, walking through the important pieces, and the user can ask questions, request changes, or approve. If changes are needed, they are made before the review document is finalized.

The review should be structured, not a wall of text. Walk through file by file, calling out what's good and what needs attention.

### When to Write

- **After:** The task summary is written.
- **Before:** Starting the next parent task.

### Output

- **Format:** Markdown (`.md`)
- **Location:** `/tasks/code_reviews/`
- **Filename:** `review-[feature-name]-[track]-task-[N].md`
  - Example: `review-mvp1-intake-system-frontend-task-1.md`
  - Example: `review-mvp1-intake-system-adk-agents-task-3.md`

### Code Review Document Structure

Every code review document must follow this structure:

```markdown
# Code Review: [Parent Task Title]

## Review Reference

- **Feature:** [Feature name]
- **Track:** [Frontend | FastAPI Backend | ADK Agents | Integration]
- **Task:** [N.0 — Parent Task Title]
- **Date Reviewed:** [YYYY-MM-DD]
- **Task File:** `tasks/tasks-[feature-name]-[track].md`
- **Summary File:** `tasks/task_summaries/summary-[feature]-[track]-task-[N].md`

## Code Structure Overview

A high-level map of how the code in this task fits together. Show the flow from entry point to output so a junior developer can understand the big picture before reading individual files.

```
[entry point] → [function/component A] → [function/component B] → [output/side effect]
```

## File-by-File Review

### `path/to/file1.ts`

**Purpose:** [What this file does in one sentence]

**Approach:** [Is the approach sound? Does it follow project patterns?]

**Walkthrough:** [Walk through the important parts of the code — key functions, data flow, decision points. Explain it as if teaching a junior developer.]

**Documentation:** [Are docstrings/JSDoc present and complete?]

**Readability:** [Any readability concerns? Naming issues? Complex logic that should be simplified?]

**Issues Found:**
- [Issue 1 — description and recommendation]
- [Issue 2 — description and recommendation]
- None — code meets standards.

### `path/to/file2.py`

[Same structure repeated for each file]

## Best Practices Checklist

| Check | Status | Notes |
|-------|--------|-------|
| All functions have docstrings/JSDoc | ✅ / ❌ | [Details if failed] |
| Comments explain "why" not "what" | ✅ / ❌ | |
| No `console.log` or `print()` | ✅ / ❌ | |
| Type hints on all Python functions | ✅ / ❌ | |
| No `any` types in TypeScript | ✅ / ❌ | |
| Error handling on async operations | ✅ / ❌ | |
| Descriptive variable/function names | ✅ / ❌ | |
| No deep nesting (max 2-3 levels) | ✅ / ❌ | |
| Tests cover happy path | ✅ / ❌ | |
| No hardcoded strings for config values | ✅ / ❌ | |

## Changes Made During Review

| File | Change | Reason |
|------|--------|--------|
| `path/to/file.ts` | [What was changed] | [Why it was changed] |
| None | — | Code passed review without changes |

## Final Assessment

[2-3 sentences summarizing the overall quality of the code, whether it's ready to proceed, and any notes for future tasks.]
```

---

## Code Cleanup

### Purpose

After the code review, the AI performs a **code cleanup** — scanning the project directory for any temporary, debug, or unnecessary files created during the parent task's implementation. This keeps the project directory clean, modular, and free of clutter. The AI should not create files that aren't part of the project structure unless the user explicitly asks for them.

### What Gets Removed

The cleanup targets files that were created during development but are not part of the project's intended architecture:

- **Temporary scripts:** Debug scripts, one-off test scripts, throwaway utilities (e.g., `test_fix.py`, `debug_sse.sh`, `temp_check.js`)
- **Scratch markdown files:** Notes, explanations, or descriptions that aren't task summaries, code reviews, or project documentation (e.g., `notes.md`, `fix_explanation.md`, `scratch.md`)
- **Debug or log output files:** Files generated during debugging (e.g., `output.log`, `debug_output.txt`, `response_dump.json`)
- **Duplicate or backup files:** Copies made during troubleshooting (e.g., `ChatWindow_backup.tsx`, `main_old.py`, `routes_copy.py`)
- **Unused boilerplate:** Files generated by tools or scaffolding that aren't needed (e.g., empty config files, unused template files)
- **Any file not in the established project structure** defined in `CLAUDE.md` unless the user explicitly requested it

### What Stays

- All files in the established project structure (`frontend/src/`, `backend/app/`, `tasks/`, etc.)
- Task summaries (`tasks/task_summaries/`)
- Code review documents (`tasks/code_reviews/`)
- Test files in their proper directories (`frontend/src/**/*.test.ts`, `backend/tests/`)
- Configuration files (`.env`, `.gitignore`, `package.json`, `requirements.txt`, YAML configs)
- Requirements docs and workflow templates

### How It Works

1. The AI lists all files in the project directory that were created or modified during the parent task.
2. The AI identifies any files that don't belong in the established project structure.
3. The AI presents the list of files to remove to the user for confirmation.
4. After user approval, the AI deletes the files and confirms the cleanup.
5. The AI verifies the directory structure is clean — no orphaned files outside the expected locations.

### Proactive Prevention

The AI should avoid creating temporary files in the first place. When debugging or testing:

- Use inline code execution or terminal output instead of writing to files.
- If a temporary file is absolutely necessary, delete it immediately after use within the same sub-task.
- Never create markdown files to explain a process unless the user explicitly requests it — put explanations in the chat conversation instead.
- Never create backup copies of files — Git handles version history.

### When to Run

- **After:** The code review is complete.
- **Before:** Starting the next parent task.

---

## How Summary, Review, and Cleanup Appear in Task Files

The summary, code review, and code cleanup are the **last three sub-tasks under every parent task**. All three are mandatory and non-skippable:

```markdown
- [ ] 1.0 Build Chat Window Component
  - [ ] 1.1 Create ChatWindow.tsx with message list and auto-scroll
  - [ ] 1.2 Create MessageBubble.tsx with role-based styling
  - [ ] 1.3 Create ChatInput.tsx with send button and disabled state
  - [ ] 1.4 Write unit tests for ChatWindow, MessageBubble, ChatInput
  - [ ] 1.5 **Write task summary** → save to `tasks/task_summaries/summary-[feature]-[track]-task-1.md`
  - [ ] 1.6 **Code review** → save to `tasks/code_reviews/review-[feature]-[track]-task-1.md`
  - [ ] 1.7 **Code cleanup** → remove temp files, verify clean directory structure
- [ ] 2.0 Build SSE Stream Handler
  - [ ] 2.1 Create useChat hook with SSE stream consumption
  - [ ] 2.2 Handle all SSE event types (message, ui_control, progress, etc.)
  - [ ] 2.3 Write unit tests for useChat hook
  - [ ] 2.4 **Write task summary** → save to `tasks/task_summaries/summary-[feature]-[track]-task-2.md`
  - [ ] 2.5 **Code review** → save to `tasks/code_reviews/review-[feature]-[track]-task-2.md`
  - [ ] 2.6 **Code cleanup** → remove temp files, verify clean directory structure
```

**The AI must not proceed to the next parent task until the summary, code review, and cleanup are all complete.**

---

## Output Format

The generated task list _must_ follow this structure:

```markdown
## Relevant Files

- `path/to/potential/file1.ts` - Brief description of why this file is relevant (e.g., Contains the main component for this feature).
- `path/to/file1.test.ts` - Unit tests for `file1.ts`.
- `path/to/another/file.tsx` - Brief description (e.g., API route handler for data submission).
- `path/to/another/file.test.tsx` - Unit tests for `another/file.tsx`.
- `lib/utils/helpers.ts` - Brief description (e.g., Utility functions needed for calculations).
- `lib/utils/helpers.test.ts` - Unit tests for `helpers.ts`.

### Notes

- Unit tests should typically be placed alongside the code files they are testing (e.g., `MyComponent.tsx` and `MyComponent.test.tsx` in the same directory).
- Use `npx jest [optional/path/to/test/file]` to run tests. Running without a path executes all tests found by the Jest configuration.
- Follow the project coding standards: every function needs a docstring/JSDoc with description, inputs, and outputs. Code must be commented and readable by a junior developer.
- **Task summaries** are saved to `tasks/task_summaries/` after each parent task's implementation sub-tasks are complete.
- **Code reviews** are saved to `tasks/code_reviews/` after each task summary.
- **Code cleanup** is performed after each code review — remove all temporary files, debug scripts, scratch notes, and anything not in the established project structure.
- All three steps are mandatory before moving to the next parent task.

## Instructions for Completing Tasks

**IMPORTANT:** As you complete each task, you must check it off in this markdown file by changing `- [ ]` to `- [x]`. This helps track progress and ensures you don't skip any steps.

Example:
- `- [ ] 1.1 Read file` → `- [x] 1.1 Read file` (after completing)

Update the file after completing each sub-task, not just after completing an entire parent task.

**IMPORTANT:** Every parent task ends with three mandatory steps:
1. **Write task summary** → save to `tasks/task_summaries/`. See the Task Summary Documents section in `generate-tasks.md` for the required format.
2. **Code review** → walk through the code with the user, then save the review document to `tasks/code_reviews/`. See the Code Review Documents section in `generate-tasks.md` for the required format.
3. **Code cleanup** → remove all temporary files, debug scripts, scratch markdown, backup copies, and any files not in the established project structure. Present the list to the user for confirmation before deleting. See the Code Cleanup section in `generate-tasks.md`.

Do not start the next parent task until all three are complete.


## Tasks

- [ ] 0.0 Create feature branch
  - [ ] 0.1 Create and checkout a new branch for this feature (e.g., `git checkout -b feature/[feature-name]`)
- [ ] 1.0 Parent Task Title
  - [ ] 1.1 [Sub-task description]
  - [ ] 1.2 [Sub-task description]
  - [ ] 1.3 **Write task summary** → save to `tasks/task_summaries/summary-[feature]-task-1.md`
  - [ ] 1.4 **Code review** → save to `tasks/code_reviews/review-[feature]-task-1.md`
  - [ ] 1.5 **Code cleanup** → remove temp files, verify clean directory structure
- [ ] 2.0 Parent Task Title
  - [ ] 2.1 [Sub-task description]
  - [ ] 2.2 **Write task summary** → save to `tasks/task_summaries/summary-[feature]-task-2.md`
  - [ ] 2.3 **Code review** → save to `tasks/code_reviews/review-[feature]-task-2.md`
  - [ ] 2.4 **Code cleanup** → remove temp files, verify clean directory structure
- [ ] 3.0 Parent Task Title (may not require sub-tasks if purely structural or configuration)
  - [ ] 3.1 **Write task summary** → save to `tasks/task_summaries/summary-[feature]-task-3.md`
  - [ ] 3.2 **Code review** → save to `tasks/code_reviews/review-[feature]-task-3.md`
  - [ ] 3.3 **Code cleanup** → remove temp files, verify clean directory structure
```

## Interaction Model

The process explicitly requires a pause after generating parent tasks to get user confirmation ("Go") before proceeding to generate the detailed sub-tasks. This ensures the high-level plan aligns with user expectations before diving into details.

## Target Audience

Assume the primary reader of the task lists is a **junior developer** who will implement the feature. Tasks should be explicit, unambiguous, and reference specific file paths and function names where possible.
