# Craton Contribution Workflows

## Introduction

This beginner-friendly guide explains how to contribute to [Project Craton](https://github.com/riscv-ottawa/craton), an RV32 microcontroller project. Work on a branch, submit a pull request, and never commit directly to `main`.

This guide supplements the authoritative [CONTRIBUTING.md](CONTRIBUTING.md). It covers branch setup, issue planning, source and documentation changes, dependency updates, verification, sign-off, and pull requests.

## Table of contents

1. [Choose a workflow](#choose-a-workflow)
2. [Workflow 1: Prepare your repository](#workflow-1-prepare-your-repository)
3. [Workflow 2: Plan the change](#workflow-2-plan-the-change)
4. [Workflow 3: Make a source or hardware change](#workflow-3-make-a-source-or-hardware-change)
5. [Workflow 4: Make a documentation change](#workflow-4-make-a-documentation-change)
6. [Workflow 5: Update a dependency submodule](#workflow-5-update-a-dependency-submodule)
7. [Workflow 6: Verify the change](#workflow-6-verify-the-change)
8. [Workflow 7: Submit and update a pull request](#workflow-7-submit-and-update-a-pull-request)
9. [Common Git commands](#common-git-commands)
10. [References and appendix](#references-and-appendix)

## Choose a workflow

Always begin with [Workflow 1](#workflow-1-prepare-your-repository). Use [Workflow 2](#workflow-2-plan-the-change) first when the goal, affected area, or ownership is unclear.

| If you are changing... | Follow... | Example |
| --- | --- | --- |
| RTL, verification, software, FPGA, physical design, containers, or CI | [Workflow 3](#workflow-3-make-a-source-or-hardware-change) | `rtl/dma/dma_frontend.sv` |
| README files, mdBook pages, design explanations, or register documentation | [Workflow 4](#workflow-4-make-a-documentation-change) | `docs/src/rtl/dma/registers.md` |
| A pinned external dependency revision | [Workflow 5](#workflow-5-update-a-dependency-submodule) | `deps/cv32e40x` |
| Both code and documentation | Workflow 3, then Workflow 4 | Add RTL and explain its registers |
| A core or DV dependency itself | Workflow 5, with work performed in its repository | Fix `cv32e40x` upstream |

After the selected implementation workflow, always use [Workflow 6](#workflow-6-verify-the-change), then [Workflow 7](#workflow-7-submit-and-update-a-pull-request). If review requests changes, return to Workflow 6 after editing.

## Workflow 1: Prepare your repository

**Goal:** Start from current upstream code, with all submodules available, on a descriptive branch.

- [ ] Fork `riscv-ottawa/craton`.
- [ ] Clone the fork recursively:

```bash
git clone --recurse-submodules https://github.com/<your-user>/craton.git
cd craton
```

Example:

```bash
git clone --recurse-submodules https://github.com/TristanL145/craton.git
```

For an existing clone:

```bash
git submodule update --init --recursive
```

- [ ] Add upstream and update local `main`:

```bash
git remote add upstream https://github.com/riscv-ottawa/craton.git
git fetch upstream
git switch main
git pull --ff-only upstream main
```

- [ ] Create a branch using a type and short description:

```bash
git switch -c feat/dma-descriptor-fetch
```

Good examples are `fix/debug-reset-polarity`, `docs/dma-registers`, `test/dma-unaligned-descriptors`, and `chore/update-cv32e40x`.

**Next:** If the change is not yet defined, use Workflow 2. Otherwise choose Workflow 3, 4, or 5 from the table above.

## Workflow 2: Plan the change

**Goal:** Make the work small, reviewable, and assigned to the correct repository and directory.

- [ ] Find or create an issue.
- [ ] State the problem, expected result, affected paths, and verification plan.

Example issue:

```text
Title: Add descriptor fetch support to the DMA frontend

Problem: The DMA frontend cannot fetch chained descriptors.
Expected result: It follows the descriptor address and reports malformed entries.
Affected paths: rtl/dma/, dv/dma/, docs/src/rtl/dma/
Verification: Add an unaligned-descriptor regression and document the registers.
```

- [ ] Split unrelated work into separate issues, branches, and pull requests.
- [ ] Decide whether the change belongs here or in a dependency. Changes to the CV32E40X core or its DV environment belong in their own repositories, not directly under `deps/`.
- [ ] Identify likely files before editing:

```text
rtl/dma/dma_frontend.sv       # implementation
dv/dma/dma_frontend_tb.sv     # verification
docs/src/rtl/dma/README.md    # component overview
docs/src/rtl/dma/registers.md # register map
```

**Next:** Use Workflow 3 for implementation, Workflow 4 for documentation-only work, or Workflow 5 for a dependency pointer update.

## Workflow 3: Make a source or hardware change

**Goal:** Add or modify code in the correct location, with licensing, style, and verification handled.

| Directory | Typical contents | Example filename |
| --- | --- | --- |
| `rtl/` | SoC, peripherals, DMA, top-level RTL | `rtl/dma/dma_frontend.sv` |
| `dv/` | Testbenches, assertions, regressions, coverage | `dv/dma/dma_frontend_tb.sv` |
| `sw/` | Boot ROM, test programs, RTOS bring-up | `sw/tests/dma_smoke.c` |
| `fpga/` | Board wrappers and constraints | `fpga/board_constraints.xdc` |
| `pd/` | Floorplan and physical-flow files | `pd/core.sdc` |
| `containers/` | Build and simulation image definitions | `containers/sim/Dockerfile` |
| `ci/` | CI shell and Python checks | `ci/check_registers.py` |

- [ ] Use descriptive lowercase filenames and the correct extension.
- [ ] Add an SPDX header to every new source file. Hardware paths `rtl/`, `pd/`, and `fpga/` use:

```systemverilog
// SPDX-License-Identifier: Apache-2.0 WITH SHL-2.1
```

Other project source and tooling use `Apache-2.0`. Markdown files do not need a per-file header. Do not edit files under `deps/`.
- [ ] Follow the lowRISC Verilog style guide for RTL.
- [ ] Add or update tests, assertions, simulations, or checks where applicable.
- [ ] Keep commits focused and format subjects as `<type>(<scope>): <description>`:

```text
feat(dma): add descriptor fetch to the frontend
fix(soc): correct debug port reset polarity
test(dma): cover unaligned descriptor lengths
```

- [ ] Sign every commit:

```bash
git add rtl/dma/dma_frontend.sv dv/dma/dma_frontend_tb.sv
git commit -s -m "feat(dma): add descriptor fetch to the frontend"
```

**Next:** If behavior, interfaces, registers, or setup changed, continue with Workflow 4. Otherwise go to Workflow 6.

## Workflow 4: Make a documentation change

**Goal:** Keep the mdBook and repository documentation accurate, discoverable, and buildable.

- [ ] Place component documentation under the matching `docs/src/` path:

```text
docs/src/rtl/dma/README.md
docs/src/rtl/dma/theory-of-operation.md
docs/src/rtl/dma/programmers-guide.md
docs/src/rtl/dma/interfaces.md
docs/src/rtl/dma/registers.md
```

- [ ] Use these page names where applicable:

| File | Purpose |
| --- | --- |
| `README.md` | Purpose, features, and overview |
| `theory-of-operation.md` | How and why the block works |
| `programmers-guide.md` | Initialization, examples, and errors |
| `interfaces.md` | Ports, parameters, and interrupts |
| `registers.md` | Register map |

- [ ] Add every new page to `docs/src/SUMMARY.md`:

```markdown
- [DMA](rtl/dma/README.md)
  - [Theory of operation](rtl/dma/theory-of-operation.md)
  - [Registers](rtl/dma/registers.md)
```

- [ ] Put new external URLs in `docs/src/refs.md` and use reference-style links.
- [ ] Add the shared reference include to pages that use those labels:

```markdown
{{#include ./refs.md}}   <!-- docs/src/*.md -->
{{#include ../refs.md}}  <!-- docs/src/<section>/*.md -->
```

- [ ] Prefer one sentence per line and write plainly.
- [ ] Preview locally with `mdbook serve docs`.

**Next:** Go to Workflow 6. If this documentation describes an unimplemented feature, return to Workflow 3 first.

## Workflow 5: Update a dependency submodule

**Goal:** Change a pinned dependency revision without copying or editing dependency source in this repository.

Current submodules:

```text
deps/cv32e40x     -> riscv-ottawa/cv32e40x
deps/cv32e40x-dv  -> riscv-ottawa/cv32e40x-dv
```

Choose the correct path:

- **Upstream fix:** Make the fix in the dependency repository, then update the pointer here.
- **Craton-specific core or DV work:** Use the dependency repository's `craton` branch, then update the pointer here.
- **Craton code that consumes the dependency:** Stay here and use Workflow 3.

Example:

```bash
cd deps/cv32e40x
git fetch origin
git checkout <commit-or-craton-branch>
cd ../..
git add deps/cv32e40x
git commit -s -m "chore(deps): update CV32E40X"
```

In the pull request, include the old revision, new revision, and reason for the update. Do not edit files inside the submodule from this repository.

**Next:** Continue to Workflow 6.

## Workflow 6: Verify the change

**Goal:** Catch policy, whitespace, license, documentation, and sign-off errors before review.

- [ ] Inspect the tree and complete diff:

```bash
git status
git diff --check
git diff upstream/main...HEAD
```

- [ ] Check DCO sign-offs:

```bash
ci/check-dco.sh upstream/main..HEAD
```

- [ ] Check SPDX identifiers:

```bash
ci/check-spdx.sh
```

- [ ] For documentation changes, run:

```bash
ci/check-book.sh
python3 ci/check-refdefs.py
lychee --config lychee.toml './*.md' 'docs/src/**/*.md'
```

- [ ] Rebase on current upstream if needed:

```bash
git fetch upstream
git rebase upstream/main
```

After resolving conflicts:

```bash
git add <resolved-file>
git rebase --continue
```

**Next:** If all relevant checks pass, use Workflow 7. If a check fails, fix it and repeat this workflow.

## Workflow 7: Submit and update a pull request

**Goal:** Give maintainers enough information to review, verify, and merge the change.

- [ ] Push the branch:

```bash
git push -u origin feat/dma-descriptor-fetch
```

After rebasing or amending commits:

```bash
git push --force-with-lease
```

- [ ] Open a pull request from your fork to `riscv-ottawa/craton:main`.
- [ ] Use a title such as `feat(dma): add descriptor fetch`.
- [ ] Include a summary, verification results, untested areas, and follow-up work:

```markdown
## Summary
- Adds descriptor fetching to the DMA frontend.
- Reports malformed descriptor entries.

## Verification
- `ci/check-dco.sh upstream/main..HEAD`
- `ci/check-spdx.sh`
- DMA regression: passed

## Not tested
- FPGA timing flow was not run.

## Follow-up
- Add performance counters separately.
```

- [ ] Wait for CI to pass.
- [ ] Obtain at least one approving maintainer review.
- [ ] Respond to review comments with signed-off commits.

**Next:** If review requests changes, edit the branch, then repeat Workflow 6 and update the pull request. If CI passes and review is approved, the change is ready for maintainer merge.

## Common Git commands

| Command | Effect | Example |
| --- | --- | --- |
| `git status` | Shows changed, staged, and untracked files | `git status` |
| `git switch -c <branch>` | Creates and switches to a branch | `git switch -c docs/dma-registers` |
| `git fetch upstream` | Downloads upstream references without changing files | `git fetch upstream` |
| `git pull --ff-only upstream main` | Updates `main` without a merge commit | `git pull --ff-only upstream main` |
| `git add <file>` | Stages a file | `git add docs/src/SUMMARY.md` |
| `git diff` | Shows unstaged changes | `git diff` |
| `git diff --cached` | Shows staged changes | `git diff --cached` |
| `git diff --check` | Finds whitespace errors | `git diff --check` |
| `git commit -s` | Commits with a DCO sign-off | `git commit -s -m "docs(dma): describe registers"` |
| `git commit --amend --signoff` | Replaces the last commit and adds sign-off | `git commit --amend --signoff` |
| `git log --oneline` | Shows compact history | `git log --oneline -5` |
| `git rebase upstream/main` | Replays branch commits on current upstream `main` | `git rebase upstream/main` |
| `git rebase --continue` | Continues a conflict-resolution rebase | `git rebase --continue` |
| `git push -u origin <branch>` | Publishes and tracks a branch | `git push -u origin docs/dma-registers` |
| `git push --force-with-lease` | Updates a remote after rewriting history | `git push --force-with-lease` |
| `git submodule update --init --recursive` | Initializes or updates dependencies | `git submodule update --init --recursive` |
| `git submodule status` | Shows pinned dependency revisions | `git submodule status` |

## References and appendix

### Repository references

- [README.md](README.md) — project overview
- [CONTRIBUTING.md](CONTRIBUTING.md) — authoritative contribution policy
- [MAINTAINERS.md](MAINTAINERS.md) — reviewers and maintainer process
- [`.gitmodules`](.gitmodules) — dependency locations
- [`ci/check-dco.sh`](ci/check-dco.sh) — DCO validation
- [`ci/check-spdx.sh`](ci/check-spdx.sh) — SPDX validation
- [`ci/check-book.sh`](ci/check-book.sh) — mdBook validation
- [`docs/book.toml`](docs/book.toml) — mdBook configuration
- [`docs/src/SUMMARY.md`](docs/src/SUMMARY.md) — book navigation

### Policy quick reference

- Submit all contributions as pull requests.
- Sign every commit with `git commit -s`.
- Use `Apache-2.0 WITH SHL-2.1` in new files under `rtl/`, `pd/`, and `fpga/`.
- Use `Apache-2.0` for other project source and tooling; preserve dependency licenses under `deps/`.
- Add new mdBook pages to `docs/src/SUMMARY.md`.
- Add new external documentation links to `docs/src/refs.md`.
- A pull request needs passing CI and one approving maintainer review.

### Final checklist

- [ ] Work is on a descriptive branch, not directly on `main`.
- [ ] The issue explains the goal and verification plan, when applicable.
- [ ] Files use the correct directory, name, extension, and SPDX identifier.
- [ ] Commit subjects use `<type>(<scope>): <description>`.
- [ ] Every commit is signed off.
- [ ] Documentation navigation and references are updated.
- [ ] Relevant local checks pass.
- [ ] The pull request states what changed, what was tested, and what was not tested.
- [ ] CI passes and a maintainer approves the pull request.
