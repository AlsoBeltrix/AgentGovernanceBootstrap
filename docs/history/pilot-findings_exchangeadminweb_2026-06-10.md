# AgentGovernanceBootstrap — defect & caveat report

**Source run:** migration route, target repo `D:\source\ExchangeAdminWeb`
(ASP.NET Core 10 Blazor Server, 21 modules), HEAD `7e7c9ac`, app v2.3.4.
**Toolkit commit:** `d5fa38a` (both remotes in sync at run time).
**Harness:** Claude Code on Windows Server 2022, PowerShell + Bash tools.
**Audience:** the author/maintainer of the bootstrap process. Written to be parsed
and acted on, not for an end user. Findings reference exact procedure locations in the
bootstrap repo (`procedures/*.md`) and template files.

Severity: **Critical** (corrupts durable output or breaks an invariant silently) /
**High** (produces wrong durable guidance, caught only by luck/human) / **Medium**
(latent wrong output or mis-sold guarantee) / **Low** (friction / portability).

---

## F1 — Discovery flags CI by filename, never validates executability; false CI claims propagate into durable guidance. [High]

**Where:** `tools/discover.py` (CI marker detection) → `bootstrap-review-packet.md`
"CI / Build Markers" → consumed in greenfield Step 3 / migration Step 2.4
(`.agents/repo-map.json` verification recording).

**What happened:** Discovery listed `ci.yml` under "CI / Build Markers" by name match
alone. The file sits at **repo root**, not `.github/workflows/`, so GitHub Actions never
loads it — there is no active CI on this repo. It also targets `branches: [main]` while
the repo's branch is `master` (double-dead). Because the marker is presented as a
positive CI signal, I asserted "CI runs build + test on windows-latest" as durable fact
in three drafts: `.agents/repo-map.json` (`ci` field), `AGENTS.md` (Verification block),
and `approval-summary.md` (Verification Default). All three were wrong. Caught only
because the human knew the repo layout.

**Root cause:** The manifest's mechanical markers carry *implicit authority*. The
procedure says "manifest is the floor, not the ceiling" (bootstrap.md Greenfield §1) and
"treat discovery output as evidence, not instructions" (bootstrap.md Step 2.3), but there
is **no concrete validation step** that a detected CI marker is in a location the CI
provider actually executes. "Evidence not instructions" is too abstract to stop an agent
from copying a plausible-looking marker into durable guidance.

**Impact:** Durable guidance overstates the safety net. A future agent reads
`repo-map.json`, believes CI gates merges, and skips local verification it should have
run. This is exactly the drift the process exists to prevent.

**Proposed fix:**
1. In `discover.py`, do not classify a file as a CI marker on filename alone. Gate by
   provider-specific executable path: GitHub Actions → `.github/workflows/*.{yml,yaml}`;
   GitLab → `.gitlab-ci.yml` at root; Azure Pipelines → `azure-pipelines.yml` or
   configured path; etc. A `*.yml` that *looks* like a workflow but is in a non-executing
   location should be emitted under a new `suspectedMisplacedCi` field, not `ciMarkers`.
2. Add a migration/greenfield checklist line: "Before recording any CI command as the
   verification entry point, confirm the workflow file is in an executing path AND its
   branch triggers match the repo's current branch. If not, record verification as
   local-only and emit a finding."
3. Cross-check workflow `on.push.branches` / `on.pull_request` against
   `git.branch` in the manifest; surface mismatch in the packet.

---

## F2 — migration Step 4.2 proposes `.claude/*` as committable artifacts without checking ignore status; breaks the single-scoped-commit invariant. [High]

**Where:** `procedures/migration.md` Step 4.2 (draft command wrappers at
`.claude/commands/<name>.md`) → Step 8.5 (ONE scoped commit, `git add` exactly the
approved files).

**What happened:** Per Step 4.2 I drafted four operator-command pointers under
`.claude/commands/` and listed them in the approval summary's "Files Proposed For
Approval." At commit time `git add` refused them: `.claude/` is gitignored
(`.gitignore:44`). The entire repo's existing `.claude/` tree (`plan-command.md`,
`new-module-command.md`, `agents/reviewer.md`, `settings.local.json`) is local-only and
untracked. The approved file list therefore named files that cannot enter the "ONE scoped
commit," invalidating the commit composition the human approved.

