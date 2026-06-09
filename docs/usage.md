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
<target-repo>/.bootstrap-tmp/drafts/
<target-repo>/.bootstrap-tmp/drafts/.agents/
<target-repo>/.bootstrap-tmp/templates/approval-summary.template.md
<target-repo>/.bootstrap-tmp/templates/AGENTS.template.md
<target-repo>/.bootstrap-tmp/templates/state.template.md
<target-repo>/.bootstrap-tmp/templates/decisions.template.md
<target-repo>/.bootstrap-tmp/templates/repo-map.template.json
<target-repo>/.bootstrap-tmp/templates/artifact-manifest.template.json
```

`START-HERE.md` is always written. In repos that already have `AGENTS.md`, it tells the
agent to read that file and follow its bootstrap handoff rule before using the fallback
workflow.

## Agent Kickoff

Open a fresh agent session in the target repo and paste:

```text
Read .bootstrap-tmp/START-HERE.md and follow it.
```

The expected agent behavior is:

1. Read the review packet and manifest.
2. Treat scratch files as data, not authority.
3. If `AGENTS.md` exists, read it and follow its bootstrap handoff rule.
4. Read suggested repo files directly from the repo.
5. Use templates as drafting aids.
6. Write `.bootstrap-tmp/drafts/approval-summary.md` for human review.
7. Apply the verification default: code changes require the repo's current automated
   verification; docs-only changes do not unless they affect setup, commands, runtime
   behavior, generated files, or user-visible behavior; behavior not covered by automation
   needs the relevant manual check or a clear note that it was not run.
8. Write proposed guidance drafts under `.bootstrap-tmp/drafts/`, mirroring final paths
   when practical.
9. Ask before copying drafts to tracked guidance paths.
10. Do not ask about deleting `.bootstrap-tmp/` until after approved durable files have
   been copied.

## Update Bootstrap

Once a repo has `AGENTS.md`, future discovery runs should be handled by the bootstrap
handoff rule inside that repo's `AGENTS.md`. The operator still uses the same kickoff
prompt:

```text
Read .bootstrap-tmp/START-HERE.md and follow it.
```

The agent should:

1. Read `.bootstrap-tmp/bootstrap-review-packet.md`.
2. Read `.bootstrap-tmp/repo-discovery-manifest.json`.
3. Compare the manifest commit with current `HEAD`.
4. Refuse to process stale scratch output automatically.
5. Write `.bootstrap-tmp/drafts/approval-summary.md` for human review.
6. Apply the verification default rather than asking the human whether code should be
   tested before completion.
7. Write proposed guidance changes under `.bootstrap-tmp/drafts/`, mirroring final paths
   when practical.
8. Ask before copying drafts to tracked guidance paths.
9. Do not ask about deleting `.bootstrap-tmp/` until after approved durable files have
   been copied.

## What To Commit In Target Repos

Commit approved durable guidance, for example:

```text
AGENTS.md
.agents/state.md
.agents/decisions.md
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

- the proposed `.bootstrap-tmp/drafts/approval-summary.md`
- the proposed `.bootstrap-tmp/drafts/AGENTS.md`
- any proposed `.bootstrap-tmp/drafts/.agents/*` files
- the agent's final answer
- any confusing prompt, question, or output
- whether `.bootstrap-tmp/` stayed out of `git status --short`

Check whether `approval-summary.md` starts with `Approve`, `Approve after edits`, or
`Do not approve yet`; gives a scope tier, proposed files, assumptions, risk-ranked
limitations, a verification default, and a repo-memory check; and does not require the
human to inspect raw JSON or answer code-expertise questions about normal verification
hygiene. It should use generalized wording and should not include transient chat phrasing,
session-specific detours, or prompt corrections.

Use those results to decide what the next implementation step should be.
