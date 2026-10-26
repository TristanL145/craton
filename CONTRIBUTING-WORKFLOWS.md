# Craton Contribution Workflows

## Introduction

This guide is a beginner-friendly checklist for contributing to [Project Craton](https://github.com/riscv-ottawa/craton), an RV32 microcontroller project. Contributions are made on a branch and submitted as pull requests; do not commit directly to `main`.

The repository currently includes RTL, verification, software, FPGA and physical-design areas, documentation, CI scripts, and two pinned dependency submodules. Read [CONTRIBUTING.md](CONTRIBUTING.md) for the complete policy.

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

Start with [Workflow 1](#workflow-1-prepare-your-repository), then choose the path that matches the change:

| Change | Next workflow | Example |
| --- | --- | --- |
| New RTL, software, verification, FPGA, physical design, container, or CI code | [Workflow 3](#workflow-3-make-a-source-or-hardware-change) | Add `rtl/dma/dma_frontend.sv` |
| README, mdBook page, design explanation, or API/register documentation | [Workflow 4](#workflow-4-make-a-documentation-change) | Add `docs/src/rtl/dma/registers.md` |
| Move a dependency to a different commit | [Workflow 5](#workflow-5-update-a-dependency-submodule) | Update `deps/cv32e40x` |
| Unsure what should change or how large the change is | [Workflow 2](#workflow-2-plan-the-change) | Turn issue `#42` into a small implementation plan |

After any implementation workflow, continue with [Workflow 6](#workflow-6-verify-the-change), then [Workflow 7](#workflow-7-submit-and-update-a-pull-request).

Choose only one implementation workflow for the main change. If a source change also needs documentation, complete Workflow 3 first, then Workflow 4 before verification.

## Workflow 1: Prepare your repository

**Goal:** Create a clean local copy with submodules and a branch based on the latest upstream `main`.

- [ ] Fork `riscv-ottawa/craton` to your account, if necessary.
- [ ] Clone the fork, including dependencies:

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

- [ ] Add and fetch upstream:

```bash
git remote add upstream https://github.com/riscv-ottawa/craton.git
git fetch upstream
```

- [ ] Create a descriptive branch from upstream `main`:

```bash
git switch main
git pull --ff-only upstream main
git switch -c feat/dma-descriptor-fetch
```

Use a short branch name such as `fix/debug-reset-polarity`, `docs/dma-registers`, or `chore/update-cv32e40x`.

**Next:** If the change needs clarification, use [Workflow 2](#workflow-2-plan-the-change). Otherwise select [Workflow 3](#workflow-3-make-a-source-or-hardware-change), [Workflow 4](#workflow-4-make-a-documentation-change), or [Workflow 5](#workflow-5-update-a-dependency-submodule).

## Workflow 2: Plan the change

**Goal:** Make the work small, reviewable, and placed in the right part of the repository.

- [ ] Find or create an issue describing the problem or feature.
- [ ] Include a clear problem statement, expected behavior, affected area, and verification plan.

Example issue:

```text
Title: Add descriptor fetch support to the DMA frontend

Problem: The DMA frontend cannot fetch chained descriptors.
Expected result: It follows the descriptor address and reports malformed entries.
Affected area: rtl/dma/, dv/dma/, docs/src/rtl/dma/
Verification: Add an unaligned-descriptor regression and document the registers.
```

- [ ] Decide whether the change belongs in this repository or a dependency.
  Changes specific to the CV32E40X core or its verification environment belong in
  the corresponding dependency repository, not directly under `deps/` here.
- [ ] Split unrelated work into separate branches and pull requests.
- [ ] Identify files before editing. For example:

```text
rtl/dma/dma_frontend.sv       # hardware implementation
dv/dma/dma_frontend_tb.sv     # verification
docs/src/rtl/dma/README.md    # component overview
docs/src/rtl/dma/registers.md # software-visible register map
```

**Next:** Use [Workflow 3](#workflow-3-make-a-source-or-hardware-change) for implementation, [Workflow 4](#workflow-4-make-a-documentation-change) for documentation-only work, or [Workflow 5](#workflow-5-update-a-dependency-submodule) for a dependency revision.

## Workflow 3: Make a source or hardware change

**Goal:** Implement code in the correct directory with the required license, style, and verification updates.

Use these locations:

| Directory | Examples |
| --- | --- |
| `rtl/` | `rtl/dma/dma_frontend.sv`, SoC, peripherals, top-level RTL |
| `dv/` | Testbenches, assertions, regressions, covergroups |
| `sw/` | Boot ROM, test programs, RTOS bring-up |
| `fpga/` | Board wrappers, configuration, `.xdc` constraints |
| `pd/` | Floorplan, timing constraints, layout checks |
| `containers/` | Simulation and physical-flow image definitions |
| `ci/` | Shell and Python CI checks |

- [ ] Use descriptive lowercase names with the appropriate extension, such as
  `dma_frontend.sv`, `check_registers.py`, or `board_constraints.xdc`.
- [ ] Add the SPDX identifier near the top of every new source file:

```systemverilog
// SPDX-License-Identifier: Apache-2.0 WITH SHL-2.1
```

Use `Apache-2.0 WITH SHL-2.1` for `rtl/`, `pd/`, and `fpga/`; use `Apache-2.0` elsewhere. Markdown files do not need a per-file header. Do not edit files under `deps/`.
- [ ] Follow the lowRISC Verilog style guide for RTL.
- [ ] Add or update tests, assertions, simulations, or checks where applicable.
- [ ] Keep commits focused and use the required subject format:

```text
feat(dma): add descriptor fetch to the frontend
fix(soc): correct debug port reset polarity
test(dma): cover unaligned descriptor lengths
```

Create signed-off commits with:

```bash
git add rtl/dma/dma_frontend.sv dv/dma/dma_frontend_tb.sv
git commit -s -m "feat(dma): add descriptor fetch to the frontend"
```

**Next:** If the implementation changes behavior that users or integrators need to understand, add documentation using [Workflow 4](#workflow-4-make-a-documentation-change). Otherwise continue to [Workflow 6](#workflow-6-verify-the-change).

## Workflow 4: Make a documentation change

**Goal:** Keep the mdBook and repository documentation accurate, navigable, and buildable.

- [ ] Put component documentation under the matching `docs/src/` directory. For example:

```text
docs/src/rtl/dma/README.md
 docs/src/rtl/dma/theory-of-operation.md
 docs/src/rtl/dma/programmers-guide.md
 docs/src/rtl/dma/interfaces.md
 docs/src/rtl/dma/registers.md
```

- [ ] Use the recommended page names when documenting a component:

| File | Content |
| --- | --- |
| `README.md` | Purpose, features, and a short description |
| `theory-of-operation.md` | How it works and why it is designed that way |
| `programmers-guide.md` | Initialization, examples, and error handling |
| `interfaces.md` | Ports, parameters, and interrupts |
| `registers.md` | Register map |

- [ ] Add every new page to `docs/src/SUMMARY.md`:

```markdown
- [DMA](rtl/dma/README.md)
  - [Theory of operation](rtl/dma/theory-of-operation.md)
  - [Registers](rtl/dma/registers.md)
```

- [ ] Define new external links once in `docs/src/refs.md` and use reference-style links.
- [ ] End pages that use shared references with the correct include:

```markdown
{{#include ./refs.md}}   <!-- docs/src/*.md -->
{{#include ../refs.md}}  <!-- docs/src/<section>/*.md -->
```

- [ ] Prefer one sentence per line and write plainly.
- [ ] Preview the book:

```bash
mdbook serve docs
```

**Next:** Continue to [Workflow 6](#workflow-6-verify-the-change). If the documentation describes a source change not yet implemented, return to [Workflow 3](#workflow-3-make-a-source-or-hardware-change).

## Workflow 5: Update a dependency submodule

**Goal:** Update a pinned external revision without placing dependency code in this repository.

The current submodules are:

```text
deps/cv32e40x     -> riscv-ottawa/cv32e40x
deps/cv32e40x-dv  -> riscv-ottawa/cv32e40x-dv
```

Choose the correct path:

- **Upstream dependency fix:** Make the fix in the dependency repository, then update its pointer here.
- **Craton-specific core or DV change:** Use the dependency repository's `craton` branch, then update the pointer here.
- **Craton code using the dependency:** Stay in this repository and use [Workflow 3](#workflow-3-make-a-source-or-hardware-change).

Example pointer update:

```bash
cd deps/cv32e40x
git fetch origin
git checkout <commit-or-craton-branch>
cd ../..
git add deps/cv32e40x
git commit -s -m "chore(deps): update CV32E40X"
```

In the pull request, state the old and new revisions and why the update is needed. Do not edit files inside the submodule from this repository.

**Next:** Continue to [Workflow 6](#workflow-6-verify-the-change).

## Workflow 6: Verify the change

**Goal:** Catch formatting, policy, documentation, and sign-off failures before opening or updating a pull request.

- [ ] Inspect the working tree and diff:

```bash
git status
git diff --check
git diff upstream/main...HEAD
```

- [ ] Check DCO sign-offs for branch commits:

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

- [ ] Rebase if upstream changed:

```bash
git fetch upstream
git rebase upstream/main
```

After resolving a conflict:

```bash
git add <resolved-file>
git rebase --continue
```

**Next:** If checks pass, continue to [Workflow 7](#workflow-7-submit-and-update-a-pull-request). If a check fails, fix the reported issue and repeat this workflow.

## Workflow 7: Submit and update a pull request

**Goal:** Give maintainers enough context to review, test, and merge the change.

- [ ] Push the branch:

```bash
git push -u origin feat/dma-descriptor-fetch
```

After a rebase or amended commit, use:

```bash
git push --force-with-lease
```

- [ ] Open a pull request from your fork to `riscv-ottawa/craton:main`.
- [ ] Use a concise title matching the commit style, such as `feat(dma): add descriptor fetch`.
- [ ] Include:

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
- Add performance counters in a separate change.
```

- [ ] Wait for CI to pass.
- [ ] Request or obtain at least one approving maintainer review.
- [ ] Respond to review comments and keep all new commits signed off.

**Next:** If review requests changes, use [Workflow 6](#workflow-6-verify-the-change) after making them. If review is approved and CI passes, the change is ready for maintainer merge.

## Common Git commands

| Command | Effect | Example |
| --- | --- | --- |
| `git status` | Shows changed, staged, and untracked files | `git status` |
| `git switch -c <branch>` | Creates and switches to a branch | `git switch -c docs/dma-registers` |
| `git fetch upstream` | Downloads upstream references without changing files | `git fetch upstream` |
| `git pull --ff-only upstream main` | Updates local `main` without creating a merge commit | `git pull --ff-only upstream main` |
| `git add <file>` | Stages a file for the next commit | `git add docs/src/SUMMARY.md` |
| `git diff` | Shows unstaged changes | `git diff` |
| `git diff --cached` | Shows staged changes | `git diff --cached` |
| `git diff --check` | Finds whitespace errors | `git diff --check` |
| `git commit -s` | Creates a commit with a DCO sign-off | `git commit -s -m "docs(dma): describe registers"` |
| `git commit --amend --signoff` | Replaces the last commit and preserves/adds sign-off | `git commit --amend --signoff` |
| `git log --oneline` | Shows compact history | `git log --oneline -5` |
| `git rebase upstream/main` | Replays branch commits on current upstream `main` | `git rebase upstream/main` |
| `git rebase --continue` | Continues a conflict-resolution rebase | `git rebase --continue` |
| `git push -u origin <branch>` | Publishes a branch and sets its tracking branch | `git push -u origin docs/dma-registers` |
| `git push --force-with-lease` | Safely updates a remote branch after rewriting history | `git push --force-with-lease` |
| `git submodule update --init --recursive` | Initializes or updates dependencies | `git submodule update --init --recursive` |
| `git submodule status` | Shows pinned dependency revisions | `git submodule status` |

## References and appendix

### Repository references

- [README.md](README.md) — project overview and quick-start information
- [CONTRIBUTING.md](CONTRIBUTING.md) — authoritative contribution policy
- [MAINTAINERS.md](MAINTAINERS.md) — current reviewers and maintainer process
- [`.gitmodules`](.gitmodules) — dependency repository locations
- [`ci/check-dco.sh`](ci/check-dco.sh) — DCO validation
- [`ci/check-spdx.sh`](ci/check-spdx.sh) — SPDX validation
- [`ci/check-book.sh`](ci/check-book.sh) — mdBook validation
- [`docs/book.toml`](docs/book.toml) — mdBook configuration
- [`docs/src/SUMMARY.md`](docs/src/SUMMARY.md) — documentation navigation

### Required policies

- Every contribution arrives as a pull request.
- Every commit must contain a `Signed-off-by:` line. Use `git commit -s`.
- New source files must use the SPDX identifier required for their path.
- Hardware paths `rtl/`, `pd/`, and `fpga/` use `Apache-2.0 WITH SHL-2.1`.
- Other project source and tooling use `Apache-2.0`; `deps/` retains its own terms.
- New mdBook pages must be listed in `docs/src/SUMMARY.md`.
- New external documentation links belong in `docs/src/refs.md`.
- A pull request needs passing CI and one approving maintainer review.

### Final checklist

- [ ] Work is on a feature/fix/docs/chore branch, not directly on `main`.
- [ ] The issue, if applicable, explains the goal and verification plan.
- [ ] Files are in the correct directory and have descriptive names.
- [ ] Commit subjects use `<type>(<scope>): <description>`.
- [ ] Every commit is signed off.
- [ ] New source files have the correct SPDX identifier.
- [ ] Documentation navigation and references are updated.
- [ ] Relevant local checks pass.
- [ ] The pull request states what changed, what was tested, and what was not tested.
- [ ] CI passes and a maintainer has approved the pull request.
