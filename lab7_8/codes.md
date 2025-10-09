##for class I started with salloc -A ACF-UTK0011 --nodes=1 --ntasks=1 --time=01:30:00 --partition=campus
## Faculty already created a conda environment for running quilma , Tree and Taxonomy
## Conda activation in class step : 
  line 1:  module load anaconda3/2024.06
  line 2: source $ANACONDA3_SH
  line 3: onda activate /lustre/isaac24/proj/UTK0385/CONDA_ENVS/qiime2-amplicon-2025.7 (created by faculty)

##about QiiME2 ("Chime 2")
## QIIME2 = Quantitative Insights Into Microbial Ecology (v2), An open-source, community-driven microbiome analysis platform, Reproducible and transparent: every step tracked in provenanc
## Widely used for 16S/18S/ITS amplicon sequencing but expanding into metagenomics and metatranscriptomics
## plugins a specific type of analysis (e.g., dada2, phylogeny, diversity, taxa)
## Example demux - import and demultiplex reads , feature-classifier - taxonomic assignment
## For more online plugins look into the website " https://amplicon-docs.qiime2.org/en/latest/references/available-plugins.html "
## prduces two types pf file ** (.qza)- Artifacts [data objects feature tables, sequences, trees, etc.] and ** (.qzv)- Vizualization [nteractive reports quality plots, taxonomic bar plots, PCoAs] and finally .qzv files can be opened in QIIME2 View (https://view.qiime2.org)

## coding begins 
## at first I logged into the UTK class server ustre/isaac24/proj/UTK0385 then cd eshahana

## for this class meta data was provided into the cnda environment and to access that
    (/lustre/isaac24/proj/UTK0385/CONDA_ENVS/qiime2-amplicon-2025.7) [eshahana@login2 eshahana]$
## verify files presence 
    ls -l emp-single-end-sequences.qza sample-metadata.tsv

## I saw  
     -rw-r--r-- 1 eshahana UTK0385 153072 Oct 3 09:30 emp-single-end-sequences.qza
     -rw-r--r-- 1 eshahana UTK0385    500 Oct 3 09:30 sample-metadata.tsv

##### Demultiplexing 
## Once I see both files listed by the ls -l command, you can run the QIIME 2 command with confidence 
    qiime demux emp-single --i-seqs emp-single-end-sequences.qza --m-barcodes-file sample-metadata.tsv --m-barcodes-column barcode-sequence --o-per-sample-sequences demux.qza --o-error-correction-details demux-details.qza
    
    Explnanation
    *->> Input Sequence File: ** emp-single-end-sequences.qza **      (Your raw, barcoded sequence data)
    *->> Input Metadata File: ** sample-metadata.tsv **               (Your file linking barcode sequences to sample names)
    *->> Output Demultiplexed ** Data: demux.qza **                      (The main output file containing sequences sorted by sample)
    *->> Output Error Details: ** demux-details.qza  **                          (A summary file of error correction results)

## FOR demultiplexing summary 
## Output demux-details.qza (A summary file of error correction results) in commant line type below codes
  
    qiime demux summarize \
--i-data demux.qza \
--o-visualization demux.qzv

## I generated files demux.qza demux.qzv and  error correction details demux

  Explanation
  *-> qiime demux summarize: This is the QIIME 2 command to generate a summary of demultiplexed sequences (like the one you created as demux.qza)
  *-> --i-data demux.qza: This specifies the input file (--i-data) to be summarized. This is the .qza file created by the previous qiime demux emp-single step.
  *-> --o-visualization demux.qzv: This specifies the output file (--o-visualization), which is the .qzv visualization file you will use to view the results (like sequence counts per sample and quality score plots) on the QIIME 2 View website.

------------------------------------------------

## Another step : if the metadata is not provided like the faculy gave us in conda environment 
## we can download and follow the steps 
## download metadata file into a new moving_pictures directory using wget
    wget -O 'sample-metadata.tsv' \
'https://moving-pictures-tutorial.readthedocs.io/en/latest/data/moving-pictures/sample-metadata.tsv’ 

# unzip 
    unzip -d emp-single-end-sequences emp-single-end-sequences.zip
# Importing data  
    qiime tools import \
--type 'EMPSingleEndSequences' \
--input-path emp-single-end-sequences \
--output-path emp-single-end-sequences.qza

## We can  again do the demultiplexing 
    qiime demux emp-single --i-seqs emp-single-end-sequences.qza --m-barcodes-file sample-metadata.tsv --m-barcodes-column barcode-sequence --o-per-sample-sequences demux.qza --o-error-correction-details demux-details.qza


## again for demultiplexing summary 
    qiime demux summarize \
