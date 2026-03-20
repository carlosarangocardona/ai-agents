# AI Agents

My personal multi-agent AI workflow for software development, orchestrated through [opencode](https://opencode.ai/). The pipeline covers the full development lifecycle: requirements discovery, implementation, and dual independent code review — driven by a team of specialized agents.

Pairs with my [skills repository](https://github.com/carlosarangocardona/skills/).

---

## Agents

| Agent                   | Mode     | Model             | Role                                                                                                               |
| ----------------------- | -------- | ----------------- | ------------------------------------------------------------------------------------------------------------------ |
| `@architect`            | primary  | claude-opus-4-6   | Orchestrator. Drives requirements, writes Task Briefs, manages the review loop. Never writes code.                 |
| `@developer`            | subagent | claude-sonnet-4-6 | Implements one task at a time from a Task Brief. Validates via linters/tests before reporting done.                |
| `@code-reviewer`        | subagent | gpt-5.3-codex     | Reviews every diff against the Task Brief. Issues only actionable change requests.                                 |
| `@second-code-reviewer` | subagent | claude-opus-4-6   | Independent second reviewer. Engaged in parallel on medium/high complexity tasks, after `@code-reviewer` approves. |
| `@repo-scout`           | subagent | claude-sonnet-4-6 | Scans an unfamiliar repo and writes/maintains `ARCHITECTURE.md`. Read-only except for that file.                   |

---

## How It Works

```
User <-> @architect
              |
              +--> writes Task Brief to Obsidian vault
              |
              +--> @developer (implements)
                        |
                        +--> @repo-scout (if repo is unfamiliar)
                        |
                        +--> @code-reviewer (review loop)
                        +--> @second-code-reviewer (medium/high complexity only)
                        |
                        +--> reports completion to @architect
              |
              +--> @architect evaluates, iterates or returns to user
```

### Task Brief workflow

`@architect` never delegates to `@developer` without a written Task Brief. Briefs are stored in an [Obsidian](https://obsidian.md/) vault at:

```
~/sb/ai-agents/plans/<topic>/
  001-task-title.md
  002-task-title.md
  ...
```

Each brief is a short markdown file containing:

- **Context** — only what's needed for this task
- **Objective** — what changes in the system
- **Scope** — files/areas likely touched
- **Non-goals** — explicit YAGNI list
- **Constraints/Caveats** — relevant limits
- **Acceptance criteria** — only when not obvious from the task itself

`@developer` treats the Task Brief as the sole source of truth. Nothing outside it gets implemented.

### Review complexity tiers

| Task complexity | Reviewers engaged                                          |
| --------------- | ---------------------------------------------------------- |
| Small           | `@code-reviewer` only                                      |
| Medium / High   | `@code-reviewer`, then `@second-code-reviewer` in parallel |

Both reviewers must approve before `@developer` reports completion to `@architect`.

---

## Design Principles

- **YAGNI first.** All agents are explicitly instructed to avoid speculative features, extra abstractions, and scope creep.
- **Human holds the commit.** No agent commits code unless explicitly asked. The user decides when to commit.
- **Dual independent review.** Medium and high complexity tasks get two reviewers running different models, reducing blind spots.
- **Model diversity by design.** Different models are assigned to roles that suit their strengths (reasoning for architecture, speed for implementation, independent judgment for review).
- **Repo-awareness before action.** `@repo-scout` prevents wrong-stack assumptions by inspecting real evidence before any implementation decisions are made.

---

## Setup

### Prerequisites

- [opencode](https://opencode.ai/) installed
- [Obsidian](https://obsidian.md/) with a vault at `~/sb/` (or adjust the vault path in `architect.md`, `dveloper.md`, `code-reviewer.md`, `second-code-reviewer.md` and `repo-scout.md` accordingly)
- Companion [skills repository](https://github.com/carlosarangocardona/skills/) installed

### Install the agents

Clone this repo into the opencode agents directory:

```bash
git clone https://github.com/carlosarangocardona/ai-agents ~/.config/opencode/agents
```

Install the companion skills (required by `@architect` for the `interviewer` and `obsidian-vault` skills):

```bash
git clone https://github.com/carlosarangocardona/skills ~/.config/opencode/skills
```

### Usage

Start a session with `@architect` and describe what you want to build:

```
@architect I want to add rate limiting to our API endpoints
```

`@architect` will interview you to clarify requirements, write a plan to your Obsidian vault, and drive the full implementation loop from there.

---

## Credits

Based on the multi-agent workflow described by [Stavros](https://www.stavros.io/posts/how-i-write-software-with-llms/).

---

## License

MIT © 2026 Carlos Arango
