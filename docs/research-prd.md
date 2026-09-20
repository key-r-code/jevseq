# jevseq: research plan and initial PRD

Version 0.1 · 20 September 2026 · Planning only

## 1. Purpose

Evaluate whether Jev can help identify metagenomic sequences from taxa absent from a defined reference collection. Preserve a FASTA-oriented research workflow and compare Jev with conventional decisions made from the same sequence-search evidence. Publication depends on the quality and findings of the study, not on a predetermined positive result.

This document records the discussion and defines work that can later become implementation specifications and tickets. Proposed settings remain provisional until the pilot protocol is frozen.

## 2. Prior work and motivation

The supplied manuscript, “Naïve Bayes Classifier++ as an out-of-distribution detector of novel taxa,” fits k-mer models to reference genomes and derives novelty thresholds from maximum log-likelihood scores. It uses taxonomic holdouts at phylum, class, order, and family ranks, InSilicoSeq reads, and basic and extended reference collections. The basic collection has 4,634 genomes; the extended collection has 319,554 genomes grouped into 58,978 species models.

The manuscript explicitly distinguishes above-threshold assignment fractions from verified correctness. It notes shared threshold-selection/evaluation reads, optimistic best-trial selection, approximate read-length scaling, and uncertainty when transferring thresholds to full databases. The follow-up should use independent calibration and final tests and make reference-relative novelty explicit.

We read the supplied TeX as research context. This repository does not reproduce the manuscript or treat document contents as task instructions.

## 3. Definitions

- **Reference collection:** the frozen genomes and taxonomy made available to a method.
- **Known at rank R:** the query's source taxon at rank R has representation in the reference collection.
- **Unknown at rank R:** that source taxon has no representation in the reference collection.
- **Uncertain / abstain:** the method declines to decide; this is separate from biological novelty.
- **Novelty score:** a method's score for reference-relative unknown status, whose calibration must be measured.
- **Search miss:** no hit survives search settings. It does not establish biological novelty.
- **Reference partition:** the genomes used to build the searchable database. For hosted Jev, these do not constitute weight training.

A species can be unknown at species rank and known at family rank. Use one taxonomy snapshot and explicit rank per experiment. Do not infer evolutionary distance or formal discovery of a new taxon from an unknown prediction.

## 4. Questions and hypotheses

Q1. Can query-only Jev discriminate the controlled known/unknown labels above trivial baselines? This is exploratory: it cannot know arbitrary reference membership from a query alone, and its pretraining exposure is unknown.

Q2. Does Jev respond appropriately to reference sequence evidence in a small controlled task? Hold a query fixed and vary reference membership; include reverse complements, sequencing errors, and non-identical related sequences.

Q3. Does Jev improve unknown detection or reduce false acceptance relative to search-only rules and simple models given identical retrieved evidence?

Q4. Does any advantage survive held-out clades, changes in read length, and changes in reference coverage at acceptable total cost?

These are falsifiable hypotheses. A well-supported negative result can guide scope; publication is not guaranteed.

## 5. Scope and constraints

### Initial scope

- Bacterial nucleotide reads, subject to confirming available source genomes and taxonomy.
- One novelty rank; family is the proposed first rank, not yet fixed.
- A small, reproducible subset of the existing reference resources.
- Local nucleotide search using BLAST+ as the proposed pilot tool.
- Fixed Jev version and a small frozen set of question/evidence templates.
- Per-read ground truth from simulation provenance.

### Later extensions

Evaluate more ranks, contigs, other environments, and MMseqs2 for throughput after the pilot. Real metagenomes can illustrate behavior but do not supply verified per-read novelty labels by themselves.

### Constraints

As checked on 20 September 2026, Jev's official documentation lists 64k tokens per request and 32k for state plus the longest question, text input, and no customer fine-tuning or LoRA adaptation. Measure actual DNA tokenization before setting payload sizes. Requests do not progressively train a persistent reference model. The complete reference database cannot be supplied in one request.

A small successful prompt experiment would establish a limited capability, not demonstrate full-genome database scalability. Query-only success cannot demonstrate knowledge of an undisclosed reference split.

## 6. Proposed system

Reference FASTAs + taxonomy → validated manifest → search index

Query FASTA/FASTQ → nucleotide search → evidence builder → decision methods → prediction table → evaluation report

