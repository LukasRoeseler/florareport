<!-- sections/pipeline.md — DRAFT -->

<!-- Use [@citekey] for citations. Do NOT add YAML headers. -->

# The FLoRA Extraction and Validation Pipeline {#sec-pipeline}

The credibility revolution in the empirical sciences has produced a
growing literature of replication and reproduction attempts. Keeping
FLoRA — the FORRT (Framework for Open and Reproducible Research
Training) Library of Reproduction and Replication Attempts — up to date
with this literature manually is no longer feasible at the rate at which
it accumulates. Our goal is a **comprehensive database of replication
and reproduction studies linked to their target studies**, maintained
continuously and openly. To approach this goal we run a four-stage
automated pipeline that discovers candidate papers, filters out false
positives, extracts the link to the original study and the replication
outcome, and finally routes each extracted record through a crowdsourced
validation step before it enters FLoRA.

The four stages are independently runnable, communicate exclusively
through versioned CSV files, and cache every external API call so that
re-runs are cheap. @fig-pipeline-overview shows the overall flow.

```{mermaid}
%%| label: fig-pipeline-overview
%%| fig-cap: "The four-stage FLoRA extraction and validation pipeline. Each stage reads one CSV and writes a richer one; the validation app is the only stage with a human-facing interface."
flowchart LR
    A[("External sources<br/>OpenAlex · Semantic Scholar<br/>Bob Reed list · I4R reports")]
    B["Stage 1: Search<br/>(discover)"]
    C["Stage 2: Filter<br/>(classify)"]
    D["Stage 3: Extract<br/>(link + outcome)"]
    E["Stage 4: Validate<br/>(human + LLM)"]
    F[("FLoRA database")]

    A --> B --> C --> D --> E --> F

    B -. candidates.csv .-> C
    C -. filtered.csv .-> D
    D -. extracted.csv .-> E
    E -. validated.csv .-> F
```

## Stage 1 — Search (Discovery)

-   The first stage casts a wide net across academic literature with the
    goal of **high recall over precision**: it is preferable to include
    false positives here than to miss a real replication, because Stage
    2 will filter them out
-   **Primary sources** are queried programmatically:
    -   **OpenAlex** [@priem2022openalex] is searched via its
        `title_and_abstract.search` filter for the phrases *replication
        of*, *direct replication*, *close replication*, *conceptual
        replication*, *replication study*, *reproduction study*, *we
        replicated*, *attempts to replicate*, and *registered
        replication report*
    -   **Semantic Scholar** [@kinney2023semanticscholar] provides
        supplementary coverage
-   **Secondary sources** are curated lists scraped through pluggable
    loader functions:
    -   The **Replication Network list** maintained by Bob Reed
        [@reed2020replicationnetwork] for economics
    -   The **Institute for Replication (I4R) reports**
        [@brodeur2024i4r] for multi-disciplinary coverage
-   Results from all sources are merged and deduplicated in two passes:
    first by exact (cleaned) DOI, then by fuzzy title match using
    `rapidfuzz` with a token-sort ratio of ≥ 90
-   DOIs already present in the FLoRA entry sheet are dropped before
    output, so downstream stages never re-process records already in the
    database
-   The stage writes `candidates.csv` with the columns `doi_r`,
    `title_r`, `abstract_r`, `year_r`, `authors_r`, `journal_r`,
    `url_r`, `openalex_id_r`, and `source`

## Stage 2 — Filter (Classification)

-   The second stage takes the candidate pool from Stage 1 and produces
    a clean, classified set of genuine replication and reproduction
    studies; the design goal here flips to **high precision** because
    Stage 3 is computationally expensive
-   A **rule-based classifier** runs first and resolves the easy cases
    using keyword patterns in `title_r` and `abstract_r`:
    -   Replication indicators (e.g. *direct replication*, *we
        replicated*, *registered replication report*) → `replication`
    -   Reproduction indicators (e.g. *we reproduced*, *computational
        reproduction*, *reanalysis of*) → `reproduction`
    -   False-positive indicators (e.g. *replication crisis*, *review of
        replications*, *meta-analysis of replication*) →
        `false_positive`
    -   Rows with no author–year citation pattern anywhere in the
        abstract (e.g. no `Smith (2020)`) are flagged as `needs_review`
        — the absence of any named original is a strong signal of a
        false positive
-   An **LLM classifier** is invoked **only** for rows the rules left as
    `needs_review`, prompting Gemini Flash with the title and abstract
    and returning `filter_status`, a supporting evidence quote, and a
    categorical confidence label (`high` / `medium` / `low`)
-   False positives are **not deleted** — they are written to
    `filtered.csv` with `filter_status = false_positive` so that Stage 3
    can skip them cleanly and so that downstream audits can measure
    false-positive rates
-   @fig-filter shows the decision flow

