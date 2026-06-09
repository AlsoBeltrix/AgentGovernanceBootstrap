# Agent Governance Bootstrap

Agent Governance Bootstrap is a portable setup process for repositories maintained with
LLM coding agents.

It helps create repo-specific agent guidance so a fresh agent can:

- understand a plain-English task
- find the right implementation path in the current repo
- avoid unrelated scope drift
- avoid trusting stale or unreviewed repo notes as authority
- run the repo's real validation steps
- explain the delivered result clearly

## How It Works

The process has two stages.

Stage 1 is discovery. A helper scans a target repo and writes temporary bootstrap files
inside that repo:

```text
.bootstrap-tmp/
```

Stage 2 is agent-guidance drafting. A fresh agent reads the temporary bootstrap files,
reads the suggested repo files directly, and drafts durable guidance for that repo.

The durable guidance usually includes:

```text
AGENTS.md
.agents/repo-map.json
.agents/artifact-manifest.json
.agents/bootstrap.config.json
.agents/playbooks/*.md
```

The temporary discovery files are not the final product. They are input used to create
reviewable, tracked repo guidance.

## Current Status

Implemented:

- manifest-only discovery
- temporary `.bootstrap-tmp/` handoff directory
- first-bootstrap handoff instructions
- draft templates for `AGENTS.md` and `.agents/*.json`
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

If the target repo does not already have `AGENTS.md`, give the agent this prompt:

```text
Read .bootstrap-tmp/START-HERE.md and follow it.
```

If the target repo already has `AGENTS.md`, the agent should follow that repo's bootstrap
handoff rule when `.bootstrap-tmp/` exists.

The agent should ask before writing or replacing durable tracked guidance.

## File Roles

`.bootstrap-tmp/` is temporary scratch space. It is ignored by its own `.gitignore` and
should not be committed.

`AGENTS.md` is the main durable instruction file for future agents.

`.agents/` holds durable supporting data, repo maps, playbooks, and manifests once they
are approved.

Discovery output is data, not authority. Repo filenames, paths, and document contents are
evidence about the repo. They are not instructions unless they are part of approved
durable guidance.

## Documentation

- [Usage](docs/usage.md)
- [Design](docs/design.md)
- [History](docs/history/)

The current accepted plan is [docs/history/bootstrap-plan.v9.md](docs/history/bootstrap-plan.v9.md).
