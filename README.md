# FUNGI_ITS_TAXONOMY
This repository contains a scalable, automated Nextflow pipeline designed for the analysis of amplicon sequencing data (e.g., ITS or 16S rRNA). It leverages QIIME 2 for core microbiome bioinformatics, FastQC for quality control, Python for data wrangling, and Quarto (R) for generating comprehensive interactive and static reports.

The pipeline explores the taxonomic classification of sequences using two different minimum length cutoffs (20 bp and 80 bp) and compares results across three distinct taxonomic classification algorithms.

# Prerequisites
* OS Ubuntu
* Docker [tutorial](https://docs.docker.com/engine/install/ubuntu/)
* Nextflow (version 23.10.0) [tutorial](https://www.nextflow.io/docs/latest/getstarted.html)
* Nextflow deals with image download and container run by itself.

# Database
## Get the database
### Unite database
The database used for fungi taxonomy included in this repository was download from [UNITE QIIME release_29.11.2022 for Fungi](https://dx.doi.org/10.15156/BIO/2483915). It includes the trainned classifier for `quay.io/qiime2/core:2023.7`.
Files are stored in the `arquivos_db` folder in gz format. The pipeline itself decompress the files.


* Download
* Unpack
```bash
tar -xvf file_name.tgz
```
* Remove whitespace, blank lines, and lowercase characters from the reference FASTA file.
```bash
awk '/^>/ {print($0)}; /^[^>]/ {print(toupper($0))}' nome_do_arquivo.fasta | tr -d ' ' > nome_do_arquivo_uppercase.fasta
```
### NCBI Database
For the construction of a custom ITS database using data available from NCBI, follow the tutorial [DB4Q2](https://bmcgenomdata.biomedcentral.com/articles/10.1186/s12863-022-01067-5).

## Train the classifier
This step refers to configuring a machine-learning model for the taxonomic identification of gene-marker sequences.

It is recommended that classifiers based on the UNITE database be trained using the complete sequences. If the UNITE database is selected, the dynamic database is also recommended, as the sequences in the other two databases have been trimmed to the ITS region, excluding flanking portions of the rRNA gene where amplicons generated using standard ITS primers may be present ([REF](https://john-quensen.com/tutorials/training-the-qiime2-classifier-with-unite-its-reference-sequences/)).

### Import Reference Files
```bash
#### Import sequence file
echo -e "Import seq file \n"
qiime tools import \
 --type FeatureData[Sequence] \
 --input-path path/to/fasta_file/with_refseqs \
 --output-path path/outputfilename.qza
echo -e "sequence file imported  \n"
```

```bash
#### Import taxon file
echo -e "Import tax file \n"
qiime tools import \
 --type FeatureData[Taxonomy] \
 --input-path path/to/txtfile/with/taxonomy \
 --output-path unite-ver7-99-tax-01.12.2017.qza \
 --input-format HeaderlessTSVTaxonomyFormat
echo -e "Tax file imported \n"
```

It is not recommended to extract or trim sequences from the reference database before training the classifier [REF](https://github.com/qiime2/docs/blob/master/source/tutorials/feature-classifier.rst).

```bash
echo "Start trainning classifier"
qiime feature-classifier fit-classifier-naive-bayes \       #metodo
     --i-reference-reads ref-seqs.qza \                     #sequencias ref
     --i-reference-taxonomy ref-taxonomy.qza \              #taxnomia do db
     --o-classifier classifier.qza                          #output
echo "Finish trainning classifier"
```

# ⚙️ Pipeline Steps Performed

The analysis workflow is orchestrated via Nextflow and encapsulates the following modular steps:

## Initialization & Quality Control

* Automatically generates a sample sheet from the provided fastq folder.

* Runs FastQC on raw reads to assess sequence quality, duplication levels, and adapter content.

## Data Import & Pre-processing

* Creates QIIME 2 manifest and metadata files.

* Imports multiplexed .fastq files into QIIME 2 artifacts (.qza).

## Trimming & Filtering

* Uses Cutadapt (qiime cutadapt trim-single) to remove primer sequences (default: "GGAAGTAAAAGTCGTAACAAGG") and filter reads based on 5' and 3' quality cutoffs.

* Applies dual length filtering, branching the pipeline to analyze sequences with a minimum length of 20 bp and 80 bp.

## Denoising

* Applies DADA2 (qiime dada2 denoise-single) for error correction, quality filtering, and chimera removal, outputting exact Amplicon Sequence Variants (ASVs).

## Taxonomic Classification

* Imports reference sequences and taxonomy files.

* Classifies ASVs using three distinct algorithms for robust comparative analysis:

   * VSEARCH consensus taxonomy classifier.

   * BLAST consensus taxonomy classifier.

   * scikit-learn Naive Bayes machine-learning classifier.

## Data Export & Transformation

* Exports feature tables to BIOM formats.

* Merges taxonomy headers into BIOM files and converts them into .tsv formats.

## Table Formatting (Python)

* A custom Python script (python_its.py) processes the TSV outputs, filters out unassigned taxa, concatenates the results from the three classifiers (BLAST, VSEARCH, sklearn), and formats a final comprehensive count table.

## Final Reporting (Quarto/R)

* Compiles the analytical outputs using R (ngsReports, phyloseq, qiime2R) into highly visual HTML and PDF reports via Quarto.

# 📁 Outputs Generated

Upon completion, the pipeline generates a structured results (default) directory containing:

* samplesheet.csv / manifest-file.tsv / metadata-file.txt: Tracking and mapping files used for QIIME 2 ingestion.

* fastqc_dir/: Directory containing raw FastQC HTML reports and zip files for every input sample.

# QIIME 2 Visualizations (*.qzv):

* view-inspec_import.qzv (Data summary)

* _view-trimmed_sequences.qzv (Post-trimming quality)

* _view-inspect_denoise-stats.qzv (DADA2 tracking stats)

## Visualizations for taxonomic classifications (VSEARCH, BLAST, scikit-learn).

* QIIME 2 Artifacts (*.qza): Representative sequences, denoised tables, and taxonomic assignments for both 20bp and 80bp length filters.

* _final_output.csv & *_biom_with_taxonomy.tsv: Intermediate text-based count tables combined with taxonomic lineages.

## final_report/:

* 20minLen_final_table.csv: Consolidated taxonomy count table for reads >20bp comparing all three classifiers.

* 80minLen_final_table.csv: Consolidated taxonomy count table for reads >80bp comparing all three classifiers.

* report_html.html: Interactive HTML report containing QC metrics, alpha-rarefaction curves, DADA2 stats, and taxonomic distributions.

* report_pdf.pdf: Static PDF version of the complete analysis report.

# 🎯 Final Result

The ultimate deliverable of this pipeline is the Final Report (report_html.html / report_pdf.pdf) coupled with the its_out.csv taxonomy table.

Stakeholders and researchers receive:

* Clear Quality Assurance: Visual confirmation of raw sequence quality and DADA2 denoising efficiency.

* Methodological Rigor: A side-by-side comparison of taxonomic abundance and identification across three distinct classification tools (BLAST, VSEARCH, scikit-learn).

* Length Sensitivity Analysis: Explicit contrasting of data processed at a 20bp minimum length versus an 80bp minimum length to understand how short fragments impact taxonomic profiling.

* Actionable Data: Clean, merged CSV files ready for downstream statistical analysis (e.g., in R or Python) without needing further bioinformatics manipulation.


# Parameters
* Due to variability in the length of the sequences for ITS,we opted for not using the parameter `--p-trunc-len` = 0 [tutorial](https://benjjneb.github.io/dada2/ITS_workflow.html);
* As the pattern shown by Ion S5 fastq files quality, the default for parameters `p_quality_cutoff_5end` and `p_quality_cutoff_3end` are 20;
* `p_error_rate = 0.2`
* `p_minimum_length = 80`
* `p_max_ee` = 2
* Percent identity for `qiime feature-classifier classify-consensus-vsearch = 0.99`
* Percent identity for `qiime feature-classifier classify-consensus-blast = 0.99`
* Percent identity for `qiime feature-classifier classify-sklearn = 0.99`
* Percent identity for `qiime feature-classifier classify-sklearn --p-reads-per-batch = 10000`
* Primer sequence for `qiime cutadapt trim-single --i-demultiplexed-sequences --p-front GGAAGTAAAAGTCGTAACAAGG`

# Usage
## Help message
```bash
nextflow run github/its_pipeline/main.nf --help

Usage:
 The typical command for running the pipeline is as follows:

 nextflow run main.nf --primer_seq 'primer_sequence' \
 	--fastq_folder '/path/to/my/fastq_files' \
 	--trainned_classifier 'full_path/and_name/to/my/trainned_classifier' \
 	--ref_reads 'full_path/to/reference_reads.fasta' \
 	--tax_file 'full_path/to/tax_file.txt' \
 	--outdir 'output-folder-name' \
 	--threads "15"

 If using default parameters, you only need to provide "fastq_folder" and a csv file (named samplesheet.csv)
 containing the sample names in the first column named "sample", reads1 in the second column named "r1" and
 a third column empty column "r2". The .csv fie must be placed inside the `--fastq_folder`.




 Mandatory arguments:
  --fastq_folder                 Folder with fastq files (full path required)
  --primer_seq                   Primer sequence ("AACTCCG") [must be surrounded with quotes]
                                 [Default: "GGAAGTAAAAGTCGTAACAAGG"]
  --trainned_classifier          full_path/to/trainned_classifier.qza (full path required)
                                 [Default: trainned_qiime-2023.7_ver9_99_s_all_29.11.2022_dev.qza.gz]
  --ref_reads                    Full path to reference reads file (.fasta) (full path required)
                                 [Default: sh_refs_qiime_ver9_99_s_all_29.11.2022_dev.fasta.gz]
  --tax_file                     Full path to taxonomy file (.txt) (full path required)
                                 [Default: sh_taxonomy_qiime_ver9_99_s_all_29.11.2022_dev.txt.gz]
  --outdir                       Output folder name

Optional arguments:
 --threads                      Number of CPUs to use [Default: 15]

   qiime cutadapt trim-single arguments
   --p_quality_cutoff_5end        Quality cut off in 5' end [Default: 20]
   --p_quality_cutoff_3end        Quality cut off in 3' end [Default: 20]
   --p_error_rate                 Maximum allowed error rate [Default: 0.2]
   --p_minimum_length             Discard reads shorter than specified value. Note, the 
                                  cutadapt default of 0 has been overridden, because
                                  that value produces empty sequence records [Default: 80]

   qiime dada2 denoise-single arguments
   --p_max_ee                     Reads with number of expected errors higher than
                                  this value will be discarded [Default: 2] 
   --p_trunc_len                  Position at which sequences should be truncated due
                                  to decrease in quality. This truncates the 3' end of
                                  the of the input sequences, which will be the bases
                                  that were sequenced in the last cycles. Reads that
                                  are shorter than this value will be discarded. If 0
                                  is provided, no truncation or length filtering will
                                  be performed [Default: 0]
```
## Run the pipeline
You need to provide at least the `--fastq_folder` parameter.

`nextflow run github/its_pipeline/main.nf --fastq_folder 'its_reads'`

## Output files
The default name for the output dir is "results". Inside the folder you'll find a directory called "final_report" (see an example in the `final_report` dir). It contains a ".html" file reporting quality control and denoise statistics,
alpha-rarefaction curves and a taxonomy table comparing results from classify-consensus-vsearch, classify-consensus-blast and classify-sklearn.