```{mermaid}
%%| label: fig-filter
%%| fig-cap: "Stage 2 classification flow. The rule-based pass resolves the easy cases; only papers flagged as needs_review are sent to the LLM."
flowchart TB
    IN[candidates.csv] --> DEDUP[Title deduplication<br/>fuzzy ≥ 90]
    DEDUP --> RULES{Rule-based<br/>classifier}
    RULES -->|keyword match| LAB1[replication]
    RULES -->|keyword match| LAB2[reproduction]
    RULES -->|keyword match| LAB3[false_positive]
    RULES -->|no author-year<br/>pattern| NR[needs_review]
    NR --> LLM[LLM classifier<br/>Gemini Flash]
    LLM --> LAB4[final filter_status]
    LAB1 --> OUT[filtered.csv]
    LAB2 --> OUT
    LAB3 --> OUT
    LAB4 --> OUT
```

## Stage 3 — Extract (Linking and Outcome Coding)

-   For every confirmed replication or reproduction, Stage 3 answers two
    questions:
    1.  **Which original study does this paper replicate?** (linking)
    2.  **What was the outcome?** (success / failure / mixed /
        uninformative / descriptive)
-   The stage first classifies each paper's `original_match_type` so
    that the routing of work can be tailored to the structure of the
    paper:
    -   `single_original` — one specific target study
    -   `multiple_match` — several candidates with the same author/year
        that must be disambiguated
    -   `multiple_original` — the paper genuinely targets several
        independent originals and will be expanded to *N* rows in the
        output
-   Linking proceeds through up to seven steps, but most papers exit
    early — the design target is **\~60 % resolution from abstract +
    references alone**, before any PDF is downloaded:
    1.  **LLM abstract + reference matching** — title, abstract, and the
        paper's OpenAlex referenced works are sent to the LLM; a
        high-confidence return resolves the link immediately
    2.  **OpenAlex author–year requery** — for papers not resolved in
        Step 1, author–year citation patterns are extracted from the
        abstract (eight regex patterns with Unicode surname support) and
        matched against the paper's referenced works with a year
        tolerance of ±1 and fuzzy surname matching
    3.  **Same-author/year disambiguation** — if exactly one
        non-umbrella candidate remains, the link is resolved by simple
        heuristic; "umbrella" framework papers (ManyLabs, Psychological
        Science Accelerator, EEGManyLabs, StudySwap) are guarded out of
        this fast path
    4.  **PDF acquisition** — an eleven-tier waterfall (arXiv → OSF →
        OpenAlex OA URL → Unpaywall direct PDFs → Semantic Scholar →
        CORE → Europe PMC → Unpaywall landing-page scraper → SerpAPI →
        headless Chromium → HTML text fallback), with each tier skipped
        once a PDF is acquired
    5.  **Reference extraction** — `pdfminer.six` parses abstract,
        intro, methods, and reference sections locally; if pdfminer
        returns no references, the PDF is sent to the LLM directly (or
        rendered to images for scanned PDFs)
    6.  **Full-context LLM identification** — title, abstract, candidate
        list, parsed intro/methods, and up to 80 references are sent to
        the LLM, returning `doi_o`, `title_o`, `link_evidence`, and
        `link_confidence`
    7.  **Outcome extraction** — a keyword pass detects clear cases
        (*replicated*, *failed to replicate*, *partial replication*,
        etc.); ambiguous cases are sent to the LLM, returning `outcome`,
        a supporting quote, and a confidence label
-   @fig-extract shows the routing
-   Every LLM call and every external API response is cached on disk
    keyed by `cache_key(doi_r + suffix)` so that re-runs only pay for
    new records; rate limits (OpenAlex 0.1 s, LLM 1 s) and a 3-attempt
    exponential backoff (1 s, 2 s, 4 s) on transient API failures are
    applied uniformly across the stage
-   Persistent failures are written as `api_error` rather than silently
    dropped, so that the validation stage sees them and can route a
    human reviewer to the gap

```{mermaid}
%%| label: fig-extract
%%| fig-cap: "Stage 3 routing. Match-type classification determines whether a paper goes through the shared single/multiple-match pipeline or the multi-original pipeline. Most papers exit at Step 1."
flowchart TB
    IN[filtered.csv row] --> MT{Match-type<br/>classifier}
    MT -->|false_positive| PASS[pass through<br/>no extraction]
    MT -->|single_original<br/>or multiple_match| SH[Shared pipeline]
    MT -->|multiple_original| MP[Multi-original pipeline<br/>→ N rows out]

    SH --> S1["Step 1: LLM<br/>abstract + refs"]
    S1 -->|high confidence| OUT[extracted.csv]
    S1 -->|else| S2["Steps 2-3: author-year<br/>requery + heuristic"]
    S2 -->|resolved| OUT
    S2 -->|else| S4["Steps 4-6: PDF +<br/>full-context LLM"]
    S4 --> S7["Step 7: outcome<br/>keyword + LLM"]
    S7 --> OUT
    MP --> OUT
```

