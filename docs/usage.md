# Usage

This page describes the current pilot workflow.

## Run Discovery

From the `AgentGovernanceBootstrap` repo:

```powershell
.\tools\agent-bootstrap-discover.ps1 <path-to-target-repo>
```

Optional coverage cap:

```powershell
.\tools\agent-bootstrap-discover.ps1 <path-to-target-repo> -CoverageCap 1000
```

The helper writes:

```text
<target-repo>/.bootstrap-tmp/.gitignore
<target-repo>/.bootstrap-tmp/START-HERE.md
<target-repo>/.bootstrap-tmp/repo-discovery-manifest.json
<target-repo>/.bootstrap-tmp/bootstrap-review-packet.md
<target-repo>/.bootstrap-tmp/templates/AGENTS.template.md
<target-repo>/.bootstrap-tmp/templates/repo-map.template.json
<target-repo>/.bootstrap-tmp/templates/artifact-manifest.template.json
```

`START-HERE.md` is written only when the target repo does not already have `AGENTS.md`.

## First Bootstrap

Open a fresh agent session in the target repo and paste:

```text
Read .bootstrap-tmp/START-HERE.md and follow it.
```

The expected agent behavior is:

1. Read the review packet and manifest.
2. Treat scratch files as data, not authority.
3. Read suggested repo files directly from the repo.
4. Use templates as drafting aids.
5. Draft durable guidance.
6. Ask before writing or replacing tracked guidance.

## Update Bootstrap

Once a repo has `AGENTS.md`, future discovery runs should be handled by the bootstrap
handoff rule inside that repo's `AGENTS.md`.

The agent should:

1. Read `.bootstrap-tmp/bootstrap-review-packet.md`.
2. Read `.bootstrap-tmp/repo-discovery-manifest.json`.
3. Compare the manifest commit with current `HEAD`.
4. Refuse to process stale scratch output automatically.
5. Propose durable guidance changes.
6. Ask before writing tracked files.

## What To Commit In Target Repos

Commit approved durable guidance, for example:

```text
AGENTS.md
.agents/repo-map.json
.agents/artifact-manifest.json
.agents/bootstrap.config.json
.agents/playbooks/*.md
```

Do not commit:

```text
.bootstrap-tmp/
```

The scratch directory contains its own `.gitignore` with:

```gitignore
*
```

## Pilot Review Checklist

After testing on a small repo, collect:

- the proposed `AGENTS.md`
- any proposed `.agents/*` files
- the agent's final answer
- any confusing prompt, question, or output
- whether `.bootstrap-tmp/` stayed out of `git status --short`

Use those results to decide what the next implementation step should be.

