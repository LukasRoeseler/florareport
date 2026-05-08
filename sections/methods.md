<!-- sections/methods.md — DRAFT (bullet points) -->

<!-- Use [@citekey] for citations. Do NOT add YAML headers. -->

# Methods {#sec-methods}

FLoRA builds on and extends the FORRT Replication Database [FReD; @roseler2024replication]. While FReD contains statistical data (effect sizes, sample sizes) for quantitative comparison, FLoRA tracks replication and reproduction attempts at the reference level without requiring statistical data, allowing it to cover any field of research with limited resources.

## Inclusion Criteria

-   To be included in FLoRA, a study must meet three conditions:
    -   **Self-identification:** The study must self-identify as a replication or reproduction (e.g., "replication of Author (Year)") before reporting results — replication must be an explicit aim, not merely an incidental result
    -   **Target specification:** The study must identify one or more specific target studies that it replicates or reproduces
    -   **Study-level scope:** The study must replicate a study or experiment, not just a single association or isolated finding
-   Replications can range from close/direct (same methods, same population) to conceptual (testing the same hypothesis with different methods), as long as the above criteria are met
-   The definition is deliberately liberal, following @huffmeier2016typology, to capture the full spectrum of replication attempts
-   **Key distinction between replications and reproductions:**
    -   If **new data** are collected or used (e.g., an additional decade of data), it is a **replication**
    -   If the **same data** are re-analysed to verify the original results, it is a **reproduction**

## Coding of Outcomes

### Replication Outcomes

-   Replication outcomes are coded based on how the replication authors characterise their own results (not re-evaluated by FLoRA coders)
-   Three outcome categories:
    -   **Successful:** The replication authors conclude that the original finding was supported
    -   **Failed:** The replication authors conclude that the original finding was not supported
    -   **Mixed:** The replication authors report a combination of successful and failed results (e.g., some conditions replicated, others did not)
-   This approach respects the original authors' interpretation but means that outcome definitions vary across entries, as different replication teams may apply different criteria for what constitutes success or failure

### Reproduction Outcomes

-   Reproductions are coded along two independent dimensions:
    -   **Computational success:**
        -   *Computationally Successful:* The original results were obtained when re-running the analysis
        -   *Computational Issues:* The original results could not be obtained (e.g., due to missing data, code errors, software version differences)
    -   **Robustness:**
        -   *Robust:* Results hold under reasonable alternative specifications
        -   *Robustness Challenges:* Results change meaningfully under alternative specifications
        -   *Robustness Not Checked:* The reproduction only attempted computational verification without testing alternative specifications

## Data Sources and Contributions

-   FLoRA aggregates entries from multiple sources:
    -   **Large-scale replication projects:** Many Labs 1 [@klein2014manylabs], Many Labs 2 [@klein2018manylabs2], Many Labs 5 [@ebersole2020manylabs5], Registered Replication Reports [e.g., @hagger2016egodepletion; @odonnell2018rrrdijksterhuis; @bouwmeester2017rrr], and the Reproducibility Project: Psychology [@osc2015reproducibility]
    -   **Collaborative initiatives:** The Collaborative Open-science and meta REsearch team [CORE; @roseler2024replication], and the Psychological Science Accelerator [@moshontz2018psa]
    -   **Community submissions:** Individual researchers submit replication findings via the online submission form or the shared Google Sheets spreadsheet
    -   **Systematic searches:** Project leads and research assistants code studies from literature searches, journal issues dedicated to replications, and existing databases such as CurateScience [@lebel2018framework]
    -   **FReD merger:** In late 2023, FLoRA and the FReD project merged, combining both databases
-   Data collection is ongoing via hackathons and workshops at conferences, collaborations with large-scale projects, and literature alerts
-   All entries are validated by at least one person who checks references, effect sizes, sample sizes, and descriptions against the source materials [see @roseler2024replication for validation procedures]

## Data Pipeline for This Report

-   This report is a living document that updates automatically:
    -   The latest `flora.csv` is downloaded from the [FReD-data repository](https://github.com/forrtproject/FReD-data) at each build
    -   A GitHub Actions workflow triggers the build:
        -   **Daily** at 07:00 CEST
        -   **On every push** to the main branch (e.g., when a section file is edited)
        -   **Manually** via the GitHub Actions interface
    -   The report is rendered with [Quarto](https://quarto.org/) using R for data processing and visualisation
    -   The rendered report is deployed to GitHub Pages at <https://lukasroeseler.github.io/florareport/>
-   All code, data, and materials are openly available in the [GitHub repository](https://github.com/lukasroeseler/florareport)
-   The FLoRA dataset itself is licensed CC BY 4.0 [@wallrich2026flora]
