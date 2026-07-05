# Agent Harness Setup Prompt

> Copy everything below the line and paste it into Copilot Chat (or any AI agent) in your target project.

---

## Prompt

You are setting up a structured agentic workflow harness for this project. Follow these steps exactly:

### Step 1 — Gather Information

Ask me the following questions before creating any files:

1. **Project name** — What should I call this project in the instructions? (Default: folder name)
2. **Ticket prefix** — What prefix do your tickets use? (e.g., JIRA, PROJ, MWD, TASK)
3. **Output location** — Should `project-context.md` live in `memorybank/` (default) or somewhere else (e.g., root, `docs/`)?

Wait for my answers before proceeding.

### Step 2 — Create the File Structure

Once I answer, create the following files. Replace `{PROJECT}` with the project name, `{PREFIX}` with the ticket prefix, and `{DATE}` with today's date.

---

#### File: `agents.md` (project root)

```markdown
# Agent Behavioral Guidelines

Behavioral rules to reduce common LLM coding mistakes. These apply to **all** agent interactions in this repository, alongside the D5 workflow and project-context rules.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks (simple typo fixes, obvious one-liners), use judgment — not every change needs full rigor.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

---

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

The test: Would a senior engineer say this is overcomplicated? If yes, simplify.

---

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

---

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

## How to Know These Are Working

- Fewer unnecessary changes in diffs — only requested changes appear
- Fewer rewrites due to overcomplication — code is simple the first time
- Clarifying questions come before implementation — not after mistakes
- Clean, minimal PRs — no drive-by refactoring or "improvements"
```

---

#### File: `.github/copilot-instructions.md`

