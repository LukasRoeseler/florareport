# FLoRA Report

**The FORRT Library of Reproduction and Replication Attempts: Documenting the Reproducibility and Replicability of the Sciences and Humanities**

A continuously updated, open-access report on the state of replication and reproduction across the sciences and humanities. Built with [Quarto](https://quarto.org/), powered by the [FLoRA](https://forrt.org/replications/) dataset, and deployed daily via GitHub Pages.

**Live report:** [lukasroeseler.github.io/florareport](https://lukasroeseler.github.io/florareport/)

---

## Contributing text

All prose lives in plain Markdown files inside `sections/`. You do **not** need to touch `index.qmd` — it only contains the page skeleton and R code for figures and tables.

| File | Content |
|---|---|
| `sections/abstract.md` | Abstract (~150–300 words) |
| `sections/lay-summary.md` | Plain-language summary (~150–250 words) |
| `sections/theory.md` | Background, replication crises, FReD → FLoRA, definitions |
| `sections/methods.md` | Inclusion criteria, outcome coding, data sources, pipeline |
| `sections/results.qmd` | Results prose + R code for figures and tables |
| `sections/discussion.md` | Patterns in replicability and reproducibility, implications |
| `sections/limitations.md` | Caveats and scope |
| `sections/conclusion.md` | Wrap-up and call to action |
| `sections/social-responsibility.md` | Equity, accessibility, community |
| `sections/acknowledgments.md` | Thanks and AI use statement |
| `sections/funding.md` | Funding sources |

### Writing conventions

Use `[@citekey]` for parenthetical citations and `@citekey` for narrative citations (e.g., `[@roseler2024replication]` renders as "(Röseler et al., 2024)"). Keys are defined in `references.bib`. Use `**bold**`, `*italic*`, and `##` / `###` for sub-headings. Do not add YAML headers or R code to the `.md` section files.

### Google Docs workflow

For collaborative editing, copy a section file's content into Google Docs, write using the conventions above, then copy the result back into the `.md` file. For cleaner conversion, download from Google Docs as `.docx` and run:

```bash
pandoc -f docx -t markdown --wrap=none "Theory.docx" -o sections/theory.md
```

### Adding references

Add new entries to `references.bib` in BibTeX format. If using Zotero, export from a shared group library with the "Better BibTeX" plugin for consistent citation keys.

---

## Repository layout

```
florareport/
├── index.qmd                       # Page skeleton + R setup (don't edit prose here)
├── sections/                       # Prose files — edit these
│   ├── abstract.md
│   ├── lay-summary.md
│   ├── theory.md
│   ├── methods.md
│   ├── results.qmd                 # Results prose + R chunks for plots/tables
│   ├── discussion.md
│   ├── limitations.md
│   ├── conclusion.md
│   ├── social-responsibility.md
│   ├── acknowledgments.md
│   └── funding.md
├── references.bib                  # BibTeX bibliography
├── apa.csl                         # APA 7 citation style (download once, see below)
├── logo.svg                        # FLoRA logo (download once, see below)
├── styles.css                      # Purple theme CSS
├── _quarto.yml                     # Quarto project config
├── data/
│   └── flora.csv                   # Downloaded fresh at each build
├── figures/                        # Generated and static figures
├── .github/workflows/
│   └── render.yml                  # CI: download data → render → deploy to Pages
└── README.md
```

---

## Local setup

```bash
# Clone the repo
git clone https://github.com/lukasroeseler/florareport.git
cd florareport

# Download files that aren't committed (one-time setup)
curl -L -o apa.csl https://raw.githubusercontent.com/citation-style-language/styles/master/apa.csl
curl -L -o logo.svg https://raw.githubusercontent.com/forrtproject/flora-explorer/refs/heads/main/logo.svg

# Download the dataset
mkdir -p data
curl -L -o data/flora.csv https://raw.githubusercontent.com/forrtproject/FReD-data/refs/heads/main/output/flora.csv

# Render
quarto render
```

The rendered site will be in `_site/`. Do not place the project in a cloud-synced folder (e.g., sciebo, OneDrive, Dropbox) — the sync client will lock `_site/` files and cause build errors.

---

## Automated builds

The GitHub Actions workflow (`.github/workflows/render.yml`) rebuilds and deploys the report on three triggers:

| Trigger | When |
|---|---|
| Cron schedule | Daily at **07:00 CEST** (05:00 UTC summer / 06:00 UTC winter) |
| Push to `main` | Whenever any source file, section, or data file changes |
| Manual | Via the **Actions** tab → "Run workflow" |

At each build, the workflow downloads the latest `flora.csv` from the [FReD-data repository](https://github.com/forrtproject/FReD-data), fetches `apa.csl` and `logo.svg` if they aren't committed, renders the Quarto site, and deploys it to GitHub Pages.

The data source URL defaults to `https://raw.githubusercontent.com/forrtproject/FReD-data/refs/heads/main/output/flora.csv` and can be overridden via the repository variable `FLORA_CSV_URL` (Settings → Secrets and variables → Actions → Variables).

---

## Citation

Please cite this report as:

> Röseler, L. et al. (2025). *The FORRT Library of Reproduction and Replication Attempts: Documenting the Reproducibility and Replicability of the Sciences and Humanities.* https://lukasroeseler.github.io/florareport/

And cite the underlying dataset as:

> Wallrich, L., Röseler, L., et al. (2026). *FORRT Library of Replication Attempts (FLoRA)* [Data set]. OSF. https://doi.org/10.17605/OSF.IO/9R62X

## License

Code: MIT. Report text and figures: CC BY 4.0. FLoRA data: CC BY 4.0 (© the FLoRA contributors).