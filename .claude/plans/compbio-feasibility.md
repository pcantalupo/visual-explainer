# Can visual-explainer be used for computational biology projects?

**Date:** 2026-02-25
**Issue:** [pcantalupo/visual-explainer#2](https://github.com/pcantalupo/visual-explainer/issues/2)

---

## Summary

Yes — visual-explainer can serve comp bio projects today for ad-hoc diagram generation, and with one new prompt file (`/bio-project-recap`) it can become a powerful dashboard generator for long-running analysis projects. The key insight is that the [compbio project documentation template](https://github.com/pcantalupo/claude-scientific-skills/blob/main/scientific-skills/markdown-mermaid-writing/templates/compbio_project_documentation.md) already defines structured, parseable sections that map directly onto visual-explainer's rendering capabilities.

This report uses the [Clark scRNA-seq project](https://github.com/pcantalupo/visual-explainer/issues/2) as a concrete example — a real 3-year, multi-analyst, published scRNA-seq project with a filled-in compbio template.

---

## Context: two complementary tools

### The compbio project documentation template

Adapted from the [original software project template](https://github.com/K-Dense-AI/claude-scientific-skills/blob/main/scientific-skills/markdown-mermaid-writing/templates/project_documentation.md) with these substitutions:

| Software template | Compbio template |
|---|---|
| Quick Start / Install | Experimental design / Sample table |
| API Reference | Script reference with arguments and I/O |
| Configuration (env vars) | Analysis parameters (QC cutoffs, thresholds) |
| Deployment | Manuscript and data submission (GEO, SRA) |
| Contributing guidelines | Decision log (PI/analyst communication) |
| Single architecture diagram | Methods comparison with selection rationale |

The template is designed for iterative projects spanning months or years with a PI, where the documentation must capture not just the final state but the decision trail — what was tried, what was rejected, and why.

### visual-explainer

A skill with 5 slash commands that generates self-contained, interactive HTML pages with Mermaid diagrams, styled tables, and dashboards. Commands:

| Command | Git dependency | Content source |
|---|---|---|
| `/generate-web-diagram` | None | User input / any topic |
| `/fact-check` | Optional | Any HTML or markdown file |
| `/plan-review` | Partial | Markdown plan + codebase files |
| `/project-recap` | **Required** | `git log`, `git status`, commit history |
| `/diff-review` | **Required** | `git diff` between branches |

---

## What works now (no changes needed)

### `/generate-web-diagram` — the workhorse

This command is fully content-agnostic. Every Mermaid diagram type recommended in the compbio template maps to a supported visual-explainer rendering:

| Compbio template recommends | Mermaid type | Clark project example |
|---|---|---|
| Analysis workflow | Flowchart | CellRanger → Seurat Method 4 → subclustering → annotation → 7 downstream branches |
| Methods comparison | Flowchart | 4 cell cycle handling methods, Method 4 selected (highlighted), others grayed |
| Project timeline | Timeline | 2022–2025 milestones across 4 sections |
| Sample relationships | ER diagram | 9 samples × 3 conditions × 3 replicates |
| QC filtering cascade | Sankey | ~26 initial clusters → remove 3 → recluster → annotate |
| Analysis dependencies | Flowchart/Gantt | Which downstream analyses depend on which RDS objects |

Concrete Clark examples that could be rendered today:
- The methods comparison flowchart (already in the template as Mermaid)
- The primary analysis workflow (already in the template as Mermaid)
- The project timeline (already in the template as Mermaid)
- A CellChat analysis summary across C/D/R conditions with comparative results
- A script dependency diagram showing which scripts feed which downstream analyses
- A sample processing status dashboard

### `/fact-check` — verify generated pages

Works on any HTML or markdown. Could verify that a generated Clark dashboard correctly reflects the actual number of clusters, cell types, samples, scripts, etc.

---

## What doesn't work as-is

### `/project-recap` — git-centric assumptions

This command runs `git log --since=<window>` to understand recent activity. For the Clark project:
- The analysis spans 2022–2025 but git commits don't track every analysis step
- Key decisions were made in PI meetings and emails, recorded in the decision log — not in commit messages
- Multiple analysts (Paul, Vishal) worked on different machines; not all work is in git
- The compbio template's decision log table is a richer source of project history than git log

### `/diff-review` — code review oriented

Designed for reviewing git diffs between feature branches. Comp bio projects don't typically use feature branches for analysis iterations. The Clark project's evolution (Method 1 → Method 4 → subclustering → downstream) is tracked in directory structure and the decision log, not branches.

### `/plan-review` — partially applicable

Could compare a planned analysis (from the compbio template) against actual scripts and outputs on disk. But the prompt assumes software project structure (imports, test files, API endpoints) rather than analysis artifacts (RDS files, marker tables, plots).

---

## The gap: `/bio-project-recap`

### What it would do

Read a filled-in `PROJECT_DOCUMENTATION_compbio.md` and generate a visual HTML dashboard — the same styled, interactive output as `/project-recap` but grounded in the analysis notebook instead of git history.

### What sections of the compbio template feed which dashboard panels

| Template section | Dashboard panel | Clark example |
|---|---|---|
| Project overview + Key resources | **Project identity card** | Clark scRNA-seq, PI: Daniel Clark, Published, GSE244633 |
| Experimental design | **Sample status matrix** | 9 samples, 3 conditions, 10X 3' v3 |
| Analysis parameters | **Parameters reference card** | QC cutoffs, clustering resolution, CC gene removal |
| Methods comparison | **Methods decision tree** (Mermaid) | 4 methods evaluated, Method 4 selected with rationale |
| Primary analysis path | **Pipeline diagram** (Mermaid) | End-to-end flowchart with decision points highlighted |
| Downstream analyses | **Analysis branch status** | 7 downstream branches, each with completion status |
| Decision log | **Decision timeline** | 12 decisions from 2022-08 to 2025-05, who decided, why |
| Key data objects | **Data object inventory** | 5 primary RDS files, 3 cell-type subsets, 2 ShinyCell apps |
| Script reference | **Script catalog** | 19 active scripts, 5 deprecated, with I/O |
| Output inventory | **File count dashboard** | ~50 xlsx, ~150 plots, ~30 RDS, ~10 HTML reports |
| Manuscript & submission | **Publication status** | Submitted, GEO public, revision complete |
| Project timeline | **Interactive timeline** (Mermaid) | 4 years of milestones |
| Environment | **Reproducibility card** | R/Seurat/Bioconductor versions, compute environment |

### What it would take

One new file: `prompts/bio-project-recap.md`

The prompt would instruct the agent to:
1. Accept a path to a `PROJECT_DOCUMENTATION_compbio.md` (or detect it in the current directory)
2. Parse the structured markdown tables (sample table, parameters, methods comparison, decision log, script reference, output inventory)
3. Optionally scan the analysis directory to verify file counts and detect new/modified files since the doc was last updated
4. Generate a styled HTML dashboard using the existing rendering pipeline (templates, CSS patterns, Mermaid theming, responsive nav)
5. Highlight staleness — sections where the doc may be out of date (e.g., file counts don't match disk, new directories not documented)

No changes to the rendering engine, templates, references, or design system. The existing `/project-recap` prompt is 200+ lines; `/bio-project-recap` would be similar in structure but read different inputs.

---

## Workflow for comp bio projects

```
Analyst maintains PROJECT_DOCUMENTATION_compbio.md
  (living document, updated as project progresses)
         │
         ├── Before PI meeting
         │     └── /bio-project-recap → HTML dashboard
         │         (project state, decision history, what's pending)
         │
         ├── Ad-hoc diagrams
         │     └── /generate-web-diagram → HTML page
         │         (pipeline flowchart, methods comparison, sample design)
         │
         ├── Verify accuracy
         │     └── /fact-check → annotated HTML
         │         (check claims against actual files on disk)
         │
         └── Handoff to new analyst
               └── /bio-project-recap → HTML dashboard
                   (full project context in one visual page)
```

---

## Recommendations

1. **Use `/generate-web-diagram` now** — no modifications needed. The Clark project's Mermaid diagrams (methods comparison, analysis workflow, timeline) can be rendered into styled HTML pages today.

2. **Create `/bio-project-recap`** — a single new prompt file in `prompts/` that reads the compbio template and generates a visual dashboard. This is the highest-value addition for comp bio users.

3. **Consider adapting `/plan-review`** — modify to compare the compbio template's planned analyses against actual scripts and outputs on disk. Lower priority than `/bio-project-recap`.

4. **Do not force `/project-recap` or `/diff-review`** onto comp bio projects — these are git-centric by design and the compbio template is the better source of truth for analysis projects.

5. **Document the two-repo workflow** — the compbio template (from claude-scientific-skills) and visual-explainer serve different roles and should be used together. The template captures content; visual-explainer renders it.