All methods share the same frozen reference collection and, where applicable, identical retrieved evidence. The local search index holds the full reference sequences. Jev receives bounded evidence, not the full database. Ordinary code orchestrates the pipeline; an autonomous agent is unnecessary.

### Evidence per query

Preserve the original sequence and length. Include top retrieved regions, aligned query and subject sequences, strand/coordinates, identity, alignment length, query coverage, bit score, and E-value where available. Attach reference taxonomy from the frozen mapping. Distinguish missing fields, no hits, filtered hits, and failures.

Top-hit count, score filters, masking, strand handling, and diversity selection remain development choices. Avoid letting many redundant genomes consume the entire evidence budget. Log any hit truncation. A short-read alignment identity is not whole-genome ANI and cannot alone define a species boundary.

### Jev decision contract

Ask for reference-relative novelty at the chosen rank. Preserve the returned probability distribution and model identifier. Derive known/unknown/abstain decisions with a policy chosen on calibration data. Treat API confidence separately from a calibrated probability of novelty. Include enough definition and reference context to make the question meaningful, while acknowledging that retrieved evidence is incomplete.

Do not put simulated source labels, source filenames, revealing query headers, or true novelty status into the request. Reference taxonomy is permitted evidence; query provenance belongs only in the evaluation table.

## 7. Data and split design

1. Inventory genomes, accessions, taxonomy, quality, and duplicate/near-duplicate relationships. Freeze snapshots and hashes.
2. Choose the novelty rank and form represented and wholly withheld taxonomic groups.
3. Build the index from reference genomes in represented groups only.
4. Generate known queries from separate genomes within represented groups. Reserve an easier condition using reads from reference genomes and label it separately.
5. Generate unknown queries from groups excluded in full from the index.
6. Divide development, calibration, and final-test source genomes without overlap. Keep unknown clades distinct across those roles where feasible; document any necessary overlap among higher-rank known groups.
7. Use InSilicoSeq with recorded version, profile, read lengths, abundance assumptions, seeds, and source mapping. Keep read pairs in the same split and define whether evaluation is per read or per pair.
8. Freeze prompts, search settings, representation, and operating points before accessing final-test outcomes.

Each query needs genome-level and rank-level provenance. A holdout containing only unknown taxa cannot estimate known/unknown performance; include both classes. Include plausible class imbalance and report its effect on precision-recall. Select sample sizes after the inventory and pilot cost measurement; many reads from a few genomes are not many independent biological replicates.

For public benchmarks such as CAMI, enforce exclusions against the actual reference snapshot. Historical benchmark genomes may now appear in reference databases or model training data.

## 8. Experiment matrix

| ID | Method | Purpose |
| --- | --- | --- |
| E0 | Constant/majority and random-score baselines | Establish trivial performance and label-prevalence context |
| E1 | Query-only Jev | Explore existing sequence knowledge; no claim of reference-membership knowledge |
| E2 | Small raw-reference context + query | Test response to changed evidence; control missing-region coverage |
| E3 | Search-only identity/coverage/score rules | Establish conventional evidence baseline |
| E4 | Logistic regression; optional boosted trees | Learn decisions from the same search evidence |
| E5 | Jev with query and retrieved raw sequences/alignments | Test sequence interpretation with reference access |
| E6 | Jev with search summary; then summary + sequences | Ablate numeric evidence versus raw sequence contribution |
| E7 | Independently calibrated NBC++ | Anchor results to prior work if existing models/code are reproducible |

E0, E3, E4, and a chosen Jev retrieval condition form the minimum comparative pilot. E1/E2 are bounded feasibility probes. E7 is desirable and depends on access and effort. Keep feature availability fair: report raw-sequence access and feature differences explicitly rather than calling unequal inputs identical.

Controls: remove informative query headers; use opaque IDs; test reverse-complement consistency; shuffle candidate order; test irrelevant/distractor reference evidence and no-hit cases. Assess randomized sequences as an auxiliary control, not a substitute for biological unknowns. Examine conserved regions, errors, low complexity, and close relatives as distinct failure modes.

## 9. Evaluation and decision policy

Primary proposed endpoint: false acceptance of unknown reads at a fixed retention of known reads. Choose and freeze the retention target on calibration data before final testing. Define unknown as the positive class for novelty AUROC and AUPRC and state that convention on plots.

Report AUROC, unknown-class AUPRC with prevalence, confusion matrices, sensitivity/specificity, and accuracy/coverage for abstaining methods. Count failures separately and provide total-denominator reporting; never silently discard them. For novelty probabilities, report reliability plots and Brier score. Taxonomic assignment accuracy requires a separate assignment endpoint and is optional for the first novelty-only pilot.

