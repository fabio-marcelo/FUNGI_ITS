# FUNGUS_ITS
This is a pipeline implemented in nextflow to identify fungus through ITS2 metabarcoding generated in S5 IonTorrent. The outputs (one csv and one html report) are stored in the `final_report` folder inside the output directory.



# Prerequisites
* OS Ubuntu
* Docker [tutorial](https://docs.docker.com/engine/install/ubuntu/)
* Nextflow [tutorial](https://www.nextflow.io/docs/latest/getstarted.html)

# Database
The database used for fungi taxonomy and included in this repository was download from [UNITE QIIME release_29.11.2022 for Fungi](https://dx.doi.org/10.15156/BIO/2483915). It includes the trainned classifier for `quay.io/qiime2/core:2023.7`.

# Pipeline

