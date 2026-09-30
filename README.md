# TCGA HNSC hypoxia and radiosensitivity

Analysis code for development and external evaluation of hypoxia related prognostic signatures in HNSCC.

## Current contents

This release contains analysis notebook source code. Saved outputs, execution counts and incidental notebook metadata have been removed. The code cells are unchanged from the supplied notebooks. Code hashes and source filenames are recorded in code_manifest.json.

The data directory contains locked coefficients, frozen candidate mapping, corrected cohort scores, clinical annotations, final adjusted results, cross validation outputs and the Round 4 stability results. Checksums are recorded in data_manifest.json.

The combined TCGA expression and clinical input, GSE65858.xlsx expression workbook and GPL10558 annotation are available as [release downloads](https://github.com/JimmyCarterOsei/TCGA-HNSC-hypoxia-radiosensitivity/releases/tag/data-inputs-2026-09-30). Download all three separately; release assets are not included in GitHub's source ZIP. Decompress clinical_rna_fire_combine.csv.gz to clinical_rna_fire_combine.csv before running the notebooks. Download instructions and checksums are provided below.

## Download the large inputs

Download these assets from [data-inputs-2026-09-30](https://github.com/JimmyCarterOsei/TCGA-HNSC-hypoxia-radiosensitivity/releases/tag/data-inputs-2026-09-30):

| Release asset | Use |
| --- | --- |
| [clinical_rna_fire_combine.csv.gz](https://github.com/JimmyCarterOsei/TCGA-HNSC-hypoxia-radiosensitivity/releases/download/data-inputs-2026-09-30/clinical_rna_fire_combine.csv.gz) | Combined TCGA expression and clinical input; decompress to `clinical_rna_fire_combine.csv`. |
| [GSE65858.xlsx](https://github.com/JimmyCarterOsei/TCGA-HNSC-hypoxia-radiosensitivity/releases/download/data-inputs-2026-09-30/GSE65858.xlsx) | Processed external-cohort expression workbook. |
| [GPL10558_HumanHT-12_V4_0_R2_15002873_B.txt](https://github.com/JimmyCarterOsei/TCGA-HNSC-hypoxia-radiosensitivity/releases/download/data-inputs-2026-09-30/GPL10558_HumanHT-12_V4_0_R2_15002873_B.txt) | Platform annotation used by the supplied notebooks. |

To download all three files from the repository root, run the following in the activated environment. It checks each asset against `release_assets_manifest.json`, extracts the TCGA CSV and checks the decompressed bytes against the original file:

```python
import gzip
import hashlib
import json
import shutil
import urllib.request
from pathlib import Path

inputs = Path("inputs")
inputs.mkdir(exist_ok=True)
manifest = json.loads(Path("release_assets_manifest.json").read_text())
base = manifest["release"].replace("/tag/", "/download/")

def sha256(path):
    digest = hashlib.sha256()
    with path.open("rb") as handle:
        for chunk in iter(lambda: handle.read(1024 * 1024), b""):
            digest.update(chunk)
    return digest.hexdigest()

for asset in manifest["files"]:
    target = inputs / asset["name"]
    with urllib.request.urlopen(base + "/" + asset["name"]) as response:
        with target.open("wb") as handle:
            shutil.copyfileobj(response, handle)
    assert target.stat().st_size == asset["size"], target.name
    assert sha256(target) == asset["sha256"], target.name
    if "decompressed" in asset:
        expected = asset["decompressed"]
        csv_path = inputs / expected["name"]
        with gzip.open(target, "rb") as source, csv_path.open("wb") as handle:
            shutil.copyfileobj(source, handle)
        assert csv_path.stat().st_size == expected["size"], csv_path.name
        assert sha256(csv_path) == expected["sha256"], csv_path.name

print("All release inputs verified.")
```

The original attached TCGA filename was `clinical_rna_fire_combine 1(in).csv`; the release uses the filename expected by the notebooks. Compression and filename normalisation do not change the CSV bytes. The decompressed file is 65,232,808 bytes with SHA256 `a907888970d2ac564da877eb20ebcb1b729af7994f701cf9eed062a4a12bcba6`.

The TCGA input has **517 source rows**. Retaining primary-tumour samples (`Sample ID` ending in `-01`) gives 515 rows. Requiring nonmissing, positive `Overall Survival (Months)` gives the analysed cohort of **514 patients and 217 deaths**. Deaths are encoded by `Overall Survival Status` beginning with `1`. The published TCGA asset was downloaded and both compressed and decompressed checksums and these cohort counts were verified.

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

Use **Python 3.12.3**, matching the supplied notebooks. The confirmed package versions are scikit-survival 0.25.0, lifelines 0.30.0, pandas 2.3.3 and scikit-learn 1.7.2. These four packages are pinned in [requirements.txt](requirements.txt), and [.python-version](.python-version) records the Python version. Other dependency versions remain unconfirmed; this is a partial environment specification, not a complete historical lockfile. See [SOFTWARE_VERSIONS.md](SOFTWARE_VERSIONS.md).

With Python 3.12.3 installed, create a clean environment from the repository root:

```bash
python3.12 -c 'import sys; assert sys.version_info[:3] == (3, 12, 3), sys.version'
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

On Windows, create the environment with your Python 3.12.3 executable and activate it with `.venv\Scripts\Activate.ps1`. Check the running notebook kernel's Python and package versions using the example in SOFTWARE_VERSIONS.md.

The original code reads input files relative to the kernel working directory and writes outputs there. Use a separate working directory for each analysis and copy the required inputs into it. Copy the selected notebook into that directory, start Jupyter there (`jupyter notebook`), select the environment's Python kernel and run cells in order. Confirm the kernel working directory with `from pathlib import Path; Path.cwd()` before running analyses. Preserve the locked coefficient CSVs as inputs to external evaluation; derivation runs write coefficient files and should be run separately.

The combined score notebook writes GSE65858_all_signatures_patient_scores.csv. Later notebooks read VERIFIED_CORRECT_scores.csv. The latter is a separately verified input from the corrected analysis package; the filename transition is not automated in this release.

Input filenames identified in the code are listed below. Some are intermediate outputs from earlier steps. In particular, notebook 02 writes `full_candidate_pool_with_coverage.csv`, which notebook 03 reads; this intermediate is generated rather than supplied in `data/`. For derivation or external evaluation using the locked inputs, start with the supplied final candidate mapping and coefficients as applicable. The TCGA combined input and GSE65858 spreadsheet are study specific processed files, not interchangeable with arbitrary downloads bearing the same accession.

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

## Data use

The locked coefficient files contain one row per selected gene. Their HR_per_SD field is the exponentiated individual penalised gene coefficient and is not the externally evaluated composite score hazard ratio.

Use VERIFIED_CORRECT_scores.csv for downstream external analyses. The separate patient score tables contain public cohort identifiers. The TCGA extract has 514 rows, of which 389 have both disease free time and status, with 125 missing and 142 events. Disease free outcomes were not analysed for the reported survival results.

The original notebooks use bare filenames. Copy the required files from data/ into the selected notebook's working directory. The data/cv_correction_check directory contains verification outputs; it does not replace the original locked coefficient inputs.
