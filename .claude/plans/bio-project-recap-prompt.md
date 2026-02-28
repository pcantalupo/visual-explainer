# Plan: Create `/bio-project-recap` prompt

## Context

GitHub Issue [pcantalupo/visual-explainer#2](https://github.com/pcantalupo/visual-explainer/issues/2) asks whether this repo can be used for computational biology projects. The feasibility analysis (`.claude/plans/compbio-feasibility.md`) concluded that the highest-value addition is a `/bio-project-recap` command that reads a filled-in [compbio project documentation template](https://github.com/pcantalupo/claude-scientific-skills/blob/main/scientific-skills/markdown-mermaid-writing/templates/compbio_project_documentation.md) and generates a visual HTML dashboard — the same output quality as `/project-recap` but grounded in the analysis notebook instead of git history.

## What to create

**One new file:** `prompts/bio-project-recap.md`

No changes to SKILL.md, templates, references, or the rendering engine.

## Prompt structure

Follow the exact pattern of the existing prompts (`project-recap.md`, `diff-review.md`, `plan-review.md`):

1. **YAML frontmatter** with `description:`
2. **Opening line**: Load the visual-explainer skill, then generate...
3. **Aesthetic direction**: Warm editorial or paper/ink, vary from previous diagrams
4. **Input detection** (`$1`): Path to a `PROJECT_DOCUMENTATION_compbio.md` file. If no argument, search current directory for files matching `PROJECT_DOCUMENTATION*.md` or `*compbio*.md`
5. **Data gathering phase** — parse the compbio template's structured sections:
   - Project overview + key resources table
   - Experimental design + sample table
   - Analysis parameters table
   - Methods comparison table + Mermaid diagram
   - Primary analysis path + Mermaid diagram
   - Downstream analyses tables
   - Key data objects table
   - Decision log table
   - Script reference table
   - Output inventory + directory structure
   - Manuscript & data submission tables
   - Environment + key package versions
   - Project timeline Mermaid diagram
6. **Optional directory scan** — if the project directory is accessible, scan for:
   - Actual file counts by type vs documented counts
   - Last-modified dates on key data objects
   - New directories/files not documented in the template
   - Missing files that the template references
7. **Verification checkpoint** — same pattern as other prompts: structured fact sheet of every claim, cite the template section or directory scan that produced it
8. **Dashboard sections** (the HTML page):
   1. **Project identity card** — from Project overview: what organism, technology, biological question, PI, status, publication/accession. Hero depth treatment.
   2. **Experimental design** — sample matrix visualization from the sample table. Condition/group summary with N per group.
   3. **Analysis pipeline** — render the Primary analysis path Mermaid flowchart with zoom controls. This is the visual anchor.
   4. **Methods comparison** — render the Methods comparison Mermaid flowchart. Highlight the selected method, gray alternatives.
   5. **Downstream analysis status** — card grid showing each downstream branch (markers, DE, CCI, etc.) with status indicators and key outputs.
   6. **Decision timeline** — from the Decision log table. Styled timeline with date, decision, who decided, rationale. Color-code by decision type (method selection, parameter choice, PI directive, deprecation).
   7. **Data objects & scripts** — compact tables of key RDS/h5ad files and active scripts. Collapsible.
   8. **Output inventory** — file count KPI cards + directory tree. Flag staleness if directory scan found discrepancies.
   9. **Publication & submission status** — from Manuscript and Environment sections. Badges for GEO/SRA status, manuscript stage.
   10. **Project timeline** — render the Timeline Mermaid diagram with zoom controls.
   11. **Staleness report** (if directory scan ran) — amber cards flagging: documented files not found on disk, undocumented directories, file count mismatches, template sections with placeholder text still present.
9. **Deliver** — write to `~/.agent/diagrams/` and open in browser. Same pattern as all other prompts.
10. **End with** `Ultrathink.` and `$@`

## Key design decisions

- **No git dependency**: The prompt reads the compbio template markdown, not git history. Git commands are not used.
- **Directory scan is optional**: The prompt works even if the project directory isn't accessible (e.g., the template file is on a different machine). When accessible, it adds a staleness/verification layer.
- **Reuses all existing rendering infrastructure**: Templates, CSS patterns, Mermaid theming, responsive nav — nothing new needed.
- **Template-agnostic parsing**: The prompt instructs the agent to parse markdown tables and Mermaid blocks generically, not hardcoded to specific column names. This means it works with variations of the compbio template.

## Files to modify

| File | Action |
|---|---|
| `prompts/bio-project-recap.md` | **Create** — the new prompt |

## Verification

1. After creating the file, run `/bio-project-recap /path/to/clark/PROJECT_DOCUMENTATION_compbio.md` against the Clark project
2. Verify the HTML opens in browser with all sections populated
3. Check that Mermaid diagrams (methods comparison, analysis workflow, timeline) render with zoom controls
4. Check responsive nav works (sidebar on desktop, horizontal bar on mobile)
5. Verify the decision timeline shows all 12 entries from the Clark decision log
