# Sources and verification notes

Checked 20 September 2026. Vendor capabilities can change; recheck before implementation.

- Supplied manuscript: *Naïve Bayes Classifier++ as an out-of-distribution detector of novel taxa*, Keyush Ramjattun, Gavin Hearne, Gail L. Rosen. Read from the user-provided `final_manuscript.tex`; not copied into this repository.
- [Jev model specifications](https://docs.typesafe.ai/models): request limits, model versions, customization restrictions, text input.
- [Jev introduction](https://docs.typesafe.ai/introduction): state and typed questions.
- [Jev confidence](https://docs.typesafe.ai/confidence): distinction between probability distribution and confidence statistic.
- [TypeSafe launch overview](https://typesafe.ai/blog/introducing-system-one-models-and-jev): vendor description of architecture and RLCD; not evidence of genomic capability.
- [NCBI BLAST+ manual](https://www.ncbi.nlm.nih.gov/books/NBK279684/): nucleotide search and tabular output fields.
- [MMseqs2 official repository](https://github.com/soedinglab/MMseqs2): search workflows and nucleotide support.
- [CAMI II](https://www.nature.com/articles/s41592-022-01431-4): metagenomic benchmark design.
- [CAMI benchmarking portal paper](https://academic.oup.com/nar/article/53/W1/W102/8126258): evaluation resources and historical-reference contamination caveat.
- [NBC novelty code](https://github.com/EESI/NBC-novelty-detection) and [NBC++](https://github.com/EESI/Naive_Bayes): locations cited by the supplied manuscript; not audited for this plan.

No Jev API calls, genome downloads, simulations, or performance measurements have been performed. Numerical examples discussed in conversation were illustrative, not observations.
