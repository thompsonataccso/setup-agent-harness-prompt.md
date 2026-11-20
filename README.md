# Setup Agent Harness Prompt

## What It Is

A **meta-prompt** — a self-contained instruction set you copy and paste into an AI agent (GitHub Copilot or Claude Code) in a **different project** to bootstrap the entire D5 agentic workflow harness from scratch. Think of it as a portable installer for a structured AI-assisted development workflow.

## What It Does

When pasted into a new project, it instructs the AI agent to:

1. **Ask setup questions** — AI tool (GitHub Copilot, Claude Code, or both), project name, ticket prefix (e.g., `PROJ-`), output location, and an optional ASD-STE100 writing style
2. **Scaffold the entire file structure**, including:
   - `agents.md` — behavioral guidelines (think first, simplicity, surgical changes)
   - the instructions file (`.github/copilot-instructions.md` or `CLAUDE.md`, see [Supported AI Tools](#supported-ai-tools)) — the full D5 workflow (Define → Discover → Deliver → Demonstrate → Document) with phase gates and stop points
   - `memorybank/` folder with tracking files (`ticket-progress.md`, `changelog.md`, `architecture.md`, `features.md`, `integrations.md`, `setup.md`, `project-context.md`)
   - `memorybank/components/` — per-feature documentation with a template
   - `<skills dir>/generate-project-context/` — a skill that populates `project-context.md` from the actual codebase
   - *(optional)* the ASD-STE100 writing-style skill (see [below](#optional-asd-ste100-writing-style))
3. **Update the README** with a section explaining the workflow commands
4. **Offer to generate component docs** by scanning the codebase for features
5. **Remind you of next steps** — run `generate project context` and use `StartTask` / `ReviewTasks`

## Supported AI Tools

The first setup question picks the tool. The workflow content is the same for every tool. Only the file locations change:

| | `copilot` | `claude` | `both` |
|---|-----------|----------|--------|
| Workflow instructions | `.github/copilot-instructions.md` | `CLAUDE.md` | `.github/copilot-instructions.md` |
| Claude Code entry point | — | `CLAUDE.md` (imports `@agents.md`) | `CLAUDE.md` (imports `@agents.md` and `@.github/copilot-instructions.md`) |
| Skills | `.github/skills/` | `.claude/skills/` | `.claude/skills/` (Copilot reads it too) |
| ASD-STE100 (optional) | skill file | Claude Code plugin (or skill file) | skill file |

> Claude Code does not read `agents.md` when a `CLAUDE.md` exists, so `CLAUDE.md` imports it with `@agents.md`.

## How To Use It

1. Open GitHub Copilot Chat or Claude Code in your **target project**
2. Copy everything below the `---` separator in the file
3. Paste it as a prompt
4. Answer the 5 setup questions (AI tool, project name, ticket prefix, output folder, ASD-STE100 writing style)
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
| **Deliver** | Plan changes → get approval → implement with verify loop → review the diff and fix findings | 🛑 Approval required before coding |
| **Demonstrate** | Run tests, prove the fix works | 🛑 Human verification required |
| **Document** | Update memorybank docs, move task to completed | — |

### The Review Step (Deliver)

After implementation is green, the agent reviews the **full diff** before handing it to you for verification. It works best with a fresh reviewer, such as a sub-agent or a different model. The review checks:

- plan conformance and Definition of Done
- correctness and edge cases
- security
- `project-context.md` rules and `agents.md` guidelines
- test coverage and hygiene

Findings are recorded in the task file as 🔴 blocking or 🟡 non-blocking. The agent fixes blocking findings in a review-fix loop. It does not apply non-blocking suggestions without your approval; instead, it lists them for you in Demonstrate. The ticket status during this step is `🔎 In Review`.

## Optional: ASD-STE100 Writing Style

[ASD-STE100](https://www.asd-ste100.org/) (Simplified Technical English) is a controlled-language standard. It was created for aerospace maintenance manuals, where a misunderstood sentence can cause an accident. Its rules:

- short sentences with one idea each
- active voice and imperative steps
- a small dictionary of simple words with one meaning each (*use*, not *utilize*)
- no more than 3 nouns in a row
- *must* / *can* instead of vague modals like *should* and *might*

Andrej Karpathy suggested asking LLMs to write in ASD-STE100 (or "80% of the way" to it) because the output is much easier to read. This matters as more of your work becomes reviewing AI output.

If you enable it in the setup wizard, the prompt:

1. Installs the `asd-ste100` skill from [JAICHANGPARK/ASD-STE100](https://github.com/JAICHANGPARK/ASD-STE100) (MIT) into the skills folder.
   - With **Claude Code**, it recommends the full plugin instead. The plugin adds the `/asd` command, an STE side pane, and lint/rewrite tools, and needs Claude Code v2.1.287+. It is installed with `--scope project` so the whole team gets it.
2. Adds a **Writing Style** section to the instructions file. This section makes the agent write task files, plans, review findings, and memorybank docs in STE. Code, identifiers, commands, and tool output are not affected.
3. Adds an STE check to the Deliver **REVIEW** step.

| Mode | Use for | Rules |
|------|---------|-------|
| `80` — 80% Pragmatic (recommended) | Software projects | STE rules, 15–22 word sentences, modern tech terms (API, cache, Docker) allowed |
| `strict` — 100% Issue 9 | Regulated / safety-critical docs | Full standard: ≤ 20-word instructions, ≤ 25-word descriptions, no contractions or Latin abbreviations |

> For GitHub Copilot, only the portable agent skill is installed. The `/asd` command, STE pane, and lint tools work only in Claude Code.