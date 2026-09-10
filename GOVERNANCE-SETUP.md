# Wiring a New Repo into the SPARXSTAR Governance Platform

You are reading this because a repository was created from
`starisian-technologies-proprietary-license`. This is the checklist that takes
that repository from "created" to "governed". Work it top to bottom.

**Time to complete:** about 15 minutes, most of it waiting for a first CI run.

**What is already done for you.** The template ships
`.github/workflows/standards.yml` (the governance caller),
`.github/dependabot.yml` (moves the pins), `sparxstar-specs.yml` (declares what
governs you) and `.github/CODEOWNERS`. Every gate in `standards.yml` is gated
behind a credential check, so this repository is **green from its first push**
and stays green until you finish step 2. A skipped gate is the expected state
of a repo that has not been wired yet — it is not a failure to debug.

---

## Where the facts live (read this before you copy anything)

This document is a **map**. It deliberately does not restate the
authentication block, the org secret names, or the enforcement rules — a
restated copy is where stale patterns survive. Each fact below has exactly one
home:

| You need | Its one home |
|---|---|
| The cross-repo auth pattern (mint blocks, App names, NEVER rules) | `sparxstar-product-specification-registry` → `specs/_platform/SPARXSTAR-CROSS-REPO-ACCESS-STANDARD.md` |
| Adopting the conformance workflows (repo types, caller templates, pin policy, advisory→gate) | `sparxstar-code-conformance` → `REUSABLE-WORKFLOWS.md` |
| The full platform wiring guide, including the propose-back flow | `sparxstar-code-conformance` → `docs/platform-setup/STARISIAN-GOVERNANCE-PLATFORM-SETUP.md` |
| The PR reviewer's interface (inputs, secrets, caller requirements) | `sparxstar-claude-pr-review` → `README.md` and `docs/consumer-setup.md` |
| Which ADRs and invariants bind you | `sparxstar-architecture-governance-registry` → `standards/` |

If this checklist and one of those documents disagree, **that document is
right** — it sits beside the code it describes. Fix this file rather than
working around it.

---

## The five governance repos

Your repository is a *consumer*. It calls into these; it never copies their
logic.

| Repo | Gives you | Reached how |
|---|---|---|
| `sparxstar-architecture-governance-registry` | ADRs, invariants, cross-repo contracts | pushed to you by governance-sync (step 3) |
| `sparxstar-product-specification-registry` | Canonical per-product tech specs | `fetch-specs.yml`, and the PR reviewer reads it for you |
| `sparxstar-code-conformance` | Lint/style/standards enforcement | `uses:` in `standards.yml` |
| `sparxstar-contracts-registry` | Shared PHP interface contracts | `contract-conformance.yml`; Composer for the published packages |
| `sparxstar-claude-pr-review` | AI review against ADRs, specs and contracts | `uses:` in `standards.yml` |

Two names are **retired**. Correct them on sight, in any document or workflow:

| Retired | Current |
|---|---|
| `sparxstar-architecture-decision-record` | `sparxstar-architecture-governance-registry` |
| `sparxstar-platform-standards` | `sparxstar-product-specification-registry` |

---

## Step 1 — Set the repository's own identity

1. **Decide public or private.** This matters mechanically, not just legally:
   the Claude PR reviewer **refuses to run on a public repository**, because it
   pulls private registry content into an Actions artifact and artifacts on a
   public repo are world-readable. On a public repo the `review` job skips by
   design and you get no AI review. Governed product repos should be private.
2. **Apply the license headers** from `LICENSE_HEADER.md` to every source file.
3. **Set the namespace.** `Starisian\Sparxstar\{ProductName}\...`, where the
   namespace segment is the repo name minus the `sparxstar-` prefix. Record it
   in `AGENTS.md`.
4. **Replace this template's `AGENTS.md`** with one describing *your* repo.
   The shipped copy is the platform coding standard, not a description of you —
   and the PR reviewer feeds `AGENTS.md` into its context, so a stale one
   actively misleads the review.
5. Delete `GOVERNANCE-SETUP.md` (this file) once the checklist is complete, or
   keep it and tick the boxes. Do not leave it half-done — a half-wired repo
   looks governed and is not.

---

## Step 2 — Get the credentials to reach this repo

**This is the step that everything else waits on, and the one that is most
often skipped.** Until it is done, every gate in `standards.yml` skips with a
`::warning::` naming this step.