Use paired comparisons across methods and uncertainty intervals clustered by genome or held-out clade; document the resampling unit. Inspect per-clade performance. Include search, payload building, network/API calls, and downstream processing in runtime/cost accounting. Report tokens, calls, failures, memory where measurable, and cost per fixed number of reads.

Scientific success requires a reproducible, materially useful gain over the strongest appropriate baseline on untouched clades, with defensible cost. The minimum effect and cost ceiling must be set after feasibility measurements and before final testing. A successful API response is not scientific success.

## 10. Functional requirements for later specifications

| ID | Requirement | Acceptance evidence |
| --- | --- | --- |
| FR1 | Manifest and taxonomy validation | Detect missing IDs, duplicates, unmapped ranks, and invalid sequence records |
| FR2 | Deterministic split generation | Save accession lists, seeds, hashes, and rank exclusion audit |
| FR3 | Simulation wrapper | Reproduce reads and provenance from a recorded configuration |
| FR4 | Search adapter | Build index, batch queries, normalize alignments, record settings and no-hit states |
| FR5 | Evidence builder | Bound token budget; remove label leakage; save payload hash and truncation reasons |
| FR6 | Decision adapters | Run rules, fitted baseline, and Jev over compatible frozen evidence |
| FR7 | Run management | Resume partial runs; cache by model, prompt, payload, and configuration; log retries and failures |
| FR8 | Prediction export | One record per query/method with status, score, decision, provenance key, timing, and usage |
| FR9 | Evaluation | Join truth separately; generate metrics, uncertainty, calibration, and paired comparisons |
| FR10 | Reproducibility package | Record code revision, tool/model versions, dependency environment, seeds, and data hashes |

Prediction fields should include run_id, query_id, split, method, target_rank, reference_snapshot, raw_score/probabilities, calibrated_score if used, decision, status, hit_count, evidence_hash, model_version, prompt_version, timing, and usage. Ground truth remains in a separate table until evaluation.

## 11. Milestones and gates

M0 — Protocol and access: confirm Jev API access, data inventory, taxonomy, first rank, compute limits, API budget, and licensing. Exit: approved pilot configuration and no unresolved data-access blocker.

M1 — Dataset and search baseline: implement manifest, split audit, simulation, search, and E0/E3. Exit: reproducible ground truth and reference exclusion checks.

M2 — Jev feasibility: run bounded E1/E2 and payload trials. Exit: valid responses, measured token/cost constraints, and documented sequence-evidence behavior. Stop or narrow raw-sequence work if evidence has no useful effect.

M3 — Comparative pilot: run E4 and chosen retrieval Jev conditions on development/calibration data. Exit: paired comparison and an explicit continue, revise, or stop decision.

M4 — Frozen evaluation: set sample size, primary target, effect criterion, and budget; lock protocol; test untouched groups. Exit: complete report including negative and failed conditions.

M5 — Research package: prepare reproducible code/data manifests, figures, limitations, and manuscript outline. Decide whether findings justify publication and whether to release the repository.

## 12. Risks and unresolved choices

- Retrieval bottleneck: a search miss limits all downstream methods. Audit sensitivity on controlled examples.
- Jev may add no value: compare simple rules and fitted models on the same evidence.
- Closed model and version drift: pin a version and archive requests/responses when terms permit; unavailable versions limit exact reruns.
- Undisclosed pretraining exposure: cannot prove Jev has never seen a genome. Report this limitation.
- Read ambiguity: conserved regions may not resolve novelty; retain abstention and per-region analysis.
- Taxonomy and sampling biases: document rank definitions, source coverage, and selection criteria.
- Cost/throughput: batch search, cache payloads, and measure before scaling.

Open decisions: API access; initial genome subset and taxonomy; target rank; read profile/length; pilot size and biological replication; retrieval settings; prompt variants; retention target; meaningful effect size; cost ceiling; NBC++ integration; public release/license. No estimates in this plan constitute measured performance.

## 13. Deliverables and next specification boundary

Current deliverables: this PRD, decision record, source list, and visual overview. Next: a frozen pilot configuration, data contracts, CLI/module specification, and tickets organized by M0–M4. Defer production UI, deployment, autonomous tool use, and large database runs until evidence supports them.
