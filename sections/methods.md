<!-- sections/methods.md — WRITE YOUR PROSE HERE -->
<!-- Use [@citekey] for citations. Do NOT add YAML or R code. -->

# Methods {#sec-methods}

*To be written.* How the FLoRA dataset is constructed and how this report
processes it.

## Inclusion Criteria

*To be written.* Studies must self-identify as a replication or reproduction,
identify a specific target study, and replicate a study or experiment (not just a
single association).

## Coding of Outcomes

### Replication Outcomes

*To be written.* Outcomes are coded as **Successful**, **Failed**, or **Mixed**
based on how the replication authors characterise their results.

### Reproduction Outcomes

*To be written.* Reproductions are coded along two dimensions: computational
success (Computationally Successful vs. Computational Issues) and robustness
(Robust, Robustness Challenges, or Robustness Not Checked).

## Data Sources and Contributions

*To be written.* Description of how entries enter FLoRA — community submissions,
large-scale projects, systematic searches.

## Data Pipeline for This Report

*To be written.* This report is rendered via a GitHub Actions workflow that
downloads the latest `flora.csv` from the
[FReD-data repository](https://github.com/forrtproject/FReD-data) daily at
07:00 CEST and rebuilds this page. All code is open and reproducible.
