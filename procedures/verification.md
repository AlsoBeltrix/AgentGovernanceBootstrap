# Fresh-Eyes Verification

Purpose: prove the drafted guidance is discoverable and internally consistent
for an agent with zero context - the exact situation it exists for. Run after
drafting, before the approval summary. Required for migration runs;
recommended for substantial greenfield runs.

Scope warning: this is a discoverability and consistency check, NOT a fact
check. It proves a zero-context agent can find the answers in the drafts and
the repo; it does not prove those answers are true. A draft that confidently
repeats a false claim about an external system (CI that never runs, a deploy
target that moved) will pass questions 1-5. Claims about external systems
must be validated under the evidence rule in `procedures/bootstrap.md`, and
the approval summary must not cite this test as proof of factual
correctness.

## How

1. Start a fresh agent context with no knowledge of this session (a subagent
   with a clean context, or a new session). Do not summarize the migration for
   it - that would defeat the test.
2. Give it only this prompt:

   "Read the draft guidance under .bootstrap-tmp/drafts/ in this repo as if those
   files were at their final paths (drafts/AGENTS.md is AGENTS.md, drafts/.agents/
   is .agents/). Using only those files plus the repo itself, answer: (1) What is
   this project? (2) What is true right now - active work, blockers? (3) What
   should happen next? (4) How are code changes verified before completion?
   (5) Which file would you update at the end of a work session, and how would
   you record a new durable decision? (6) For every claim in these files about
   CI, deployment, or another external system, name the repo evidence showing
   the claim is currently ACTIVE rather than merely present as a file - for
   example, a workflow that sits in a path its provider executes and triggers
   on the current branch. Mark UNVERIFIED any claim you cannot support.
   Answer concisely. If any answer is not discoverable from the files, say
   MISSING and name what you needed."

3. Grade the answers yourself against the drafts. Any MISSING, wrong, or
   guessed answer is a defect in the drafts, not in the fresh agent. Treat
   every UNVERIFIED claim from question 6 as a defect too: prove it with a
   concrete query, or downgrade it in the drafts to a labeled assumption or
   local-only verification.
4. Fix the drafts, then re-run the test once with another fresh context.
5. Record the outcome as one plain-English sentence for the approval summary,
   stating what the test covers, for example: "A fresh agent given only the
   new files correctly answered what the project is, what is in progress,
   what comes next, and how changes are verified, and no external-system
   claim remained unverified - this checks discoverability and consistency,
   not external truth." If the second run still has defects, say so honestly
   and list them as risks in the approval summary instead of hiding them.
