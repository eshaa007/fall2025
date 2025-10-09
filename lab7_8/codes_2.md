# QIIME 2 Analysis Workflow

This document provides a step-by-step guide for performing microbiome analysis using QIIME 2. The instructions are based on the class environment and include commands, explanations, and troubleshooting tips.

---

## Getting Started

### Allocating Resources
To begin, allocate resources using the following command:
```bash
salloc -A ACF-UTK0011 --nodes=1 --ntasks=1 --time=01:30:00 --partition=campus
```

### Conda Environment Setup
The faculty has provided a pre-configured Conda environment for running QIIME 2. Activate it as follows:
```bash
module load anaconda3/2024.06
source $ANACONDA3_SH
conda activate /lustre/isaac24/proj/UTK0385/CONDA_ENVS/qiime2-amplicon-2025.7
```

---

## About QIIME 2

**QIIME 2** (Quantitative Insights Into Microbial Ecology) is an open-source platform for microbiome analysis. It is widely used for 16S/18S/ITS amplicon sequencing and is expanding into metagenomics and metatranscriptomics.

### Key Features:
- **Plugins**: Perform specific analyses (e.g., `dada2`, `phylogeny`, `diversity`, `taxa`).
- **File Types**:
    - `.qza`: Artifacts (data objects like feature tables, sequences, trees, etc.).
    - `.qzv`: Visualizations (interactive reports like quality plots, taxonomic bar plots, etc.).
- **Visualization**: `.qzv` files can be viewed at [QIIME 2 View](https://view.qiime2.org).

For more plugins, visit the [QIIME 2 Plugin Directory](https://amplicon-docs.qiime2.org/en/latest/references/available-plugins.html).

---

## Workflow

### Step 1: Verify Input Files
Ensure the required files are present:
```bash
ls -l emp-single-end-sequences.qza sample-metadata.tsv
```
Expected output:
```
-rw-r--r-- 1 eshahana UTK0385 153072 Oct 3 09:30 emp-single-end-sequences.qza
-rw-r--r-- 1 eshahana UTK0385    500 Oct 3 09:30 sample-metadata.tsv
```

---

### Step 2: Demultiplexing
Run the following command to demultiplex the sequences:
```bash
qiime demux emp-single \
    --i-seqs emp-single-end-sequences.qza \
    --m-barcodes-file sample-metadata.tsv \
    --m-barcodes-column barcode-sequence \
    --o-per-sample-sequences demux.qza \
    --o-error-correction-details demux-details.qza
```

#### Explanation:
- **Input Sequence File**: `emp-single-end-sequences.qza`
- **Input Metadata File**: `sample-metadata.tsv`
- **Output Files**:
    - `demux.qza`: Demultiplexed sequences.
    - `demux-details.qza`: Error correction details.

#### Generate Demultiplexing Summary:
```bash
qiime demux summarize \
    --i-data demux.qza \
    --o-visualization demux.qzv
```

---

### Step 3: Denoising with DADA2
Denoise the sequences using the `dada2` plugin:
```bash
qiime dada2 denoise-single \
    --i-demultiplexed-seqs demux.qza \
    --p-trim-left 10 \
    --p-trunc-len 150 \
    --o-representative-sequences rep-seqs.qza \
    --o-table table.qza \
    --o-denoising-stats stats.qza
```

#### Notes:
- **Trim and Truncation Values**: Determine these values by analyzing the quality plot in `demux.qzv`.
- **Troubleshooting**: If you encounter "No reads passed the filter," adjust the truncation length based on the quality plot.

#### Verify Output:
```bash
ls -l rep-seqs.qza table.qza stats.qza
```

#### Summarize Denoising Results:
```bash
qiime metadata tabulate \
    --m-input-file stats.qza \
    --o-visualization stats.qzv
```

---

### Step 4: Phylogeny Pipeline
Build a rooted phylogenetic tree:
1. **Align Sequences**:
     ```bash
     qiime alignment mafft \
         --i-sequences rep-seqs.qza \
         --o-alignment aligned-rep-seqs.qza
     ```
2. **Mask Hyper-variable Columns**:
     ```bash
     qiime alignment mask \
         --i-alignment aligned-rep-seqs.qza \
         --o-masked-alignment masked-aligned-rep-seqs.qza
     ```
3. **Build and Root the Tree**:
     ```bash
     qiime phylogeny fasttree \
         --i-alignment masked-aligned-rep-seqs.qza \
         --o-tree unrooted-tree.qza
     qiime phylogeny midpoint-root \
         --i-tree unrooted-tree.qza \
         --o-rooted-tree rooted-tree.qza
     ```

---

### Step 5: Core Diversity Metrics
Calculate alpha and beta diversity metrics:
```bash
SAMPLING_DEPTH=1000

qiime diversity core-metrics-phylogenetic \
    --i-phylogeny rooted-tree.qza \
    --i-table table.qza \
    --p-sampling-depth $SAMPLING_DEPTH \
    --m-metadata-file sample-metadata.tsv \
    --output-dir core-metrics-results_1000
```

#### Generate Alpha Rarefaction Curve:
```bash
qiime diversity alpha-rarefaction \
    --i-table table.qza \
    --p-max-depth $SAMPLING_DEPTH \
    --m-metadata-file sample-metadata.tsv \
    --o-visualization alpha-rarefaction.qzv
```

---

### Step 6: Taxonomy Assignment
Assign taxonomy using a pre-trained classifier:
1. **Download Classifier**:
     ```bash
     wget -O 'silva-138-99-nb-classifier.qza' \
     'https://data.qiime2.org/classifiers/sklearn-1.4.2/silva/silva-138-99-nb-classifier.qza'
     ```
2. **Classify Sequences**:
     ```bash
     qiime feature-classifier classify-sklearn \
         --i-classifier silva-138-99-nb-classifier.qza \
         --i-reads rep-seqs.qza \
         --o-classification taxonomy.qza
     ```
3. **Visualize Taxonomy**:
     ```bash
     qiime metadata tabulate \
         --m-input-file taxonomy.qza \
         --o-visualization taxonomy.qzv
     ```

---

## Additional Notes

### Choosing the Right Classifier
- **Version Compatibility**: Ensure the classifier matches the scikit-learn version in your environment.
- **Target Region**: Use a classifier specific to your sequencing primers (e.g., V4 region for 16S rRNA).
- **Reference Database**: Silva is recommended for bacterial/archaeal studies.

### Troubleshooting
- **DADA2 Errors**: Adjust `--p-trim-left` and `--p-trunc-len` based on the quality plot.
- **Diversity Analysis Issues**: Verify the sampling depth and ensure all required files are generated successfully.

---

This workflow provides a comprehensive guide for analyzing microbiome data using QIIME 2. For further details, refer to the [QIIME 2 Documentation](https://qiime2.org/documentation/).