## Stage 4 — Validate (Human + LLM Consensus)

-   The final stage takes each extracted record and routes it to **two
    independent human validators** before it enters FLoRA; the goal of
    Stage 4 is to catch the kinds of errors a single automated extractor
    cannot reliably catch — mis-identified originals, mis-coded
    outcomes, and conceptual extensions that the filter waved through
-   The validation app is implemented as a FastAPI service backed by a
    PostgreSQL database with a static single-page front end; it is
    self-contained and can be deployed as a single web process
-   Each record is presented to a validator alongside the open-access
    URL of the replicating paper (cached from Unpaywall, arXiv, or OSF
    so reviewers can read without leaving the app)
-   Validators answer three structured questions, each with a `correct`
    / `incorrect` response and an optional free-text correction:
    -   **type_check** — does the paper actually report a replication or
        reproduction, rather than a conceptual extension or commentary?
    -   **original_check** — is the proposed original study the right
        one?
    -   **outcome_check** — does the coded outcome (success / failure /
        mixed / uninformative / descriptive) describe what the paper
        found?
-   When both human reviews are in, the **consensus engine** decides
    what happens to the record:
    -   **Both reviewers agree on all three checks and on any
        corrections** → record is marked `validated` and written to the
        validated table; the LLM is also queried for an audit trail, but
        cannot override two agreeing humans
    -   **Both agree on the checks but propose different corrections**
        (e.g. both say the original DOI is wrong but disagree on the
        right replacement) → record is marked `need_review` for an
        administrator to resolve; the engine deliberately refuses to
        arbitrate free-text disagreements
    -   **Reviewers disagree on a check** → the LLM (Gemini Flash) is
        called as a third reviewer; if it agrees with one of the two
        humans, that verdict is recorded; if it produces a three-way
        split or an API error, the record falls back to `need_review`
-   The LLM is configured **conservatively**: when uncertain it votes
    `correct`, biasing the engine toward trusting the upstream
    extraction in genuinely ambiguous cases rather than generating
    spurious disagreement that would inflate the review backlog
-   The decision rule is summarised in @fig-consensus

```{mermaid}
%%| label: fig-consensus
%%| fig-cap: "The Stage 4 consensus engine. The LLM serves as a tiebreaker when the two human reviewers disagree on a check; it is never permitted to override two agreeing humans."
flowchart TB
    START([two human reviews complete]) --> Q1{Checks<br/>agree?}
    Q1 -->|yes| Q2{Corrections<br/>agree?}
    Q1 -->|no| LLM[Call LLM<br/>as tiebreaker]
    Q2 -->|yes| V1[validated<br/>LLM logged for audit]
    Q2 -->|no| NR1[need_review<br/>admin resolves]
    LLM --> Q3{LLM matches<br/>one human?}
    Q3 -->|yes| V2[validated<br/>with matched verdict]
    Q3 -->|three-way split<br/>or API error| NR2[need_review]
```

-   The database carries the operational state across five tables:
    -   `validators` — reviewer identity and gamification state (points,
        level)
    -   `unvalidated` — one row per record under review, carrying JSONB
        blobs for each reviewer and for the LLM
    -   `validation_queue` — per-slot assignments binding a record to a
        specific reviewer; this separates the *record* from the
        *assignment* so that re-reviews after an admin override do not
        rewrite history
    -   `validated` — the final consensus output, ready to export to
        FLoRA
    -   `record_metadata` — provenance and the full extraction trace
        (LLM prompts, OpenAlex candidate lists, parsed PDF sections) for
        any record, retained for audit and for prompt-engineering review

## Operations and Reproducibility

-   The validation app pulls the upstream `extracted.csv` from the
    `flora-extractor` GitHub repository on a nightly cron at 02:00 UTC,
    archives the file as `extracted_DD.MM.YYYY.csv`, and inserts only
    rows whose `pair_id` is not already in the database; the import is
    idempotent by construction
-   The validated output is published back to the FReD-data repository
    [@wallrich2026flora] under CC BY 4.0, and is consumed by the report
    build described in @sec-methods
-   All extraction code, validation code, and the report build itself
    are openly available; the four-stage pipeline can be re-run from any
    commit, and every external API call is cached so re-runs are cheap
    and deterministic up to the cache horizon

## Why a Hybrid Human–LLM Design?

-   A pure-LLM verifier would scale better but would reproduce the same
    errors that Stage 3 makes, because Stage 3 is itself driven by an
    LLM; a pure-human verifier would in principle be most reliable but
    cannot keep pace with the literature at the throughput we need
-   The dual-human plus LLM-tiebreaker design is the standard
    human-in-the-loop adjudication pattern, with one additional
    constraint: the LLM is never given the casting vote against two
    agreeing humans
-   This commitment is the central design choice of the validation stage
    and the reason we are willing to publish the resulting data as the
    canonical FLoRA record: the LLM is a tiebreaker, never an oracle