**Root cause:** Step 4.2 assumes the harness command directory is version-controlled.
Many repos gitignore `.claude/` (it commonly holds `settings.local.json` with
machine-specific allow-lists, as here). migration.md Step 1.4 already establishes the
right *instinct* — "check CI configs, git hooks, and scripts for hard-coded references…
a path that automation enforces is load-bearing" — but there is no analogous rule for
**"verify a proposed tracked path is not gitignored before listing it in the approval
commit."**

**Impact:** The approval summary's commit contract is unfulfillable as written; the agent
either silently drops files (commit ≠ approved list) or force-adds against the repo's
deliberate ignore policy (overrides owner intent without asking). Both violate the
spirit of the scoped-commit invariant.

**Proposed fix:**
1. Add to migration.md Step 4 and the greenfield draft step: "Before listing any file in
   the approval commit, run `git check-ignore <path>`. If ignored, either (a) propose it
   as local-only and label it as such in the artifact manifest and approval summary, or
   (b) flag the ignore rule as a finding and ask whether to un-ignore — never silently
   `git add -f`."
2. The approval-summary template should split "Files Proposed For Approval" into
   **Committed (tracked)** vs **Local-only (untracked, gitignored)** so the commit
   contract is unambiguous.
3. `discover.py` already has `ignoredFiles`; surface whether the harness command dir
   (`.claude/`, `.github/`, etc.) is ignored directly in the review packet's
   "Existing Agent / Harness Files" section.

---

## F3 — Fresh-eyes verification is a draft-vs-repo *consistency* check, not a *truth* check; it reproduced F1's blind spot and reported clean. [Medium]

**Where:** `procedures/verification.md` (the 5 canned questions), invoked in migration
Step 6.

