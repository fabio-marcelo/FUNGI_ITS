# FUNGUS_ITS
This is a pipeline implemented in nextflow to identify fungus through ITS2 metabarcoding generated in S5 IonTorrent. The outputs (one csv and one html report) are stored in the `final_report` folder inside the output directory.



# Prerequisites
* OS Ubuntu
* Docker [tutorial](https://docs.docker.com/engine/install/ubuntu/)
* Nextflow [tutorial](https://www.nextflow.io/docs/latest/getstarted.html)
* Nextflow deals with image download and container run by itself.

# Database
The database used for fungi taxonomy and included in this repository was download from [UNITE QIIME release_29.11.2022 for Fungi](https://dx.doi.org/10.15156/BIO/2483915). It includes the trainned classifier for `quay.io/qiime2/core:2023.7`.
Files are stored in the `arquivos_db` folder in gz format. The pipeline itself decompress the files.

# Pipeline
The pipeline is executed through `quay.io/qiime2/core:2023.7` image and includes a python script for data preprocessing and a R script to generate the html report through `quarto`.

## Parameters
* Due to variability in the length of the sequences for ITS,we opted for not using the parameter `--p-trunc-len` = 0 [tutorial](https://benjjneb.github.io/dada2/ITS_workflow.html);
* As the pattern shown by Ion S5 fastq files quality the default for parameters `p_quality_cutoff_5end` and `p_quality_cutoff_3end` is 20;
* `p_error_rate = 0.2`
* `p_minimum_length = 80`
* `p_max_ee` = 2
* Percent identity for `qiime feature-classifier classify-consensus-vsearch = 0.99`
* Percent identity for `qiime feature-classifier classify-consensus-blast = 0.99`
* Percent identity for `qiime feature-classifier classify-sklearn = 0.99`
* Percent identity for `qiime feature-classifier classify-sklearn --p-reads-per-batch = 10000`
