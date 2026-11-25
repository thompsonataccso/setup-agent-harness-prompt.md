# Setup Agent Harness Prompt

## What It Is

A **meta-prompt** — a self-contained instruction set you copy and paste into an AI coding agent or CLI, including GitHub Copilot, Claude Code, Codex, Cursor, Gemini CLI, Cline, Roo Code, Windsurf, Continue, Aider, Amazon Q Developer, Amp, Zed, JetBrains AI Assistant, and OpenHands. It bootstraps the entire D5 agentic workflow harness for a **brand-new project** (you describe it) or an **existing codebase** (the agent scans it).

## What It Does

When pasted into a project, it instructs the AI agent to:

1. **Ask setup questions** — setup mode (new or existing project), AI tool (GitHub Copilot, Claude Code, Codex, Cursor, Gemini CLI, Cline, Roo Code, Windsurf, Continue, Aider, Amazon Q Developer CLI, Amp, Zed, JetBrains AI Assistant, OpenHands, or another tool), harness sharing (committed or local-only), project name, ticket prefix (e.g., `PROJ-`), output location, and an optional ASD-STE100 writing style
2. **Interview you for a project brief** *(new projects only)* — purpose, MVP scope, stack, architecture, data, integrations, auth, hosting, testing, conventions, constraints
3. **Scaffold the entire file structure**, including:
   - behavioral guidelines — `agents.md` for Copilot/Claude; otherwise included in the selected platform's native instructions file
   - the instructions file (for example, `.github/copilot-instructions.md`, `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.continue/rules/d5-workflow.md`, `CONVENTIONS.md`, or `.amazonq/rules/d5-workflow.md`, see [Supported AI Tools](#supported-ai-tools)) — the full D5 workflow (Define → Discover → Deliver → Demonstrate → Document) with phase gates and stop points
   - `memorybank/` folder with tracking files (`ticket-progress.md`, `changelog.md`, `architecture.md`, `features.md`, `integrations.md`, `setup.md`, `project-context.md`)
   - *(new projects)* `memorybank/project-brief.md` — the brief, open questions, and a decision log
   - `memorybank/components/` — per-feature documentation with a template
   - `<skills dir>/generate-project-context/` — a skill that populates `project-context.md` from the actual codebase (or from the brief when there is no code yet)
   - *(optional)* the ASD-STE100 writing-style skill (see [below](#optional-asd-ste100-writing-style))
4. **Update the README** with a section explaining the workflow commands
5. **Follow up by mode** — generate component docs from the code (existing) or propose an initial ticket backlog (new)
6. **Remind you of next steps** — run `generate project context` and use `StartTask` / `ReviewTasks`

## Setup Modes: New vs. Existing Project

The first setup question picks the mode. The agent proposes a default based on whether the folder already has source code or package manifests.

| | `new` — start a new project | `existing` — add the harness to a codebase |
|---|---|---|
| Source of project knowledge | **You** — a project-brief interview (or paste a spec/PRD) | **The code** — the agent scans it |
| Extra file | `memorybank/project-brief.md` (scope, decisions, open questions, decision log) | — |
| Memorybank docs | Pre-filled from the brief, items marked **Planned** | Placeholders, filled by scanning the code |
| `project-context.md` | Stack + conventions from the brief (`status: 'from_brief'`) | Generated from the code with `generate project context` |
| Existing files | — | Never overwritten — the agent proposes a merge and asks first |
| Follow-up | Proposes a backlog: `PREFIX-001` = project scaffolding, then one ticket per MVP feature (status `🗂️ Backlog`) | Offers to generate component docs and project context from the code |
| Code written during setup | None — every ticket goes through D5 | None |

**New project lifecycle:** brief → backlog → `StartTask PREFIX-001` (scaffolding) → `generate project context` again, so that the rules come from the real code instead of the brief → build the features ticket by ticket. While tasks run, the agent reads the brief in Discover and keeps it up to date in Document (open questions resolved, decisions logged).

## Supported AI Tools

The second setup question picks one tool or a supported pair. The workflow content is the same for every tool; entry points and file locations vary:

| Choice | Workflow instructions | Skills |
|--------|-----------------------|--------|
| `copilot` | `.github/copilot-instructions.md` | `.github/skills/` |
| `claude` | `CLAUDE.md` | `.claude/skills/` |
| `codex` | `AGENTS.md` | `.agents/skills/` |
| `cursor` | `AGENTS.md` | `.agents/skills/` |
| [`gemini`](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/gemini-md.md) | `GEMINI.md` | `.agents/skills/` |
| [`cline`](https://docs.cline.bot/customization) | `AGENTS.md` | `.cline/skills/` |
| [`roo`](https://github.com/RooCodeInc/Roo-Code-Docs/blob/main/docs/features/custom-instructions.md) | `AGENTS.md` | `.agents/skills/` |
| [`windsurf`](https://docs.windsurf.com/windsurf/cascade/agents-md) | `AGENTS.md` | `.agents/skills/` |
| [`continue`](https://docs.continue.dev/customize/rules) | `.continue/rules/d5-workflow.md` | `.agents/skills/` (explicitly read by the rule) |
| [`aider`](https://aider.chat/docs/config/aider_conf.html) | `CONVENTIONS.md` + `.aider.conf.yml` | `.agents/skills/` (explicitly read by the instructions) |
| [`amazonq`](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-project-rules.html) | `.amazonq/rules/d5-workflow.md` | `.agents/skills/` (explicitly read by the rule) |
| [`amp`](https://ampcode.com/docs/markdown/customize/agents-md) | `AGENTS.md` | `.agents/skills/` |
| [`zed`](https://zed.dev/docs/ai/instructions) | `AGENTS.md` | `.agents/skills/` (manual skill library) |
| [`jetbrains`](https://www.jetbrains.com/help/ai-assistant/configure-project-rules.html) | `.aiassistant/rules/d5-workflow.md` | `.agents/skills/` (manual skill library) |
| [`openhands`](https://docs.openhands.dev/overview/skills/repo) | `AGENTS.md` | `.agents/skills/` (manual skill library) |
| `both` | `.github/copilot-instructions.md` + `CLAUDE.md` | `.claude/skills/` |
| `codex-cursor` | `AGENTS.md` | `.agents/skills/` |
| `other` | Confirm the native path from official docs | Confirm native skill support or use `.agents/skills/` as a manual library |

> Claude Code does not read `agents.md` when a `CLAUDE.md` exists, so `CLAUDE.md` imports it with `@agents.md`.
> Codex, Cursor, Cline, Roo Code, Windsurf, Amp, Zed, and OpenHands read root `AGENTS.md`. Gemini CLI uses `GEMINI.md`; Continue, Aider, Amazon Q Developer CLI, and JetBrains AI Assistant use their native rule/configuration files.
> For tools that use `.agents/skills` or `.cline/skills`, skills are discoverable natively. For Continue, Aider, Amazon Q Developer CLI, Zed, JetBrains AI Assistant, and OpenHands, the instructions explicitly point to the skill file rather than relying on automatic discovery.

## Harness Sharing: Committed vs. Local-Only

The third setup question decides whether the harness is part of the repository. Use `local` when teammates work with other workflows and you do not want to push the D5 files on them.

| | `shared` (default) | `local` |
|---|---|---|
| Harness files | Committed to the repo — the whole team uses D5 | Listed in `.git/info/exclude` — **never committed**, exist only on your machine |
| Memorybank and branches | Committed, so it follows each branch | Not tracked, so it does **not** change when you switch branches |
| Document phase | Updates the memorybank docs directly on the branch → `✅ Done` | Writes **Pending Documentation Updates** into the task file → `🚢 Awaiting Merge` |
| Closing a task | Automatic at the end of Document | **`MergeMemorybankTask PREFIX-XXXX`** after the branch is merged into the integration branch |
| Repository `README.md` | Gets a harness section | Untouched — the section goes to `memorybank/README.md` |
| Already-tracked files (e.g., a team `CLAUDE.md`) | Merged after your approval | Never modified — the agent uses a local alternative where the platform supports one (for example, `CLAUDE.local.md`, `AGENTS.override.md`, `.cursor/rules/d5-workflow.mdc`, `.rules`, or `.continue/rules/d5-workflow.local.md`) |
| ASD-STE100 Claude plugin | `--scope project` | `--scope local` |

**Why the merge step exists (local mode):** the local memorybank describes the integration branch (e.g., `main` or `dev`). If an unmerged feature branch wrote straight into the baseline docs, they would describe code that the integration branch does not have yet. Instead, each task keeps its doc changes in the task file. After the merge, `MergeMemorybankTask` does the following:

1. Checks with a read-only git query that the branch is merged.
2. Applies the pending updates to the baseline docs.
3. Runs an alignment check across `project-context.md`, `features.md` and the component docs.
4. Moves the task to `completedTasks/`.

The agent never runs state-changing git commands in this mode. At the end of setup, it checks that `git status` shows none of the harness files.

> Teammates who don't use the harness also change the code. When Discover finds that a baseline doc disagrees with the code, the agent trusts the code, logs the drift in `memorybank/changelog.md`, and proposes a doc update.

## How To Use It

1. Open your selected AI coding tool in your **target project**
2. Copy everything below the `---` separator in the file
3. Paste it as a prompt
4. Answer the 7 setup questions (setup mode, AI tool, harness sharing, project name, ticket prefix, output folder, ASD-STE100 writing style)
   - **New project:** also answer the project-brief interview, or paste an existing spec/PRD
   - **Local-only:** also name the integration branch (e.g., `main`, `dev`)
5. Let the agent create all the files
6. **Existing project:** say `"generate project context"` to populate rules from your actual codebase.
   **New project:** approve the proposed backlog. Run `"generate project context"` again after `PREFIX-001` (scaffolding) is done.
7. Start using `StartTask PREFIX-001` for your first task

## What You Get After Setup

| Command | Description |
|---------|-------------|
| `StartTask PREFIX-XXXX` | Start a new task through the D5 phases |
| `ReviewTasks` | Review all incomplete tasks and resume where you left off |
| `MergeMemorybankTask PREFIX-XXXX` | *Local-only mode:* after the branch is merged, apply the task's pending doc updates to the memorybank and archive the task |

## The D5 Workflow (installed by this prompt)

| Phase | Purpose | Human Gate? |
|-------|---------|-------------|
| **Define** | Capture goal, constraints, definition of done | — |
| **Discover** | Investigate codebase, research best practices, write findings | — |
| **Deliver** | Plan changes → get approval → implement with verify loop → review the diff and fix findings | 🛑 Approval required before coding |
| **Demonstrate** | Run tests, prove the fix works | 🛑 Human verification required |
| **Document** | Update memorybank docs, move task to completed (*local-only:* record pending doc updates, then `MergeMemorybankTask` after merge) | — |

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