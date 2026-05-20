---
description: Generate a visual HTML dashboard from a filled-in compbio project documentation template — experimental design, analysis pipeline, decision history, and project status at a glance
---
Load the visual-explainer skill, then generate a comprehensive visual compbio project recap as a self-contained HTML page.

Follow the visual-explainer skill workflow. Read the reference template, CSS patterns, and mermaid theming references before generating. Use a warm editorial or paper/ink aesthetic with muted blues and greens, but vary fonts and palette from previous diagrams.

**Input detection** — determine the project documentation file from `$1`:
- If `$1` is a path to a file, use that file
- If no argument, search the current directory for files matching `PROJECT_DOCUMENTATION*.md` or `*compbio*.md` (case-insensitive). If multiple matches, list them and ask the user to choose. If exactly one match, use it.

**Critical: no summarization.** The dashboard must be a *superset* of the documentation file — everything in the source document appears in the HTML, enhanced with better presentation, plus any additional insights from the directory scan. Specific rules:
- **Every row** in every markdown table must appear in the corresponding dashboard table or card. Do not summarize, condense, combine, or omit rows. A 12-entry decision log produces 12 timeline entries. A 15-row script table produces 15 script entries.
- **Every analysis sub-section** (markers, DE, enrichment, CCI, trajectory, subclustering, etc.) gets its own card with the full parameter details from the source, not a rolled-up summary.
- **Full rationale text** in the decision log — reproduce the complete "Context / rationale" cell, do not truncate to a one-line summary.
- **The directory structure** code block must be reproduced verbatim in the output inventory section.
- **All Mermaid diagrams** from the source document must be rendered in the dashboard — do not drop any.
- If the source document has content not covered by the dashboard sections below (e.g., External datasets, Visualizations, References, Annotation details), add those as additional sections rather than dropping them.

**Data gathering phase** — read the documentation file in full and parse these structured sections. The template uses markdown tables and Mermaid code blocks — parse them generically by structure (header rows, data rows, fenced blocks), not by hardcoded column names. Extract:

1. **Project overview + key resources table** — project description, organism, technology, biological question, PI, analyst, status, publication/accession, repository links
2. **Experimental design + sample table** — conditions, groups, N per group, technology, batches, sample-level notes. Also the experimental conditions summary table.
3. **Analysis parameters table** — QC cutoffs, clustering resolution, DE test, significance thresholds, and which script applies each parameter
4. **Methods comparison table + Mermaid diagram** — all methods evaluated, their status (Selected / Rejected / Exploratory), and the rationale for the primary selection. Extract the Mermaid flowchart if present.
5. **Primary analysis path + Mermaid diagram** — the end-to-end analysis workflow description and its Mermaid flowchart. Also annotation strategy and key data objects tables.
6. **Downstream analyses tables** — marker analysis, differential expression, functional annotation, cell-cell interaction, and any additional analysis sub-sections. Capture analysis type, method, grouping, parameters, and output locations.
7. **Key data objects table** — RDS/h5ad/h5 files with paths, descriptions, and metadata columns
8. **Decision log table** — date, decision, who decided, and context/rationale for each entry
9. **Script reference table** — active scripts with purpose, arguments, inputs, outputs. Also deprecated scripts if present.
10. **Output inventory + directory structure** — the directory tree (from the fenced code block) and file counts by type table
11. **Manuscript & data submission tables** — manuscript status/version, supplementary materials, GEO/SRA/dbGaP accession and embargo status
12. **Environment + key package versions** — R version, Seurat version, Bioconductor, compute environment, and the per-package version table
13. **Project timeline Mermaid diagram** — the timeline diagram showing milestones chronologically

