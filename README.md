# jevseq

Research framework for evaluating Jev and sequence-search baselines on reference-relative novelty detection in metagenomics.

**Status:** research planning, 20 September 2026. No experiments have run and no performance claims are established.

## Start here

- [Research plan and PRD](docs/research-prd.md): scope, experiments, requirements, evaluation, and milestone gates.
- [Visual research overview](docs/research-overview.html): open locally in a browser; self-contained and usable offline.
- [Decision record](docs/decisions.md): agreed direction, proposals, and unresolved choices.
- [Sources](docs/sources.md): manuscript context and technical references.

## Research question

Does Jev improve reference-relative novelty detection beyond the sequence-search evidence it receives?

The intended interface is reference FASTAs plus query FASTA/FASTQ, with per-query predictions and reproducible evaluation. The first pilot compares search-only decisions with Jev decisions from identical retrieved evidence. A query-only Jev baseline probes existing sequence knowledge, with limited interpretability because its training corpus is unknown.

## Planned structure

Implementation may later add `src/`, `configs/`, `tests/`, and `scripts/`. Keep reference genomes, simulated reads, API credentials, and raw experiment results outside version control. No runtime dependencies or experiment code exist yet.

## Repository policy

Start private. Select a license and data-sharing policy before any public release. Do not commit the source manuscript or its datasets without a separate decision.