```markdown
# {PROJECT} - GitHub Copilot Instructions

## Prerequisites — Always Load Project Context

Before **any** planning, discovery, or implementation work, read `memorybank/project-context.md` in full. That file contains the authoritative rules for tech stack, coding conventions, testing, and critical anti-patterns. Follow those rules exactly — they take precedence over generic best practices.

---

## D5 Agentic Workflow

> ⚠️ **CRITICAL — FOLLOW THE WORKFLOW EXACTLY. NO SHORTCUTS. NO SKIPPING STEPS.**
> The D5 workflow exists for a reason. Every phase, every checkpoint, every stop point is intentional. You MUST:
> - Complete each phase **fully and in order** before moving to the next.
> - **Never skip** creating the task file or adding the ticket to `memorybank/ticket-progress.md`.
> - **Never make code changes** before receiving explicit human approval in the Deliver phase.
> - **Never proceed past a 🛑 STOP checkpoint** without explicit human confirmation.
> - If in doubt, **stop and ask** — do not assume approval or take initiative to "save time."
>
> Violating any of these rules breaks trust and invalidates the workflow.

All development tasks **must** follow the D5 workflow. No code changes should be made without completing each phase in order.

### StartTask Command

When the user says **"StartTask {PREFIX}-XXXX"** (or similar), initiate the D5 workflow:

> 💡 **Model Hint:** Use a **thinking/reasoning model** (e.g., Claude Opus, o3) for the **Define** and **Discover** phases — these require deep reasoning and thorough investigation. Once the implementation plan is **approved**, switch to a **faster implementation model** (e.g., Claude Sonnet, GPT-4o) for the **Deliver** grunt work.

1. Ask the user to provide the ticket details or describe the task.
2. Walk through the **Define** phase by asking:
   - **Goal**: What is the expected outcome?
   - **Constraints**: Are there technical, time, or scope constraints?
   - **Definition of Done**: What conditions must be met to consider this complete?
3. Create a task tracking file at `memorybank/activeTasks/{PREFIX}-XXXX.md` using the template below.
4. Add an entry to `memorybank/ticket-progress.md` with status `🔍 Discovery`.
5. Proceed through the remaining D5 phases.

### ReviewTasks Command

When the user says **"ReviewTasks"** (or similar), review all in-progress tasks:

1. Read `memorybank/ticket-progress.md` and identify all tasks that are **not** `✅ Done`.
2. For each incomplete task, read its task file at `memorybank/activeTasks/{PREFIX}-XXXX.md`.
3. Present a **summary table** showing: Ticket ID, Title, Current status/phase, What's blocking or next.
4. Ask the user which task(s) to **resume**, **update**, or **close**.
5. If resuming, pick up the D5 workflow from the phase where it was left off.

### The D5 Phases

#### 1. Define — Goal, Constraints, and Definition of Done

- Capture the ticket details ({PREFIX}-XXXX).
- Ask clarifying questions to establish:
  - **Goal**: What problem are we solving?
  - **Constraints**: Performance, compatibility, scope limitations.
  - **Definition of Done**: Specific acceptance criteria.
- Record all of this in the task tracking file.
- Update `memorybank/ticket-progress.md` status to `🔍 Discovery`.

#### 2. Discover

- **First:** Read `memorybank/project-context.md` if not already loaded this session.
- **Read component docs:** Check `memorybank/components/` for any existing documentation on the feature being changed. If a component doc exists, read it to understand current business rules, data flow, and validation before investigating the code.
- Investigate the codebase **thoroughly** to understand the root cause or area of change.
- **Scan all layers** — do not skip files. Examine all relevant source directories, configurations, tests, and shared modules.
- Read relevant files, search for symbols, trace the full flow end-to-end.
- **Search externally** for improvements, best practices, and package updates:
  - Check official documentation for updated guidance.
  - Check package registries for newer versions.
  - **Always prefer official docs** over third-party blog posts.
  - Note any relevant deprecations, new APIs, or recommended patterns.

**📝 Progressive Updates — Write findings to the task file as you go:**
- **Update `{PREFIX}-XXXX.md` incrementally** after each discovery step — do NOT wait until the end.
- After scanning each layer, immediately append findings to the Discover section.
- The task file serves as a **resumable checkpoint** — the user can stop and continue later.
- By the end of Discover, the task file should contain a **complete picture**: all affected files, root cause analysis, external research, and a draft of the implementation plan.
- Update `memorybank/ticket-progress.md` status to `📋 Awaiting Approval`.
- **Do NOT edit any code in this phase.**

#### 3. Deliver — The Smallest Possible Change

> ⚠️ **HARD RULE — NO EXCEPTIONS**: The Deliver phase is split into **PLAN** and **IMPLEMENT**. You MUST complete PLAN and receive explicit human approval BEFORE writing any code. Under NO circumstances should you modify any source file until the user explicitly approves.

**Step 1 — PLAN (no code changes allowed):**
- Plan the minimal set of changes required.
- Document planned changes in the task file under **Implementation Plan**.
- **🛑 STOP — Human Approval Required.**
  - Present the plan to the user.
  - **Do NOT proceed until the user explicitly approves.**

**Step 2 — IMPLEMENT (only after explicit approval):**
- Update `memorybank/ticket-progress.md` status to `🚧 In Progress`.
- **Follow the implement-verify loop:**
  1. **Act:** Make the next planned change (smallest coherent unit).
  2. **Verify:** Run the build and relevant tests after each change.
  3. **Keep or fix:** If tests pass, move on. If tests fail, fix before continuing.
  4. **Stop when:** Plan is complete and green, OR progress stalls twice on the same item (escalate), OR scope change needed (request approval).
- **Never** change existing patterns or architecture without explicit approval.
- **Never** over-engineer. Prefer the simplest solution.
- **Never** fold unrelated fixes into the patch — log those in `memorybank/changelog.md`.
- If tests do not exist for the affected area, write them.

#### 4. Demonstrate — Prove the Fix with Test Output

- Update `memorybank/ticket-progress.md` status to `🧪 Testing`.
- Run existing tests to confirm nothing is broken.
- Run new/updated tests to confirm the fix works.
- Record test output in the task file under **Test Results**.
- Build must pass.
- **🛑 STOP — Human Verification Required.**
  - Present test results to the user.
  - Ask the user to manually test and confirm.
  - **Do NOT proceed until the user explicitly confirms.**
  - **Feedback repair loop:** If issues reported:
    1. Diagnose the issue.
    2. Apply smallest fix.
    3. Re-run build + tests.
    4. Present updated results. Repeat until confirmed.

#### 5. Document — Prepare the Change for Review

- Update `memorybank/ticket-progress.md` status to `📝 Documenting`.
- Update the task file with a summary of all changes made.
- **Documentation sync loop** — for each memorybank doc file, check if the task's changes affected its domain. Update if yes, record "N/A" if no:
  - [ ] `memorybank/features.md` — new features, changed behavior
  - [ ] `memorybank/architecture.md` — new entities, services, modules, schema
  - [ ] `memorybank/integrations.md` — new external APIs, auth changes
  - [ ] `memorybank/setup.md` — new env vars, config changes
  - [ ] `memorybank/components/*.md` — update any affected component docs (business rules, data flow, validation, key files). If the task changes a feature that has no component doc yet, **create one** using the template in `memorybank/components/_index.md`. If an existing component doc's business rules, data flow, validation, or key files changed, **update it** to reflect the new reality.
- **Alignment check** — after updating docs, verify consistency across the three knowledge layers:
  1. `memorybank/project-context.md` — tech stack rules, coding conventions, anti-patterns
  2. `memorybank/features.md` — feature list and high-level behavior
  3. `memorybank/components/*.md` — detailed per-feature flows, business rules, validation
  
  If any of these contradict each other after the task's changes, reconcile them now. For example: if a component doc references a validation pattern that `project-context.md` now forbids, update the component doc. If `features.md` lists behavior that a component doc shows has changed, update `features.md`.
- If the task introduced architectural changes, run `generate-project-context` skill in update mode.
- Move task file from `memorybank/activeTasks/` to `memorybank/completedTasks/`.
- Update `memorybank/ticket-progress.md` status to `✅ Done`.
- Log any ad-hoc or out-of-scope changes in `memorybank/changelog.md`.

---

## Task Tracking File Template

When creating `memorybank/activeTasks/{PREFIX}-XXXX.md`, use this structure:

```
# {PREFIX}-XXXX: [Title]

## Define
- **Goal**:
- **Constraints**:
- **Definition of Done**:

## Discover
_(Updated progressively — each section filled in as discovery proceeds)_

### Findings

### Affected Files
| File | Role | Notes |
|------|------|-------|

### Root Cause / Area of Change

### External Research
- Relevant docs/updates:
- Package updates available:

### Discovery Summary

## Deliver
### Implementation Plan
- [ ] Change 1
- [ ] Change 2

### Approval
- **Status**: ⏳ Awaiting approval / ✅ Approved / ❌ Rejected
- **Approved by**:
- **Notes**:

### Changes Made
- File: `path/to/file` — Description of change

## Demonstrate
### Test Results
- Build: ✅ / ❌
- Tests: ✅ / ❌
- Output:

### Human Verification
- **Status**: ⏳ Awaiting / ✅ Confirmed / 🔄 Iteration needed
- **Feedback**:

## Document
- **Summary**:
- **Documentation Updated**:
```

---

## Tracking Files

### `memorybank/ticket-progress.md`
Tracks the **status of all tickets** across sessions. Allows planning one day and resuming the next.

### `memorybank/changelog.md`
Log of **ad-hoc changes** outside any task file — quick fixes, opportunistic improvements, config tweaks.
```

---

#### File: `memorybank/ticket-progress.md`

```markdown
# Ticket Progress

Track the status of all {PREFIX} tasks across sessions. Updated at **every phase transition** in the D5 workflow.

## Status Legend

| Emoji | Status | Meaning |
|-------|--------|---------|
| 🔍 | Discovery | Investigating the codebase |
| 📋 | Awaiting Approval | Implementation plan ready, waiting for human approval |
| 🚧 | In Progress | Approved and actively being implemented |
| 🧪 | Testing | Running tests and verifying the fix |
| 📝 | Documenting | Writing up changes and updating docs |
| ✅ | Done | Completed and moved to completedTasks/ |

## Tickets

| Ticket | Title | Status | Last Updated | Notes |
|--------|-------|--------|--------------|-------|
```

---

#### File: `memorybank/changelog.md`

```markdown
# Changelog — Ad-hoc Changes

Changes made outside the scope of any task file. Includes quick fixes, opportunistic improvements, or config tweaks noticed during other work.

| Date | Description | Files Changed | Related Task (if any) |
|------|-------------|---------------|-----------------------|
```

---

#### File: `memorybank/architecture.md`

```markdown
# Architecture

## Project Structure

_(Fill in after running generate-project-context or manually)_

## Key Modules

| Module | Responsibility |
|--------|---------------|

## Data Flow

_(Describe how data flows through the system)_
```

---

#### File: `memorybank/features.md`

```markdown
# Features

## Active Features

| Feature | Description | Status |
|---------|-------------|--------|
```

---

#### File: `memorybank/integrations.md`

```markdown
# Integrations

## External APIs

| Service | Purpose | Auth Method | Notes |
|---------|---------|-------------|-------|
```

---

#### File: `memorybank/setup.md`

```markdown
# Setup

## Prerequisites

_(List required tools, runtimes, and their versions)_

## Environment Variables

| Variable | Purpose | Required | Default |
|----------|---------|----------|---------|

## Getting Started

```bash
# Clone and install
```

## Running Tests

```bash
# ...
```
```

---

#### File: `memorybank/project-context.md`

```markdown
---
date: '{DATE}'
sections_completed: []
status: 'not_started'
---

# Project Context for AI Agents

_This file contains the unobvious rules and patterns AI agents must follow when implementing code in this project. It is not a tutorial or onboarding doc — it captures the things an agent would otherwise get wrong._

> **⚠️ This is a placeholder.** Say "generate project context" to populate this file with real rules derived from your codebase.

---

## Technology Stack & Versions

_Run generate-project-context to fill._

## Critical Implementation Rules

### Language-Specific Rules

_Run generate-project-context to fill._

### Framework-Specific Rules

_Run generate-project-context to fill._

### Testing Rules

_Run generate-project-context to fill._

### Code Quality & Style Rules

_Run generate-project-context to fill._

### Development Workflow Rules

_Run generate-project-context to fill._

### Critical "Don't-Miss" Rules

_Run generate-project-context to fill._

---

## Usage Guidelines

**For AI agents:** Read this file before implementing. When a rule applies, follow it exactly. When in doubt between this file and a generic best practice, prefer this file.

**For humans maintaining this file:** Keep it lean. Every rule consumes context budget on every future invocation. If a rule has become obvious from the code itself, remove it. Update when the tech stack or conventions change.

Last Updated: {DATE}
```

---

#### Directories to create (with `.gitkeep` files):

- `memorybank/activeTasks/.gitkeep`
- `memorybank/completedTasks/.gitkeep`
- `memorybank/components/.gitkeep`

---

#### File: `.github/skills/generate-project-context/SKILL.md`

```markdown
---
name: generate-project-context
description: 'Standalone skill that creates or updates project-context.md — a lean, LLM-optimized rules document that AI agents read before implementing code. It scans the project for tech stack, patterns, and conventions, then walks the user through seven rule categories one at a time. Use whenever the user says "generate project context", "create project context", "update project context", or "set up project rules for AI".'
---

Follow the instructions in ./workflow.md.
```

---

#### File: `.github/skills/generate-project-context/workflow.md`

```markdown
# Generate Project Context Workflow

This skill produces a single artifact: `project-context.md`, a lean rules document that AI agents read before implementing code.

## Initialization

1. **Find the project root.** Use `git rev-parse --show-toplevel` or current working directory.
2. **Choose output location.** Default: `memorybank/project-context.md`.
3. **Detect existing context.** If found, ask user: update or start fresh?
4. **Set the date.** Today's date in YYYY-MM-DD.
5. **Begin Step 1.**

## Step-execution rules

- One step at a time.
- Halt at every CHECKPOINT — wait for user input.
- Be a facilitator, not an oracle.
- Lean over comprehensive — only capture what agents would otherwise get wrong.

## Step 1 — Discover

Scan the project for:

1. **Tech-stack signals** — package manifests, runtimes, frameworks, versions
2. **Config signals** — linters, formatters, tsconfig, CI files, editor configs
3. **Source-tree signals** — naming conventions, layering, entry points
4. **Existing agent rules** — CLAUDE.md, AGENTS.md, .cursorrules

Build a structured discovery summary and present it to the user.

### CHECKPOINT — Wait for user confirmation before proceeding.

## Step 2 — Generate Rules (seven categories, one at a time)

For each category: draft 3–8 rules from discovery, show to user, wait for approval, save.

1. **Technology Stack & Versions** — exact versions that constrain implementation
2. **Language-Specific Rules** — unobvious patterns specific to the language(s)
3. **Framework-Specific Rules** — framework patterns agents could miss
4. **Testing Rules** — how tests are organized, what agents must do
5. **Code Quality & Style Rules** — things linters don't enforce but the team cares about
6. **Development Workflow Rules** — branch/commit/PR/deploy conventions
7. **Critical "Don't-Miss" Rules** — anti-patterns, security gotchas, hard-won lessons

## Step 3 — Finalize

- Review for duplicates, vagueness, contradictions
- Tighten for density (imperative voice, no hedges)
- Append usage footer with date
- Show final summary to user
```

---

#### Directory: `memorybank/components/`

Create this folder with a `.gitkeep` and an `_index.md` file:

#### File: `memorybank/components/_index.md`

```markdown
# Component Documentation

This folder contains detailed documentation for each feature/component in the system. Each file documents one logical component — its business rules, data flow, UI interactions, validation, and integration points.

## Purpose

When an agent needs to modify a feature, it reads the relevant component doc to understand:
- **What** the feature does (business rules, user-facing behavior)
- **How** data flows through the system (frontend → API → service → DB → external systems)
- **Where** the code lives (files, modules, layers involved)
- **Why** certain decisions were made (constraints, edge cases, gotchas)

## File Naming

Use kebab-case matching the feature name:
- `person-edit.md` — Editing a person record
- `duplicate-detection.md` — Finding and resolving duplicate entries
- `date-validation.md` — Date input rules and validation logic
- `authentication.md` — Auth flow across frontend and API

## Template

Each component doc should follow this structure:

---

# [Component/Feature Name]

## Overview
_(1–2 sentences: what this feature does from the user's perspective)_

## Business Rules
- Rule 1: ...
- Rule 2: ...
- Rule 3: ...

## Data Flow

```
[Trigger] → [Frontend Component] → [API Endpoint] → [Service] → [Repository/DB]
                                                   ↘ [External System]
```

### Step-by-step:
1. User does X in the UI
2. Frontend calls `POST /api/...`
3. Controller validates input, calls service
4. Service applies business logic
5. Repository persists changes
6. (Optional) External system is notified

## Validation Rules

| Field | Rule | Where Enforced | Error Message |
|-------|------|----------------|---------------|
| dateOfBirth | Must be in the past | API + Client | "Date must be before today" |

## Key Files

| File | Role |
|------|------|
| `src/Components/EditPerson.razor` | UI form |
| `src/Controllers/PersonController.cs` | API endpoint |
| `src/Services/PersonService.cs` | Business logic |

## Edge Cases & Gotchas
- _(Things that have caused bugs or confusion before)_

## Related Components
- [Link to related component doc]

---
```

---

### Step 3 — Update README

Add a section to the project's `README.md` (create one if it doesn't exist) that documents the agent harness. Include:

1. **Development Workflow (D5)** — brief description + table of commands (`StartTask {PREFIX}-XXXX`, `ReviewTasks`)
2. **Agent Configuration** — table linking to `.github/copilot-instructions.md`, `agents.md`, `memorybank/project-context.md`
3. **Documentation (Memory Bank)** — table of all memorybank files with descriptions
4. **Component Documentation** — table linking to each `memorybank/components/*.md` file (initially just the `_index.md` template reference)

Use this structure:

```markdown
## Development Workflow (D5)

All development follows the **D5 agentic workflow** with continuous progress tracking.

| Command | Description |
|---------|-------------|
| `StartTask {PREFIX}-XXXX` | Start a new task through the D5 phases |
| `ReviewTasks` | Review all incomplete tasks and resume where you left off |

### Agent Configuration

| File | Purpose |
|------|--------|
| `.github/copilot-instructions.md` | D5 workflow definition, task tracking, phase gates |
| `agents.md` | Behavioral guidelines — think first, simplicity, surgical changes |
| `memorybank/project-context.md` | Tech stack rules, coding conventions, anti-patterns |

### Documentation (Memory Bank)

| File | Contents |
|------|----------|
| `memorybank/architecture.md` | Solution structure, layers, key design decisions |
| `memorybank/features.md` | Feature list and high-level behavior |
| `memorybank/integrations.md` | External services and APIs |
| `memorybank/setup.md` | Local development setup |
| `memorybank/ticket-progress.md` | Status tracker for all tickets across sessions |
| `memorybank/changelog.md` | Ad-hoc changes outside task files |
| `memorybank/components/` | Per-feature documentation (business rules, data flow, validation) |
```

Do NOT overwrite existing README content — append the section at the end, or integrate it naturally if the README already has structure.

---

### Step 4 — Generate Component Docs

After creating all the scaffolding files, ask me:

> I've created the `memorybank/components/` folder with the template. Would you like me to scan the codebase now and generate component documentation for the major features I find?
>
> I'll identify distinct features/flows (e.g., CRUD operations, search, auth, data sync) and for each one, create a doc that maps:
> - Business rules
> - UI → API → Service → DB data flow
> - Validation rules
> - Key files involved
> - Edge cases
>
> Should I proceed? I'll present each component doc for your review before saving.

If the user says yes, scan the codebase for distinct features and generate one component doc at a time, presenting each for approval before saving. Work through them one by one — don't batch.

---

### Step 5 — Confirm Completion

After creating all files, show me:
1. A tree of all files created
2. Remind me to say **"generate project context"** to populate the rules from my actual codebase
3. Remind me to say **"document components"** to generate/update component documentation
4. Remind me the workflow commands: `StartTask {PREFIX}-001` and `ReviewTasks`
