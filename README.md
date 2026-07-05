# Setup Agent Harness Prompt

## What It Is

A **meta-prompt** — a self-contained instruction set you copy and paste into an AI agent (like Copilot Chat) in a **different project** to bootstrap the entire D5 agentic workflow harness from scratch. Think of it as a portable installer for a structured AI-assisted development workflow.

## What It Does

When pasted into a new project, it instructs the AI agent to:

1. **Ask setup questions** — project name, ticket prefix (e.g., `PROJ-`), output location
2. **Scaffold the entire file structure**, including:
   - `agents.md` — behavioral guidelines (think first, simplicity, surgical changes)
   - `.github/copilot-instructions.md` — the full D5 workflow (Define → Discover → Deliver → Demonstrate → Document) with phase gates and stop points
   - `memorybank/` folder with tracking files (`ticket-progress.md`, `changelog.md`, `architecture.md`, `features.md`, `integrations.md`, `setup.md`, `project-context.md`)
   - `memorybank/components/` — per-feature documentation with a template
   - `.github/skills/generate-project-context/` — a skill that populates `project-context.md` from the actual codebase
3. **Update the README** with a section explaining the workflow commands
4. **Offer to generate component docs** by scanning the codebase for features
5. **Remind you of next steps** — run `generate project context` and use `StartTask` / `ReviewTasks`

## How To Use It

1. Open Copilot Chat (or any AI agent) in your **target project**
2. Copy everything below the `---` separator in the file
3. Paste it as a prompt
4. Answer the 3 setup questions (project name, ticket prefix, output folder)
5. Let the agent create all the files
6. Say `"generate project context"` to populate rules from your actual codebase
7. Start using `StartTask PREFIX-001` for your first task

## What You Get After Setup

| Command | Description |
|---------|-------------|
| `StartTask PREFIX-XXXX` | Start a new task through the D5 phases |
| `ReviewTasks` | Review all incomplete tasks and resume where you left off |

## The D5 Workflow (installed by this prompt)

| Phase | Purpose | Human Gate? |
|-------|---------|-------------|
| **Define** | Capture goal, constraints, definition of done | — |
| **Discover** | Investigate codebase, research best practices, write findings | — |
| **Deliver** | Plan changes → get approval → implement with verify loop | 🛑 Approval required before coding |
| **Demonstrate** | Run tests, prove the fix works | 🛑 Human verification required |
| **Document** | Update memorybank docs, move task to completed | — |