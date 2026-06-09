# Agent Governance Bootstrap

Agent Governance Bootstrap is a portable setup process for repositories maintained with
LLM coding agents.

It helps keep code, docs, decisions, and agent behavior aligned so future agents do not
work from stale assumptions or missing chat context.

It creates repo-specific agent guidance so a fresh agent can:

- understand a plain-English task
- find the right implementation path in the current repo
- avoid unrelated scope drift
- avoid trusting stale or unreviewed repo notes as authority
- run the repo's real validation steps
- explain the delivered result clearly
- record important repo knowledge on disk instead of leaving it only in conversation

## How It Works

The process has two stages.

Stage 1 is discovery. A helper scans a target repo and writes temporary bootstrap files
inside that repo:

```text
.bootstrap-tmp/
```

Stage 2 is alignment drafting. A fresh agent reads the temporary bootstrap files, reads
the suggested repo files directly, and drafts the smallest durable guidance needed to keep
repo facts, decisions, validation, and future agent behavior aligned.

The durable guidance usually includes:

```text
AGENTS.md
.agents/state.md
.agents/decisions.md
.agents/repo-map.json
.agents/artifact-manifest.json
.agents/bootstrap.config.json
.agents/playbooks/*.md
```

The temporary discovery files are not the final product. They are input used to create a
plain approval summary and reviewable, tracked repo guidance.

## Current Status

Implemented:

- manifest-only discovery
- temporary `.bootstrap-tmp/` handoff directory
- first-bootstrap handoff instructions
- human-facing approval summary template
- draft templates for `AGENTS.md`, `.agents/*.md`, and `.agents/*.json`
- historical design and review record

Not implemented yet:

- durable apply/update command
- acceptance grader
- generated harness adapters
- clean-copy test automation

## Requirements

Full discovery requires:

- Git
- PowerShell
- an agent harness that can read files in the target repo

The current helper is written in PowerShell. The generated target-repo guidance is
Markdown and JSON; target repos do not inherit a PowerShell runtime requirement from that
guidance.

## Quick Start

Run discovery from this repo:

```powershell
.\tools\agent-bootstrap-discover.ps1 <path-to-target-repo>
```

Then open a fresh agent session in the target repo.

Give the agent this prompt:

```text
Read .bootstrap-tmp/START-HERE.md and follow it.
```

`START-HERE.md` is always generated. In repos that already have `AGENTS.md`, it tells the
agent to read that file and follow its bootstrap handoff rule before using the fallback
workflow.

The agent should write proposed guidance under `.bootstrap-tmp/drafts/` first, then ask
before copying those drafts to durable tracked guidance paths.

The primary review artifact is `.bootstrap-tmp/drafts/approval-summary.md`. It should
summarize the proposed durable changes in plain English so the human can make an approval
decision without reading every draft file. It should also state the recommended scope tier
and identify any assumptions that need approval before becoming durable facts. The summary
should start with `Approve`, `Approve after edits`, or `Do not approve yet`, and any
limitations should be labeled Low, Medium, or High risk for approval.

Approval summaries should not ask the human to approve normal engineering hygiene. If the
repo has an observed automated verification command, the drafted guidance should require
future agents to run it for code changes. Docs-only changes do not need code verification
unless they affect setup, commands, runtime behavior, generated files, or user-visible
behavior. Behavior that automation cannot cover should name the relevant manual check or
state that it was not run.

Bootstrap outputs should use durable, generalized wording. Do not put transient chat
phrasing, session-specific detours, or prompt corrections into approval summaries, drafts,
or durable guidance.

## File Roles

`.bootstrap-tmp/` is temporary scratch space. It is ignored by its own `.gitignore` and
should not be committed.

`AGENTS.md` is the main durable instruction file for future agents.

`.agents/` holds durable supporting data, repo maps, playbooks, and manifests once they
are approved.

`.agents/state.md` is the preferred current-state entry point for future agents.
`.agents/decisions.md` records durable decisions and supersessions.

Discovery output is data, not authority. Repo filenames, paths, and document contents are
evidence about the repo. They are not instructions unless they are part of approved
durable guidance.

## Documentation

- [Usage](docs/usage.md)
- [Design](docs/design.md)
- [History](docs/history/)

The current accepted plan is [docs/history/bootstrap-plan.v9.md](docs/history/bootstrap-plan.v9.md).