**Optional directory scan** — if the project directory is accessible (i.e., the documentation file's parent directory or a project root can be identified), scan for:
- Actual file counts by type (`.xlsx`, `.png`, `.pdf`, `.rds`, `.h5ad`, `.h5`, `.html`, `.log`) vs documented counts in the Output inventory table
- Last-modified dates on key data objects listed in the Key data objects table
- New directories or files not documented in the Output inventory directory structure
- Missing files that the template references (scripts, data objects, output paths) but that don't exist on disk
- Template sections that still contain placeholder text (bracketed `[text]` patterns)

If the project directory is not accessible, skip this scan entirely — the dashboard works from the documentation file alone.

**Verification checkpoint** — before generating HTML, produce a structured fact sheet of every claim you will present in the dashboard:
- Every quantitative figure: sample counts, cluster counts, file counts, parameter values, decision log entry counts
- Every gene name, cell type, method name, package version, and accession number you will reference
- Every pipeline step, analysis branch, and status indicator
- For each, cite the source: the template section, table row, or Mermaid block where you read it. If from directory scan, cite the scan result.
Verify each claim against the documentation. If something cannot be verified or contains placeholder text, mark it as uncertain rather than stating it as fact. This fact sheet is your source of truth during HTML generation — do not deviate from it.

**Dashboard sections** — the HTML page should include:

1. **Project identity card** — from Project overview: organism, technology, biological question, PI, analyst(s), current status (active/published/archived), publication DOI or accession numbers. *Visual treatment: this is the visual anchor — use hero depth (elevated container, larger padding 20-24px type, subtle accent-tinted background).* Include key resource links as styled badges.

2. **Experimental design** — sample matrix visualization from the sample table. Show condition/group summary with N per group, technology, and batch structure. Use a compact grid or table with color-coded condition labels. Flag any samples with notes (e.g., low cell recovery).

3. **Analysis pipeline** — render the Primary analysis path Mermaid flowchart with zoom controls (+/−/reset buttons), Ctrl/Cmd+scroll zoom, and click-and-drag panning (grab/grabbing cursors). See css-patterns.md "Mermaid Zoom Controls" for the full pattern. Include the analysis parameters as a compact reference panel beside or below the diagram. *Visual treatment: elevated — this is the second visual anchor after the identity card.*

4. **Methods comparison** — render the Methods comparison Mermaid flowchart with zoom controls. If no Mermaid diagram exists in the template, generate one from the methods table. Highlight the selected/primary method with accent styling, gray out rejected and exploratory alternatives. Show the selection rationale prominently.

5. **Downstream analysis status** — one card per downstream analysis sub-section from the source document. Every sub-section (markers, DE, enrichment, CCI, trajectory, subclustering, etc.) gets its own card. Each card includes:
   - Status indicator (active, complete, exploratory, deprecated)
   - **Every row** from that sub-section's table — each analysis run as a line item with method, grouping, comparison, parameters, and output location. Do not roll up multiple rows into a summary.
   - Notable findings or PI decisions that shaped the analysis
   Use styled cards with left-border color accents by status.

6. **Decision timeline** — from the Decision log table. **Every row** in the decision log becomes a timeline entry — do not skip, merge, or summarize entries. Each entry shows the full date, full decision text, who decided, and the **complete rationale text** from the source (not a truncated summary). Styled vertical timeline. Color-code entries by decision type:
   - **Method selection** (blue) — choosing or switching an analysis method
   - **Parameter choice** (green) — setting QC cutoffs, thresholds, resolution
   - **PI directive** (purple) — decisions driven by PI feedback or biological interpretation
   - **Deprecation** (amber) — abandoning a previous approach
   - **Data/design** (teal) — sample exclusions, batch decisions, reference changes
   Infer the type from the decision text and context. If ambiguous, default to a neutral gray.

7. **Data objects & scripts** — two collapsible tables that include **every row** from the source tables:
   - **Key data objects**: every entry from the Key data objects table — file name, path, description, key metadata columns, and last-modified date (if directory scan ran). Do not omit entries.
   - **Active scripts**: every entry from the Script reference table — name, purpose, key arguments, input → output. Show deprecated scripts (every row from the deprecated scripts table) in a collapsed sub-section.

8. **Output inventory** — file count KPI cards (one per file type from the file counts table) using the KPI card pattern from css-patterns.md, with large hero numbers. Include the "Primary locations" column from each row. Below the cards, render the **complete directory structure** code block from the template verbatim (preserve all directories, comments, and nesting). If directory scan ran, show actual vs documented counts and flag mismatches with amber indicators.

9. **Publication & submission status** — from Manuscript and Data submission sections. Styled badges for:
   - Manuscript stage (draft / submitted / published) with version
   - GEO status (not submitted / private / public) with accession
   - SRA status with accession
   - Embargo dates if applicable
   Compact card layout. Omit this section entirely if the template section contains only placeholder text.

10. **Project timeline** — render the Project timeline Mermaid diagram with zoom controls (+/−/reset buttons), Ctrl/Cmd+scroll zoom, and click-and-drag panning. Same zoom pattern as sections 3 and 4.

11. **Environment & reproducibility** — from the Environment section. Show R version, Seurat version, Bioconductor version, compute environment, and pipeline version as a compact info panel. Below it, render the **complete key package versions table** with every row (package, version, used for). Include renv.lock and sessionInfo paths if documented.

12. **Additional sections** — render any remaining sections from the source document that aren't covered above. Common examples:
    - **External datasets** — table of external datasets with accession, publication, purpose, location
    - **Visualizations** — table of generated plots with type, description, location, status. Include filtering notes.
    - **Annotation details** — table of annotation steps with tool, reference, notes
    - **References** — footnote citations
    Each gets its own collapsible section with the complete table contents. Do not drop sections just because they aren't in the numbered list above.

13. **Staleness report** (only if directory scan ran) — amber-tinted cards flagging:
    - Documented files not found on disk (with the path that was expected)
    - Undocumented directories found on disk
    - File count mismatches (documented vs actual) by type
    - Template sections with placeholder text still present (bracketed `[text]` patterns)
    - Key data objects with stale last-modified dates (older than the most recent decision log entry)
    Use severity indicators: red left border for missing files, amber for mismatches, blue for informational (placeholder text, undocumented dirs). Omit this section entirely if no issues were found.

**Visual hierarchy**: Sections 1 and 3 should dominate the viewport on load (hero depth for identity, elevated for pipeline diagram). The decision timeline (6) should be visually prominent as the narrative backbone. Sections 7-12 are reference material and should feel lighter (flat or recessed depth, compact layout, collapsible where appropriate). Section 13 (staleness report) is an alert layer — visually distinct with amber background.

Overflow prevention on any side-by-side or grid-based sections: apply `min-width: 0` on all grid/flex children and `overflow-wrap: break-word`. Never use `display: flex` on `<li>` for marker characters — use absolute positioning instead (see css-patterns.md Overflow Protection).

Include responsive section navigation. On desktop, use a sidebar nav. On mobile, use a horizontal scrollable bar. Use a warm, approachable visual language: muted blues for pipeline and methods, greens for active/selected status, amber for deprecation and staleness warnings, purple for PI directives, teal for data/design decisions. Write to `~/.agent/diagrams/` and open in browser.

Ultrathink.

$@
