# Starisian Technologies Proprietary License

> **Template Repository** — Use this as the canonical source for all Starisian Technologies license language, file headers, and governance documents.

---

## Overview

This repository contains the official **Starisian Technologies Proprietary License** language, code ownership policies, and CI workflow templates. It is intended to be used as a **GitHub Template Repository** so that all new Starisian Technologies projects inherit consistent licensing, security, and governance scaffolding from day one.

**Lead Developer / IP Owner:** Max Barrett ([@StarisianDevelopment](https://github.com/starisiandevelopment))  
**Organization:** [Starisian Technologies](https://github.com/Starisian-Technologies) / Max Barrett  
**Jurisdiction:** Los Angeles, California

---

## Repository Contents

| File / Path | Purpose |
|---|---|
| [`LICENSE.md`](./LICENSE.md) | Full proprietary license (public & private/commercial variants) |
| [`LICENSE_HEADER.md`](./LICENSE_HEADER.md) | Copy-paste license headers for source files (full & short) |
| [`CODE_OWNERSHIP.md`](./CODE_OWNERSHIP.md) | Technical governance and code ownership policy |
| [`.github/CODEOWNERS`](.github/CODEOWNERS) | GitHub CODEOWNERS file — auto-assigns @MaximillianGroup to all reviews |
| [`SECURITY.md`](./SECURITY.md) | Vulnerability reporting policy |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | Contribution and CLA policy |
| [`GOVERNANCE-SETUP.md`](./GOVERNANCE-SETUP.md) | **Start here in a new repo** — checklist to wire it into the governance platform |
| [`AGENTS.md`](./AGENTS.md) | Platform coding standard — replace with a description of your repo |
| [`.github/workflows/standards.yml`](.github/workflows/standards.yml) | CI: governance caller (conformance gates + Claude PR review), credential-gated |
| [`.github/dependabot.yml`](.github/dependabot.yml) | Opens the PRs that move the pinned workflow tags forward |
| [`sparxstar-specs.yml`](./sparxstar-specs.yml) | Declares which specs, contracts and ADRs govern this repo |

---

## Using This Template

1. Click **"Use this template"** on GitHub to create a new repository under the Starisian Technologies organization.
2. **Work [`GOVERNANCE-SETUP.md`](./GOVERNANCE-SETUP.md) top to bottom.** It is the checklist that takes the new repo from "created" to "governed" — credentials, the governance allowlist, and turning on the checks for your repo type.
3. Copy the appropriate license header from [`LICENSE_HEADER.md`](./LICENSE_HEADER.md) into every source file.
4. Replace [`AGENTS.md`](./AGENTS.md) with one describing your repo. The shipped copy is the platform coding standard, not a description of you — and the PR reviewer reads `AGENTS.md` into its context, so a stale one actively misleads the review.
5. The `.github/CODEOWNERS` file automatically **requests review** from the code owners on all pull requests. Note that CODEOWNERS only requests — it does not make their approval *required*. Enforcement comes from a branch protection rule or ruleset, which this template does not provision; add one if the new repo needs approval to be mandatory.

### What runs on day one

`.github/workflows/standards.yml` calls the platform's pinned reusable
workflows rather than implementing checks itself. Every gate sits behind a
`preflight` job that checks the credentials are visible and mints a real
scoped token to prove the GitHub App can reach what each gate reads, so **a
repo created from this template is green on its first push** and stays green
through every partially-wired state until `GOVERNANCE-SETUP.md` step 2 is
complete. A gate that skips with a `::warning::` is the expected state of a
repo that has not been wired yet — the warning names the step that fixes it.

Enforcement logic is never copied into a consumer repo. A conformance check
changes once in `sparxstar-code-conformance`, and the PR reviewer changes once
in `sparxstar-claude-pr-review`; every repo picks each up by moving that
pin. The two are separate release lines — moving one does not move the other.

---

## License

All files in this repository — including the license text itself — are the exclusive property of **Starisian Technologies**.  
See [`LICENSE.md`](./LICENSE.md) for full terms.

**This software is NOT open source.** Viewing this repository on GitHub does not grant any rights beyond those explicitly stated in `LICENSE.md` and the [GitHub Terms of Service](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service).

---

*© 2026 Starisian Technologies (Max Barrett). All Rights Reserved. Patent Pending.*