Cross-repo access uses **two org-level GitHub Apps and no PATs, ever**. Their
names, their org Variable and Secret names, and the exact mint block are
defined once — in the **Cross-Repo Access Standard** (`SPARXSTAR-CROSS-REPO-ACCESS-STANDARD.md`,
product-spec registry). Read it there. Do not copy a mint block out of an older
document; several stale copies are in circulation.

What an **org owner** must do for this repository:

1. **Confirm the read App is installed and can reach what this repo READS.**
   Org Settings → GitHub Apps → the composer-resolver App → Repository access.
   *Installing an App org-wide is not the same as scoping it to a repo.* This
   is the single most common misconfiguration on this platform — see
   Troubleshooting.
   Scope it to the repos your workflows read — the two registries (step 3
   below) and any private Composer/npm dependencies. **The PR reviewer does
   not need App access to your own repo**: its two mint steps name the
   registries explicitly, and your PR-head checkout uses the default
   `GITHUB_TOKEN`. Adding your repo there grants cross-repo reach nothing
   uses. The conformance workflows are the case that needs the installation
   itself, for private-dependency git auth.
2. **Confirm the org secrets and variables reach this repository.** They
   already exist at org level; a new repo simply has to be inside their
   visibility scope. Never recreate them, and never create a repo-level copy.
   `COMPOSER_RESOLVER_CLIENT_ID` is not optional: the reviewer's
   `build-context` job validates it and exits 1 with an explicit error when
   it is empty, before minting anything.
3. **Scope the read App to the registries too**, if it is not already: the PR
   reviewer checks out both `sparxstar-architecture-governance-registry` and
   `sparxstar-product-specification-registry`. Missing either produces the
   opaque failure in Troubleshooting.

If you see a secret or variable name in an instruction file that you cannot
find in org settings, **it does not exist** — several fabricated token names
have shipped here before. Verify against org settings or a green workflow run,
never against memory or a document.

---

## Step 3 — Get on the governance allowlist

Governance snapshots are **not** pushed to every repo in the org. Coverage is
an explicit allowlist:

> `sparxstar-architecture-governance-registry` → `config/governance-consumers.txt`

A repo that is not in that file receives nothing. This is deliberate — the sync
scopes its write token to exactly the repos on the list, and an unscoped token
would cover the whole installation.

**To be covered:** open a PR adding this repository's name to that file. The
file is in the sync workflow's trigger paths, so merging the addition fires the
sync to you — no second action needed.

Two rules for that file, both learned the hard way:

- **Every name must resolve to a real repository the App is installed on.**
  The list is passed to token generation as a single `repositories:` scope, and
  GitHub 404s the *entire* request if one name is wrong. One bad name stops the
  snapshot reaching **every** repo, and it dies before the per-repo skip logic
  can help.
- **Add the repo, then check the run.** If your
  `.github/instructions/governance/` is still empty afterwards, the fix is not
  to re-add the name — trigger the sync manually from the registry's Actions
  tab (`Governance Snapshot Sync` → Run workflow) and read the log.

Once synced you receive four compiled, overwrite-on-every-change files in
`.github/instructions/governance/`:

`adr-reference.compiled.md` · `invariants.compiled.md` ·
`contracts.compiled.md` · `open-questions.compiled.md`

Never edit them. They are regenerated from the registry on every governed
change, and an edit is silently discarded on the next sync.

---

## Step 4 — Turn on the checks for your repo type

Open `.github/workflows/standards.yml`. Two jobs are already active for every
repo type: `version-check` and `review`.

Now uncomment the block matching your repo type — `wp-plugin`,
`standalone-react`, or `standalone-node`. The blocks are in the file, each
labelled with the caller template it mirrors. For a repo type not listed, or
for the authoritative version of any block, copy from
`sparxstar-code-conformance/caller-templates/` — those files sit beside the
workflows they call and are the maintained originals.

Three independent version axes. Move them deliberately, and never together by
accident:

| Axis | In this file | Means |
|---|---|---|
| Workflow ref (`@v1.0.2`) | every `uses:` | which executable runs. Bug fixes bump this. |
| `contract_ref` on `version-check` | `v1.0.2` | must equal the workflow ref above, exactly |
| `profile_version` (`v1`) | per job | the dependency/config contract you conform to |

`contract_ref` on the **`review`** job is a different axis again: it names a
tag on the *registries*, not on this platform's workflows. Leave it unless you
know the registries carry the tag you are naming.

**Pin immutable patch tags only.** Never `@main`. Never the moving `@v1` alias.
The reasoning is in `REUSABLE-WORKFLOWS.md`; Dependabot moves the pins for you.