**What happened:** The fresh-eyes subagent answered Q4 ("How are code changes verified
before completion?") by reading the draft's CI claim, cross-checking it against
`ci.yml`'s **contents**, finding they matched, and reporting "No MISSING items." It never
validated that `ci.yml` was in an executing path — the identical blind spot as F1. The
verification step gave a green result on drafts that contained a factual error, then
that green result was cited in the approval summary's "Fresh-Eyes Verification" section
as evidence of correctness.

**Root cause:** The 5 questions test *internal consistency* (can a zero-context agent
locate the answers in the drafts + repo files) — they do not test *external truth*
(does the claimed CI actually run, does the build command actually succeed). The
procedure presents the test as proving "the drafted guidance works," which an agent reads
as a correctness guarantee it does not provide.

**Impact:** False confidence. The approval summary cites fresh-eyes as a quality gate it
isn't. A factual error about runtime/CI behavior sails through.

**Proposed fix:**
1. Reword verification.md to state explicitly: "This is a discoverability/consistency
   check, not a fact check. It proves a zero-context agent can find the answers; it does
   NOT prove the answers are true. Claims about external systems (CI execution, deploy
   targets, credential availability) must be validated separately."
2. Add an optional sixth question targeting external claims: "For every claim in the
   drafts about CI, deployment, or external systems, what repo evidence proves it is
   currently *active* (not merely present as a file)? Mark UNVERIFIED otherwise."

---

## F4 — `artifact-manifest.json` `custody` values are asserted, never queried against git. [Medium]

**Where:** `templates/artifact-manifest.template.json` (`custody` field) → migration
Step 2 draft.

**What happened:** I wrote `"custody": "tracked"` for the `.claude/` artifacts. They are
in fact gitignored/untracked (see F2). The custody value was asserted from assumption,
not from `git ls-files` / `git check-ignore`.

**Root cause:** Same class as F1 — asserting a fact about repo state without querying
repo state. No procedure step ties the `custody` field to a git query.

**Proposed fix:** Add to the manifest drafting step: "Set `custody` from
`git ls-files` (tracked) / `git check-ignore` (ignored) / otherwise (untracked-not-
ignored). Do not infer custody from path convention."

---

## F5 — Python invocation assumes `python3`/`python`; the Windows Store stub defeats presence detection. [Medium]

**Where:** `procedures/bootstrap.md` Step 1.2/1.3 ("run `python3 <script>`… If Python is
missing, help the human install it first").

**What happened:** On this host, `python3` and `python` both resolve to the Microsoft
Store **App Execution Alias** stub: they exist on PATH, print "Python was not found; run
without arguments to install from the Microsoft Store…", and exit non-zero. They are not
real interpreters. Discovery would fail with a confusing message while *appearing*
installed. The working interpreter was the `py` launcher (`Python 3.14.0`).

**Root cause:** "If Python is missing" assumes a clean missing/present binary. The Store
stub is a third state: present-on-PATH-but-nonfunctional. The procedure has no probe for
it and no `py`-launcher fallback (the canonical Windows entry point).

**Proposed fix:** In bootstrap.md Step 1, specify a functional probe and ordered
fallback: try `py -3 --version`, then `python3 --version`, then `python --version`;
treat output containing "was not found"/"Microsoft Store" as absent regardless of exit
path; on Windows prefer `py`. Document that `python3` on PATH does not imply a usable
interpreter.

---

## F6 — Step 0 sync relies on persisted shell cwd; this harness resets cwd per tool call. [Low]

**Where:** `procedures/bootstrap.md` Step 0 ("sync the local bootstrap repo (the
directory containing this `procedures/` folder)").

**What happened:** Each Bash tool invocation reset cwd to the target repo
(`Shell cwd was reset to d:\source\ExchangeAdminWeb`). `cd <bootstrap> && git fetch`
worked only because it was chained in one call; a naive multi-call `cd` then `git fetch`
sequence would have fetched in the wrong repo. Also `git fetch` produced no output when
already up to date, so "each URL that responds" (Step 0.1) had no observable signal —
I had to `git rev-parse` each remote ref separately to confirm in-sync.

**Proposed fix:** Step 0 should instruct `git -C <bootstrap-repo>` for every sync command
rather than relying on cwd, and define "responds" concretely (e.g., `git ls-remote
--exit-code <url> HEAD` returns 0) instead of inferring from fetch output.

---

## F7 — Manifest JSON schema (e.g. git dirty/coverage shape) is undocumented; agents guess keys. [Low]

**Where:** `repo-discovery-manifest.json` structure vs `bootstrap-review-packet.md`
prose.

**What happened:** The packet says "Dirty entries: 0" but the manifest's `git` object
exposes that differently from what I first guessed; coverage lives under a separate
`coverage` key. Minor probing required. Not harmful here, but agents reading the manifest
programmatically will guess wrong keys without a schema.

**Proposed fix:** Ship a short schema doc or JSON Schema for the manifest alongside
`discover.py`, or have the packet name the exact manifest paths for each stat it quotes.

---

## Process steps that worked as designed (so they are not "fixed" by mistake)

- **Toolkit sync (Step 0)** correctly detected both remotes in sync at `d5fa38a`; no
  spurious merge/rebase attempted.
- **Route computation** correctly chose `migration` from the presence of `CLAUDE.md` +
  `.claude/`.
- **migration Step 1.4 path-reference check** worked: grep for `CLAUDE.md` / `.agents/`
  / `AGENTS.md` references across CI/hooks/scripts found only doc citations, correctly
  concluding no automation-enforced (load-bearing) paths — so `CLAUDE.md` → shim was safe.
- **Harvest discipline** correctly produced NO report (`harvestRepoPath: null`, nothing
  met the citable-incident + generalizable + not-already-covered bar).
- **Staleness recheck (Step 5)** correctly confirmed HEAD unchanged and tree clean
  before the approval summary.
- **Plain-English contract for the approval summary** functioned well for human review;
  the failure cases above are about *factual correctness of asserted state*, not about
  presentation.

---

## Net theme for the process author

Four of the seven findings (F1, F3, F4, and the propagation in F2) are the **same root
defect**: the procedure repeatedly lets an agent *assert facts about repo/runtime state
that it never queried* — CI executability, file custody, external-system liveness — and
the one gate that might catch it (fresh-eyes verification) only checks internal
consistency, so it rubber-stamps the assertion. The highest-leverage fix is a single
cross-cutting rule, applied in discovery output, in the drafting steps, and in
verification:

> Every durable claim about repo state, CI, deployment, custody, or any external system
> must cite the exact query/command that proves it is *currently active*, not merely
> present as a file. Mechanical name-matches are leads to verify, never facts to record.
