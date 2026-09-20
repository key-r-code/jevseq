# Decision record

20 September 2026

## Agreed direction

- Name: **jevseq**.
- Preserve the user's original objective: FASTA input and reference-relative known/unknown decisions.
- Investigate query-only Jev as a bounded baseline, while recognizing that the reference split is not visible to it.
- Explore sequence search to make a large reference collection accessible within Jev's input limits.
- Record the plan before implementing experiments. No results exist yet.

## Working defaults, pending refinement

- Private GitHub repository under the authenticated account.
- Local BLAST+ pilot; consider MMseqs2 after sensitivity/throughput measurements.
- Family-level holdout and bacterial reads as initial scope proposals.
- InSilicoSeq and existing reference resources, after inventory and provenance checks.
- Compare Jev against search-only rules and a fitted baseline using matched evidence.

## Alternatives discussed

- Full reference FASTAs in a single Jev request: incompatible with current context limits.
- Customer fine-tuning: the current official service does not offer it.
- Selected k-mer counts/frequencies: possible experimental representation; not chosen for the initial pipeline. Dense 9-mer tables have 262,144 possible non-canonical entries per model.
- Multiple successive reference uploads: do not create persistent learned weights.
- Contig prioritization or broader classifier evidence fusion: possible future projects, outside the initial focus.

## Choices still open

See PRD section 12. Family rank, BLAST settings, sample size, API budget, success thresholds, and publication scope remain proposals rather than user-approved experimental constants.