**Start advisory, gate when clean.** Every uncommented job ships
`enforcement_mode: advisory` — violations are reported as warnings and do not
block merge. Once a job reports zero violations, change it to `gate`. Wiring a
new gate must never stall a team; leaving it advisory forever must never become
the norm.

---

## Step 5 — Declare what governs you

Fill in `sparxstar-specs.yml` at the repo root. Leaving it empty is valid and
the reviewer still runs, but findings stay generic — filling this in is what
makes a finding cite the rule it broke.

**Use the `- id:` shape. Anything else is ignored without an error:**

```yaml
specs:
  - id: rlc-games
contracts:
  - id: cross-repo-lineage-node-contract
adrs:
  - id: ADR-011
```

The reviewer parses this with a regex, not a YAML engine. It requires a
section header that is exactly `specs:` — nothing after the colon — followed
by `- id: <value>` lines. Both `specs: [rlc-games]` and a bare `- rlc-games`
parse to **nothing**, silently, and the run reports `(none declared)` while
the repo reads as governed. The shipped file documents this at the point of
use; the trap is that there is no error to notice.

After your first fill-in, open the review job's log and confirm your IDs are
listed. Ten seconds, and it is the only thing separating a working
declaration from a silently empty one.

Look every ID up before you write it; an ID that does not resolve is worse than
an empty list, because the repo reads as governed when it is not. The file
itself says where each kind of ID lives, and after step 3 the compiled files in
`.github/instructions/governance/` are the fastest place to find the ones that
already apply to you.

The reviewer reads this file from the pull request's **base** commit, so a
change to it governs the *next* PR, not the one that changes it.

---

## Step 6 — Prove it

Open a throwaway pull request that changes one line, and confirm on the PR:

- [ ] `Preflight` ran and printed **no** `::warning::` about missing credentials
- [ ] `Version drift` passed (a warning that you are behind the recommended tag is acceptable; below the floor is a hard fail)
- [ ] `Claude PR review` posted a comment — or was correctly skipped because the repo is public
- [ ] Your uncommented repo-type jobs ran
- [ ] `.github/instructions/governance/` contains the four compiled files
- [ ] CODEOWNERS requested review automatically

A gate that **skips** has not passed. Read the `Preflight` log: it names the
step above that fixes it.

---

## Troubleshooting

**`repository not found` at a checkout step, right after a token minted
successfully.**
The mint succeeding proves the App exists and the key is valid. It proves
nothing about *scope*. Org Settings → GitHub Apps → composer-resolver →
Repository access, and confirm every repo the workflow reads is included — for
the PR review gate that means both registries as well as this repo. This is the
documented most common failure on this platform, and the error text never says
so.

**A caller fails at startup with an error about an undeclared secret.**
Secrets do not cross the `workflow_call` boundary automatically. The callee must
declare the secret under `on.workflow_call.secrets` **and be re-tagged**, before
a caller pinning that tag can pass it. Order is always: callee declares → callee
re-tags → caller passes. Open the caller and the callee's `secrets:` block in
the same sitting and compare the exact names; a mismatch there *is* the bug.
Never diagnose this from memory.

**A reusable workflow was edited but nothing changed.**
You are pinned to a tag, and the tag still points at the old commit. Both the
immutable semver tag and any moving major alias must be moved onto the commit
containing the edit. Verify with
`git ls-remote origin 'refs/tags/v1*'` — never from memory.

**`Claude PR review` never runs.**
In order: is the repository private? Are both `ANTHROPIC_API_KEY` and
`COMPOSER_RESOLVER_PRIVATE_KEY` visible to it? Is the trigger `pull_request`?
It must never be `pull_request_target` — that would run PR-head code with a
read-write token in the base-repo context.

**`.github/instructions/governance/` is empty.**
You are not on the allowlist. Step 3.

---

## Never

These are hard prohibitions, carried from the Cross-Repo Access Standard's
NEVER table. That table is the authority; this is a reminder at the point of
use.

- Never `secrets: inherit`. Pass named secrets only.
- Never a PAT for cross-repo access. Use the Apps.
- Never an API key in Variables. Keys are Secrets.
- Never define a token a workflow does not use.
- Never `@main`, and never the moving `@v1` alias, on a `uses:` line.
- Never `pull_request_target` on a workflow that checks out PR-head code.
- Never edit a file under `.github/instructions/governance/`.
- Never restate the mint block in this repo. Link to the standard.
