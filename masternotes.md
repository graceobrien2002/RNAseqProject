# Workflow details for the _Candida Albicans_ RNA sequence analysis
## Upstream workflow
### Data Accession 
Data for Candida Albicans was obtained using GTF and FNA files from NCBI's genome database. The link to access these datasets is [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000182965.3/) .
Before cleaning, FastQC was performed on forward and reverse reads to use later as comparison of how effective cleaning was. For reference here are the FastQC reads before cleaning was performed: [Forward read](file:///Users/graceobrien/Downloads/WTA2_1_fastqc.html) [Reverse read](file:///Users/graceobrien/Downloads/WTA2_2_fastqc.html) .
### Preprocessing and Quality Control
All preprocessing and quality control was performed on SSH key of google cloud grapes controller. Trimmomatic (version 0.39) was used to trim reads based on quality. 
Trimmomatic script: [linked here](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/trimmomatic_run1) .
We removed specific [adapters sequences](https://github.com/graceobrien2002/RNAseqProject/blob/main/scripts1/TruSeq3-PE_adapter_sequence) for Illumina. We used trimmomatic filters of:
ILLUMINACLIP:2:30:10 to trim adapter sequences. 
HEADCROP:10 to remove the first 10 bases. 
TRAILING:10 to remove the last 10 bases. 
SLIDINGWINDOW:4:15 to improve data quality. This scans over 4 bases at a time, and if the quality score is not at least an average of 15, trimmomatic will trim that read.
MINLEN:75 to ensure that only reads with at least 75 bases are kept after trimming.
FastQC (version 0.12.0) was then performed on the forward and reverse trimmed paired-end reads to visualize the quality control of data. DESCRIBE FINDINGS HERE. LINK OUTPUT HTML. 
## Sequence Mapping
Bowtie2 (version 2.5.3) was used on forward and reverse paired-end reads obtained from trimmomatic to map the data to reference sequence LABEL AND LINK REFSEQ HERE (where was it found). LINK SCRIPT HERE.
DESCRIBE FINDINGS HERE. LINK OUTPUT HERE.

## Downstream workflow
All downstream analyses done in R (VERSION)
Using DESeq2 (VERSION) LINK SCRIPT HERE.
