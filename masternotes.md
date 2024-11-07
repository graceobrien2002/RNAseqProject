# Workflow details for the _Candida Albicans_ RNA sequence analysis
## Upstream workflow
### Data Accession 
Data for Candida Albicans was obtained using GTF and FNA files from NCBI's genome database. The link to access these datasets is [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000182965.3/) .
### Preprocessing and Quality Control
Trimmomatic (version 0.39) was used to trim reads based on quality. 
Trimmomatic script: [linked here](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/trimmomatic_run1) .
We removed specific adapters sequences for Illumina LINK ADAPTER. We also trimmed DESCRIBE FILTERS HERE.
FastQC (version 0.12.0) was then performed on the forward and reverse trimmed paired-end reads to visualize the quality control of data. DESCRIBE FINDINGS HERE. LINK OUTPUT HTML. 
## Sequence Mapping
Bowtie2 (version 2.5.3) was used on forward and reverse paired-end reads obtained from trimmomatic to map the data to reference sequence LABEL AND LINK REFSEQ HERE (where was it found). LINK SCRIPT HERE.
DESCRIBE FINDINGS HERE. LINK OUTPUT HERE.

## Downstream workflow
All downstream analyses done in R (VERSION)
Using DESeq2 (VERSION) LINK SCRIPT HERE.
