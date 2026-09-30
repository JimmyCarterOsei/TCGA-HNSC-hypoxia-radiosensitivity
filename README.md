# TCGA HNSC hypoxia and radiosensitivity

Analysis code for development and external evaluation of hypoxia related prognostic signatures in HNSCC.

## Current contents

This release contains analysis notebook source code. Saved outputs, execution counts and incidental notebook metadata have been removed. The code cells are unchanged from the supplied notebooks. Code hashes and source filenames are recorded in code_manifest.json.

Processed expression matrices, clinical records, annotation inputs and coefficient CSVs are not included in this code upload. The repository is not yet a self contained reproducibility package. Do not describe those data as publicly available here until they have been added and checked.

## Analysis map

- notebooks/02 and 03: candidate pool construction and gene mapping.
- notebooks/04: original LASSO derivation, including the original standardisation before cross validation.
- notebooks/11: subsequent check with standardisation within training folds. The penalty grid remains based on the full development cohort.
- notebooks/05: external evaluation of locked signatures.
- notebooks/06: adapted Eustace comparator.
- notebooks/12: combined external score construction, including corrected RSI ranking and the 28 gene Lendahl adaptation.
- notebooks/13: external clinical variable extraction.
- notebooks/22: main clinical adjustment and sensitivity models.
- notebooks/25: corrected survival curves with risk tables and censoring marks.
- notebooks/Cohort_table_ROUND3.ipynb: cohort characteristics with reverse Kaplan Meier follow up.
- notebooks/Gene_selection_stability_ROUND4.ipynb: bootstrap selection stability at exact penalties, with explicit warning capture.
- notebooks/09 and 10: additional corrected Lendahl and RSI analyses. These exploratory comparators are not presented as validated clinical assays.
- historical/01: earlier TCGA signature analysis framework, including experimental comparator work. This is retained for provenance and should not replace later corrected analyses.

## Running the notebooks

Install the packages listed in requirements.txt in an isolated Python environment. Exact package versions are awaiting confirmation, so that file is not a record of the original environment.

The original code reads input files relative to the kernel working directory and writes outputs there. Use a separate working directory for each analysis and copy the required inputs into it. Open the selected notebook from that directory and run cells in order. Preserve the locked coefficient CSVs as inputs to external evaluation; derivation runs write coefficient files and should be run separately.

The combined score notebook writes GSE65858_all_signatures_patient_scores.csv. Later notebooks read VERIFIED_CORRECT_scores.csv. The latter is a separately verified input from the corrected analysis package; the filename transition is not automated in this release.

Input filenames identified in the code are listed below. Some are intermediate outputs from earlier steps. The TCGA combined input and GSE65858 spreadsheet are study specific processed files, not interchangeable with arbitrary downloads bearing the same accession.

- FROZEN_candidate_pool.csv
- GPL10558_HumanHT-12_V4_0_R2_15002873_B.txt
- GSE65858.xlsx
- HNSC_hypoxia_signature_lambda_1se.csv
- HNSC_hypoxia_signature_lambda_min.csv
- VERIFIED_CORRECT_scores.csv
- clinical_rna_fire_combine.csv
- full_candidate_pool_final_coverage.csv
- full_candidate_pool_with_coverage.csv
- gse65858_clinical_variables.csv

## Limitations and provenance

This upload has passed source equality and Python syntax checks. The full workflow has not been rerun as part of repository preparation.

Several original notebooks suppress warnings globally; their code is retained as supplied. The Round 4 stability notebook instead captures warnings for each fit. Do not infer that absence of saved warnings in an older notebook establishes convergence.

The cohort table notebook also extracts disease free endpoint information for checking. Overall survival is the analysed endpoint of the reported work.

The signatures concern prognostic associations. Cohort specific standardisation and the absence of established independent clinical value limit individual patient application.
