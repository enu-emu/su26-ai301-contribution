# su26-ai301-contribution

# Contribution 1: Add documentation on how to run the Burr UI server

**Contribution Number:** 1  
**Student:** Ena Salazar
**Issue:** [apache/burr #272 — Add documentation on how to run the Burr UI server](https://github.com/apache/burr/issues/272)  
**Status:** Phase III — Complete

---

## Why I Chose This Issue

I chose this issue because it felt like a well-scoped first contribution. Documentation work lets me focus on understanding the project without needing to immediately dive into complex code changes. I'm also drawn to tools that work with LLMs, and Burr's UI server is a core part of what makes the framework usable in practice. Writing this documentation will force me to actually run and understand the tool end to end, which is exactly the kind of hands-on learning I'm looking for.

---

## Understanding the Issue

### Problem Description

Burr has a built-in UI server that lets you visualize and debug your AI application's execution in real time. However, there is no dedicated section in the documentation explaining how to install it, run it, or use it. New users have no clear reference point for getting the UI up and running.

### Expected Behavior

The documentation should have a dedicated section covering:

1. How to install the UI server (`pip install "apache-burr[start]"`)
2. How to run it (`burr` command)
3. Optional commands and flags (e.g. creating demo data)
4. What you can do with the UI: browse runs by project, inspect individual steps, replay state for debugging, etc.

### Current Behavior

The only mention of the UI server lives in `docs/concepts/tracking.rst`, where it gets a single line: "run `burr` in the terminal." There is no installation guidance, no explanation of optional commands, and no description of what the UI actually shows or how to use it.

### Affected Components

- `docs/concepts/tracking.rst` — contains the only existing mention of the UI server
- Likely a new file: `docs/concepts/ui.rst` or an expanded section within `tracking.rst`

---

## Reproduction Process

### Environment Setup

**1. Fork the repo on GitHub**

Go to https://github.com/apache/burr and click "Fork" in the top right. This creates your own copy at `github.com/enu-emu/burr`.

**2. Clone your fork locally**

```bash
git clone https://github.com/enu-emu/burr.git
cd burr
```

This downloads the repo to your machine and puts you inside the folder.

**3. Add the original repo as "upstream"**

```bash
git remote add upstream https://github.com/apache/burr.git
```

`origin` points to your fork. `upstream` points to the original. You can verify both with:

```bash
git remote -v
```

**4. Create and activate a Python virtual environment**

```bash
python -m venv burr-fork
source burr-fork/bin/activate
```

This creates an isolated Python environment so your project dependencies don't conflict with other things on your machine. You should see `(burr-fork)` in your terminal prompt when it's active.

> Note: the virtual environment folder lives inside the `burr` directory. To activate it from a different folder (e.g. `su26-ai301-contribution`), run:
> ```bash
> source ../burr/burr-fork/bin/activate
> ```

**5. Install the package in editable mode**

```bash
pip install -e .
```

The `-e` flag means "editable" — changes you make to the source code take effect immediately without reinstalling.

**6. Create a working branch**

```bash
git checkout -b docs/burr-ui-server
```

This creates a new branch named `docs/burr-ui-server` and switches to it. Always work on a branch, never directly on `main`.

**7. Push the branch to your fork on GitHub**

```bash
git push -u origin docs/burr-ui-server
```

The `-u` flag sets up tracking so future `git push` commands know where to send changes. After this the branch exists both locally and on GitHub.

### Steps to Reproduce

1. Visit the [Burr documentation](https://burr.dagworks.io/)
2. Search for any dedicated page on the UI server — none exists
3. Check `docs/concepts/tracking.rst` — only a one-line mention of running `burr`
4. A new user trying to get the UI running has no complete reference

### Reproduction Evidence

- **Branch link:** [enu-emu/burr — docs/burr-ui-server](https://github.com/enu-emu/burr/tree/docs/burr-ui-server)
- **My findings:** The gap is confirmed. `tracking.rst` has a minimal mention but no dedicated UI documentation exists anywhere in the `docs/` folder.

---

## Solution Approach

### Analysis

The root cause is a documentation gap, not a code bug. The Burr UI server exists and works — `docs/concepts/tracking.rst` mentions it in a single line (`"run burr in the terminal after pip install 'apache-burr[start]'"`). There is no dedicated page explaining the installation, startup options, notebook integration, FastAPI embedding, or what the UI actually shows. A user encountering the UI for the first time has no complete reference to follow.

### Proposed Solution

Create a new documentation page `docs/concepts/ui.rst` that covers the full UI server workflow: installation, starting the server, notebook/Colab usage, embedding in a FastAPI app, and a description of the UI's capabilities. Then update `tracking.rst` and `docs/concepts/index.rst` to link to it.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Burr includes a UI server that lets you visualize and debug your app's execution, meaning you can replay any run step by step, inspect what state each action received and returned, and track how your app moved through its state machine. However, there is no dedicated documentation page covering how to install it, run it, or what it can do. New users have no reference point for getting the UI up and running or understanding its capabilities.

**Match:** `docs/concepts/tracking.rst` is the closest existing reference. It documents the tracking client and mentions the `burr` command to start the server, but lacks installation steps and a description of the UI's capabilities. The structure and `.rst` formatting conventions in that file (headings, code blocks, notes) will serve as the model for the new documentation.

**Plan:**
1. Create a new file `docs/concepts/ui.rst` to house the dedicated UI server documentation.
2. Write a section covering installation: `pip install "apache-burr[start]"` and what the extra installs.
3. Write a section covering how to start the server: `burr` command, default port (7241), and that it opens a browser window automatically.
4. Write a section covering optional startup flags — all confirmed from `burr --help`:
   - Default behavior copies demo data into `~/.burr` so new users can explore the UI immediately without needing their own app running first. Pass `--no-copy-demo_data` to skip it.
   - `--port INTEGER`: change the default port (7241).
   - `--host TEXT`: bind to a specific host (e.g. `0.0.0.0` to expose on a network or in Docker).
   - `--no-open`: start the server without auto-opening a browser window.
   - `--backend [local|s3]`: choose the storage backend (local filesystem or S3).
   - `--dev-mode`: run in development mode.
5. Write a section covering notebook/Colab usage: `%load_ext burr.integrations.notebook` + `%burr_ui` magic.
6. Write a section covering the FastAPI embedding option via `mount_burr_ui` for users who want to run the UI within their own server.
7. Add a section describing what the UI actually shows — using the demo data as the walkthrough example since that's what a new user sees on first launch: browsing runs by project, clicking into a run to see each action's inputs and outputs, inspecting state at each step, and replaying a run from a prior state for debugging.
8. Update `docs/concepts/tracking.rst` to replace its current one-line mention with a cross-reference link to the new page.
9. Add the new page to `docs/concepts/index.rst` so it appears in the navigation.

**Implement:** [Branch link](https://github.com/enu-emu/burr/tree/docs/burr-ui-server) — commits will be added here as work progresses.

**Review:** Before submitting the PR, I will self-review against `CONTRIBUTING.rst` and `developer_setup.md` in the project root, which specify that documentation lives in `docs/` and is written in `.rst` format using Sphinx conventions. I will also check the existing `.rst` files (particularly `tracking.rst`) to match heading levels, code-block formatting, and cross-reference style. The project's commit convention (from `CONTRIBUTING.rst`) uses short imperative subject lines — I will follow that for all commits on this branch.

**Evaluate:** Verification steps:
1. Run `make html` inside the `docs/` folder to confirm Sphinx builds without warnings or errors on the new page.
2. Confirm the new page appears in the left-hand navigation under "Concepts" in the built HTML output.
3. Confirm the cross-reference link in `tracking.rst` resolves correctly in the build.
4. No automated tests are expected for documentation-only changes; the Sphinx build passing is the standard verification for this project type.

---

## Testing Strategy

### Unit Tests

Not applicable — this is a documentation-only change. No source code was modified.

### Integration Tests

Not applicable.

### Manual Testing

- Ran `make html` inside `docs/` — Sphinx build passed with 0 new warnings from `ui.rst` or `index.rst` (13 pre-existing warnings from other files were unchanged)
- Confirmed `ui.rst` appears under "Concepts" in the built HTML navigation
- Ran `es_test_notebook.ipynb` end to end — counter app ran to `Final count: 5` and the run appeared in the UI at `localhost:7241` under `es-test-counter`
- Manually verified every UI feature described in the "What You Can Do" section using the demo projects

---

## Implementation Notes

### Week 1 Progress

Explored the Burr codebase to understand how the UI server works end to end. Confirmed the documentation gap: `tracking.rst` had one line mentioning `burr`, nothing more. Set up a local virtual environment, installed `apache-burr[start]`, and got the UI server running at `localhost:7241`. Ran into two environment issues: a pandas/Python 3.13 incompatibility (fixed with `pip install --upgrade pandas`) and a missing frontend build when running `burr` from inside the source directory (fixed by running `burr` from the home directory). Explored all four demo projects in the UI to understand what it actually shows before writing a word of documentation.

### Week 2 Progress

Built a test Jupyter notebook (`es_test_notebook.ipynb`) to verify the full tracking flow — created a counter state machine with `.with_tracker()`, ran it, and confirmed the run appeared in the UI under `es-test-counter`. This gave me firsthand knowledge of every feature described in the docs. Created `docs/concepts/ui.rst` with installation, CLI options (table format), demo data walkthrough, "What You Can Do" section with step-by-step Try-it instructions for each UI feature, notebook launch, FastAPI embedding, and how to wire up tracking. Registered the page in `docs/concepts/index.rst`. Ran a Sphinx build to confirm zero new warnings. Committed and pushed to the `docs/burr-ui-server` branch.

### Code Changes

- **Files created:** `docs/concepts/ui.rst` — 271 lines covering the full UI server workflow
- **Files modified:** `docs/concepts/index.rst` — added `ui` to the concepts toctree
- **Key commits:**
  - [`08827c4`](https://github.com/enu-emu/burr/commit/08827c4) — Add burr-fork venv to .gitignore
  - [`e344503`](https://github.com/enu-emu/burr/commit/e344503) — docs: add Burr UI server documentation (issue #272)
- **Approach decisions:** Created a new file rather than expanding `tracking.rst` to keep concerns separated — the tracking client and the UI server are distinct enough to warrant their own pages. Used a list-table for CLI flags to make them scannable. Added "Try it:" instructions for every UI feature so readers have something concrete to do rather than just passive reading.

---

## Pull Request

**PR Link:** [apache/burr #810](https://github.com/apache/burr/pull/810)

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**

- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