--i-data demux.qza \
--o-visualization demux.qzv

------------------------------------------------------

## Now denoising  with DADA2  and Deblur 
## method denoises single-end sequences, dereplicates them, and filters chimeras
##  Take a look at the input, parameters, and outputs and give values for ‘trun_len’ and ‘trim_left’ based on what you saw in your  ** demux.qzv* ** 
## has to be able to understand that demux qzv because there is no given rule. 

## The code given in class 
    
qiime dada2 denoise-single \
--i-demultiplexed-seqs demux.qza \
--p-trim-left VALUE \
--p-trunc-len VALUE \
--o-representative-sequences rep-seqs.qza \
--o-table table.qza \
--o-denoising-stats stats.qza


***** trim left and trim right values  values are highly specific to your sequencing data's quality and need to be determined by looking at the quality plot from the visualization file you just created (demux.qzv).
TO do this : -- Download demux.qzv: Transfer the file demux.qzv from the cluster (sk131602) to your local computer.
            -- View the File: Open a web browser and go to the QIIME 2 View website ([https://view.qiime2.org](https://view.qiime2.org)
             --The demux.qzv file you just generated contains a quality plot that shows the average quality scores across all cycles/bases of your reads. --You need this to choose your trimming and truncation lengths.
            --Find the Plot: Look for the interactive "Per-base quality scores" plot.

## the code I used 

qiime dada2 denoise-single \
--i-demultiplexed-seqs demux.qza \
--p-trim-left 10 \
--p-trunc-len 150 \
--o-representative-sequences rep-seqs.qza \
--o-table table.qza \
--o-denoising-stats stats.qza

## sometimes error can occure saying "No reads passed the filter" in DADA2 means that zero sequences met your filtering criteria. This almost always happens because truncation length (--p-trunc-len 250) is too long for the actual length and quality of your reads."
##  If my reads are, for example, 240 bases long, or if the quality of the reads drops off sharply before base 250, then every single read will be thrown out when you set --p-trunc-len 250.

## so look into " per base quality plot and  the base position where the average quality score (the median line) consistently drops below 30 or 20

#### to check the denoising code success 
    ls -l rep-seqs.qza table.qza stats.qza

### Expected Successful Output
    -rw-r--r-- 1 eshahana UTK0385 123456 Oct 3 10:07 rep-seqs.qza
    -rw-r--r-- 1 eshahana UTK0385 7890 Oct 3 10:07 table.qza
    -rw-r--r-- 1 eshahana UTK0385 987 Oct 3 10:07 stats.qza


### Summarize the Denoising Results
## After DADA2 runs successfully, the best way to confirm the filtering efficiency is to visualize the stats.qza file. This tells you how many reads were lost at each filtering step

qiime metadata tabulate \
--m-input-file stats.qza \
--o-visualization stats.qzv

#### if dada2 does not run succesfully we can not run the diversity
## Sometimes the dada codes run a gives success but failed to do the next diversity step phylogeny steps 
## then we need to again tru to fix the trim left and trunc len values
### Verify Denoising Stats by seeing the stats.qza file in the directory 



## Generate and download summary visualizations for the feature table and feature data produced by denoising
qiime feature-table summarize \
--i-table table.qza \
--m-sample-metadata-file sample-metadata.tsv \
--o-visualization table.qzv

qiime feature-table tabulate-seqs \
--i-data rep-seqs.qza \
--o-visualization rep-seqs.qzv

## or if the above does not run we can try the below one 
## Since the DADA2 step is complete, you should check the definitive post-denoising feature count
qiime feature-table summarize \
--i-table table.qza \
--o-visualization table_summary.qzv

## The output will be  
table_summary.qzv ## we have to view it [https://view.qiime2.org](https://view.qiime2.org)]. 
                  ## Look for the Minimum Frequency in the "Per-sample summary" tab.

## This will help us to set sample deapth number for diversity 
## for example in class based on my raw counts, the smallest sample was 1854, so DADA2 likely dropped its final count far below 1800.
## Thus sampling deapth set to 1000

------------------------------------------
## Core analysis 1 - Phylogeny pipeline

## This pipeline creates a rooted phylogenetic tree (rooted-tree.qza) . This tree is required for phylogenetic diversity metrics (like UniFrac and Faith's PD)
## 1 . Align your sequences  2. Filter positions with too much variability 3. Generate an unrooted tree with a hybrid tree-building approach ( FAst tree , RAxMl, IQ - tree), 4. Root the tree at the midpoint


#  Align sequences
qiime alignment mafft \
--i-sequences rep-seqs.qza \
--o-alignment aligned-rep-seqs.qza

# Mask hyper-variable columns
qiime alignment mask \
--i-alignment aligned-rep-seqs.qza \
--o-masked-alignment masked-aligned-rep-seqs.qza

#  Build and root the tree
qiime phylogeny fasttree \
--i-alignment masked-aligned-rep-seqs.qza \
--o-tree unrooted-tree.qza
qiime phylogeny midpoint-root \
--i-tree unrooted-tree.qza \
--o-rooted-tree rooted-tree.qza


--------------------------------------
## core analysis 2 - Core Diversity Metrics

## This is where you calculate alpha and beta diversity and generate interactive visualizations (PCoA plots) using the rooted tree and your feature table.

# Set the sampling depth
SAMPLING_DEPTH=1000

# Run  Core Metrics Pipeline to fix the "less than two dimensions" error
qiime diversity core-metrics-phylogenetic \
--i-phylogeny rooted-tree.qza \
--i-table table.qza \
--p-sampling-depth $SAMPLING_DEPTH \
--m-metadata-file sample-metadata.tsv \
--output-dir core-metrics-results_1000

# Generate Alpha Rarefaction Curve for confirmation
qiime diversity alpha-rarefaction \
--i-table table.qza \
--p-max-depth $SAMPLING_DEPTH \
--m-metadata-file sample-metadata.tsv \
--o-visualization alpha-rarefaction.qzv

## After successfully completed the Phylogeny and Diversity analysis steps the output files :




## Core analysis 3 - taxonomy
## This step assigns a name (genus, species, etc.) to each sequence.
## need to download a pre-trained classifier file  " classifier.qza " for my  specific sequencing region (e.g., 16S V4) and place it in your working directory before running this.
## we need a pre-trained Naive Bayes classifier that is specific to our primers (e.g., 515F/806R for V4) and reference database (SILVA or Greengenes).

## downloaded the classsifier file from the conda environment provided 
wget -O 'silva-138-99-nb-classifier.qza' 'https://data.qiime2.org/classifiers/sklearn-1.4.2/silva/silva-138-99-nb-classifier.qza'

## the filename itself is a standard QIIME 2 naming convention:

    silva: The name of the reference database. 
    138: The version number of the database.
    99: The sequence identity threshold (99%

## silva-138-99-nb: This means the classifier was trained on the Silva database (version 138), using sequences clustered at 99% similarity. The nb stands for Naïve Bayes, which is the type of machine learning model used to assign the taxonomy.

## Our target region was the V4 region of the 16S rRNA gene.
## Context: The data you are analyzing (from the QIIME 2 Moving Pictures tutorial) was generated using primers that target this specific, highly variable region of the 16S gene, which is common for bacterial and archaeal community studies.
## Ideal Classifier: Ideally, you would download a classifier that was pruned (trimmed) specifically to this V4 region.



## Assign Taxonomy 
qiime feature-classifier classify-sklearn \
--i-classifier silva-138-99-nb-classifier.qza \
--i-reads rep-seqs.qza \
--o-classification taxonomy.qza


## output :
 

 ## Vizualize Taxonomy
qiime metadata tabulate \
--m-input-file taxonomy.qza \
--o-visualization taxonomy.qzv


## Output:





---------------------------------
## Some explanation


## How do I know which classifier to download?

The general rule for choosing a QIIME 2 classifier is driven by three main criteria:

## QIIME 2 Version's Dependencies (The Most Important!)

You must match the version of the underlying software (like scikit-learn) in your environment to the version used to train the classifier.

    How to check: In your case, when the error occurred, QIIME 2 told you exactly what was required: your current system was running scikit-learn 1.4.2.

    Action: You must always look for a classifier package that explicitly mentions that version number in the path (e.g., sklearn-1.4.2).

## The Target Gene Region

Different classifiers are optimized for different variable regions (V1-V2, V3-V4, V4, etc.). You must choose a classifier that covers the region your sequencing primers targeted.

    Your Data: The Moving Pictures tutorial data you are using targets the V4 region of the 16S rRNA gene, typically using the 515F/806R primers.

    Action: Ideally, you would download a V4-specific classifier. Since you used a general 16S classifier (silva-138-99-nb), it worked, but in the future, look for classifiers explicitly pruned for your region (e.g., a "515F-806R" classifier).

## The Reference Database

This choice depends on the type of organisms you expect to find.

    Silva: The most common and broad bacterial/archaeal database. (What you used)

    Greengenes: An older, popular database (no longer updated).

    UNITE: Essential for Fungi (ITS region).

Rule of Thumb: Always start with Silva (for bacteria/archaea) and ensure the scikit-learn version matches your current QIIME 2 installation.